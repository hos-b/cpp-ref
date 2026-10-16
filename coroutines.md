# Coroutines
given the impossible "user" API, it needs a separate file.
C++ implements the _stackless_ coroutine: when suspended, control is returned to the immediate caller. look up Andreas Weis cppcon talk.

# 0. Examples

## 0.1. async computation
the most basic way to do this is to use blocking calls:
```cpp
auto [ec, bytes_read] = read(socket, buffer);
```
to improve the performance, one might use an async library:
```cpp
async_read(socket, buffer, [](std::error_code ec, std::size_t bytes_read) {
    // ...
});
```
there is an inversion of the control flow and the parameters. the caller calls some function and gets values back in the blocking case. in the async case, the library runs a callback function which takes as arguments the expected return values.

coroutines allow us to change the async code s.t. it looks like the blocking case:
```cpp
auto [ec, bytes_read] = co_await async_read(socket, buffer);
```
this will suspend the current coroutine (the `co_*` keywords are only usable inside coroutines) and return control to its caller (or whoever resumed it), while the read is running in the background. as soon as the data becomes available, the coroutine is resumed (on whichever thread/executor the library decides) and the execution continues right after the `co_await`. no thread is blocked while waiting.

## 0.2. suspend/resume
lazy computation:
```cpp
MyCoroutine co = start_computation(initial_data);
auto some_results = co.provide(some_data);
auto more_results = co.provide(more_data);
auto final_results = co.results;
```

## 0.3. basic example: fibonacci
the normal implementation has a flaw:
```cpp
std::vector<int> fibo(int n);
```
it needs to keep all `n` elements in the memory. in general, if we have a slow computation which processes the numbers one by one, we would have to pay for the storage of all numbers. dealing with infinite ranges is also hard.

the previous solutions to these problems were factories:
```cpp
struct FiboGenerator
{
    int next();
private:
    some_sort_of_state;
};
...
// returns a FiboGenerator object that will
// start from the first fibonacci number.
FiboGenerator make_fibo_gen();
```
from the outside coroutines are indistinguishable from such functions. we only have their interface. 
coroutines should be thought of as factory functions that is producing the actual coroutine.

# 1. Intro
calling a coroutine function creates the coroutine frame (the state of the coroutine: parameter copies, local variables, the promise, the suspension point), and returns an object of `ReturnType` to the caller. the interface of `ReturnType` is going to determine what the coroutine can do. the end user will only ever need to understand and deal with this object.

the frame is typically heap allocated (the compiler may elide the allocation if the coroutine's lifetime is provably nested inside the caller's). since it is *stackless*, only the frame is kept alive when suspended, there is no separate stack.

a function is a coroutine if its body contains at least one of the following keywords:
- `co_return`
- `co_yield`
- `co_await`

as soon as the compiler detects this, it changes the rules of how the code inside the body works. reading it as a typical function will not make sense. note that the plain `return` statement cannot be used in a coroutine, only `co_return`.

the following cannot be coroutines: `main`, constructors/destructors, `constexpr` functions and functions with a deduced return type (`auto`) or variadic arguments (`...`). the coroutine support header is `<coroutine>`.

it would be great if the fibonacci coroutine could look something like:
```cpp
FiboGenerator make_fibo_gen() {
    int i1 = 1;
    int i2 = 1;
    for (;;) {
        co_yield i1;
        i1 = std::exchange(i2, i1 + i2);
    }
}
```

# 2. Barebones Coroutine

```cpp
struct ReturnType
{
    struct promise_type{};
};

ReturnType hello_coroutine() {
    co_return;
}
```
this doesn't actually compile yet, because the compiler needs a few functions to be defined by the `promise_type`. (the return type has to be declared before the coroutine that uses it.)

## 2.1. promise_type
`std::future` and `std::promise` are part of the C++ threading library. future is what is handed out to the caller, so that it can retrieve the result. the future has the role of the `ReturnType` in coroutines.
we also need something that the callee can put the value in, which is the promise type for threads. the coroutine promise has a similar _idea_ behind it. this object remains inside the coroutine.

unlike the threading object pair, where the producer constructs the promise on its own, the coroutine promise is automatically constructed by the compiler inside the coroutine frame. it is the central intersection point for the coroutine code and the caller code. it determines what happens in the essential timepoints during a coroutine's lifetime, namely what happens at the startup and the completion of the coroutine and how various return values are handled, when the coroutine reaches the end of its control flow.

the compiler expects 5 functions to be inside the promise type:
```cpp
struct promise_type
{
    ReturnType get_return_object() { return {}; }
    void return_void() {}
    void unhandled_exception() {}
    std::suspend_always initial_suspend() noexcept { return {}; }
    std::suspend_always final_suspend() noexcept { return {}; }
};
```
1. `get_return_object`: its return type is the coroutine's return type (`ReturnType`). when the coroutine starts, the compiler implicitly calls this function. it is now up to the `promise_type` to construct the return object that will be handed back to the caller (in the basic example, `FiboGenerator` would be constructed here).

   we do not have direct access to where it is stored (compiler magic): the result lives as a local in the call and is only handed to the caller when the coroutine first suspends or finishes. it is created *before* `initial_suspend` is evaluated. so, with `suspend_always` as `initial_suspend`, the caller gets it right away; with `suspend_never` the body runs up to its first suspension point (e.g. the first `co_yield`) or to completion, and only then does the caller get the object.

3. `return_void`: decides what happens when we reach an empty `co_return;` statement or when the control flow falls off the end of the body. there's also a corresponding `return_value` for when the `co_return` is used in conjunction with a value, but in our example, that is not done. a promise must not have both. falling off the end of a coroutine whose promise has no `return_void` is undefined behaviour.

4. `unhandled_exception`: called inside the implicit `catch` block around the body when an exception escapes it. it can inspect it with `std::current_exception()` and store it, rethrow it, or terminate. afterwards the coroutine proceeds to `final_suspend`.

5. `initial_suspend`
6. `final_suspend`: these two are the customization points that allow us to execute some code, when the coroutine first starts executing (initial_suspend) and shortly before when the coroutine stops executing for good (final_suspend), e.g. if we return from the coroutine or the control flow falls off the end or if we exit via an exception. both return an `Awaitable`. `final_suspend` must be `noexcept`.

`std::suspend_always` tells the compiler, that we always want to suspend at that particular point (intial, final) and hand the control flow back to the caller.
`std::suspend_never` lets the coroutine continue executing without relinquishing control.

the code can actually compile now:
```cpp
ReturnType hello_coroutine()
{
    std::println("Hello from coroutine");
    co_return;
}

int main() {
    hello_coroutine(); // prints nothing
    return 0;
}
```
if we change the `initial_suspend` declaration from `suspend_always` to `suspend_never`, then we see the message on the standard output (`std::println` needs `<print>`, C++23; use `std::cout` otherwise).

note that this example leaks the coroutine frame: it is suspended at `final_suspend` and nobody calls `destroy()` on it (see 2.9).

## 2.2. pausing and awaitables
an `Awaitable` is a type whose objects can be the operand of `co_await`.
```cpp
AsyncRead awaitable = async_read(socket, buffer);
auto [ec, bytes_read] = co_await awaitable;
```
calling `co_await` is an opportunity for suspension. it is the operator that explicitly lets us pause the coroutine and return control flow back to the caller (or resumer). `co_yield x` is defined as `co_await promise.yield_value(x)`, and `initial_suspend`/`final_suspend` are also awaited implicitly, so they all use the same mechanism.

the `Awaitable` controls whether we actually go into suspension and what gets exectued during. similar to how `promise_type` controls what happens during startup and returning, the `Awaitable` controls what happens whether and when we go in and out of suspension.
```cpp
struct Awaitable
{
    bool await_ready();
    void await_suspend(std::coroutine_handle<promise_type>);
    void await_resume();
};
```
1. `await_ready`: decides whether we go into suspension by returning the readiness flag, i.e. `true` = the result is ready, do not suspend (`await_suspend` is skipped, but `await_resume()` is still called). `false` = suspend.
2. `await_suspend`: executed once the coroutine is already considered suspended, so it is safe to hand its handle to another thread, which may resume it even before `await_suspend` returns. it takes as its argument the `coroutine_handle` of the coroutine that called the `co_await`. it's templated but can also be used without a type (`<>`), s.t. it can refer to any coroutine. if we use the specialization, it means that the awaitable can only be waited upon by coroutines that use a specific `promise_type`. it is possible that we would want to leave open what coroutines `co_await` on our `Awaitable`. it can return:
    - `void`: the coroutine stays suspended, control returns to the caller/resumer.
    - `bool`: `true` = stay suspended, `false` = resume the coroutine immediately.
    - `std::coroutine_handle<>`: stay suspended and resume that coroutine instead (symmetric transfer, see 2.8).
3. `await_resume`: this is the function that gets called **right before** the coroutine retakes the control (also when `await_ready` returned `true`). its return value is the value of the whole `co_await` expression.

### mind blown
`initial_suspend` and `final_suspend` actually return an `Awaitable`:
```cpp
struct std::suspend_always
{
    constexpr bool await_ready() const noexcept { return false; }
    constexpr void await_suspend(std::coroutine_handle<>) const noexcept {}
    constexpr void await_resume() const noexcept {}
};
```
`std::suspend_never` is the same with `await_ready` returning `true`.
### coroutine_handle
defines what we can actually do inside `await_suspend`:
```cpp
template <typename Promise = void>
struct std::coroutine_handle
{
    void resume() const;
    void destroy() const;
    bool done() const;
    Promise& promise() const;    // not available for coroutine_handle<void>
    void* address() const;
    static coroutine_handle from_promise(Promise&);
    static coroutine_handle from_address(void*);
    explicit operator bool() const;
};
```

1. `resume`: it resumes a **suspended** coroutine. calling it on a coroutine that is not suspended (e.g. running or already finished) is undefined behaviour.
2. `destroy`: destroys the coroutine frame of a suspended coroutine. the other way of destroying it is when the coroutine runs to completion *and* `final_suspend` does not suspend. note that `coroutine_handle` is not a smart pointer type, i.e. it won't destroy itself, and copies of it are not tracked.
3. `done`: `true` if the coroutine is suspended at its final suspend point. only valid on a suspended coroutine.
4. `address`/`from_address`: convert to and from a raw pointer, which is useful for passing the handle through C-style callbacks. `coroutine_handle<Promise>` converts implicitly to `coroutine_handle<>`.

## 2.3. order of execution
```cpp
struct ReturnType / std::coroutine_traits<ReturnType, ...>
{
    struct promise_type
    {
        // start -->
        promise_type(T...); // optional
        ReturnType get_return_object();
        std::suspend_always initial_suspend();
        // <--
        // shutdown -->
        void return_value(T); / void return_void();
        void unhandled_exception();
        std::suspend_always final_suspend() noexcept;
        // <--
    };
};
```
we don't always need a nested typedef for the `promise_type`. we can instead specialize the `std::coroutine_traits` tempalted class to point out where the `promise_type` is. this also unlocks some new possibilities, like having a coroutine with `ReturnType = void`.

as soon as the compiler detects a coroutine, it will look at the return type, which in turn defines what the promise type is. the compiler first allocates the coroutine frame (using `operator new`, or a `promise_type::operator new` if provided) and copies the arguments into it (by value, so reference parameters can dangle!). the promise is then constructed; the optional constructor receives the arguments passed when creating the coroutine (`T...`), otherwise the default constructor is used. afterwards, the `get_return_object` will get called which constructs the object returned to the caller. the `initial_suspend` awaitable is then `co_await`ed and after that (if not suspended) the function body starts executing. similarly during shutdown you can either `co_return` a value using `return_value` or fall off the end, which is the same as having an empty `co_return`, in which case the `return_void` will get called. a promise can only have one of `return_void` and `return_value`. `unhandled_exception` is called when we exit with an exception instead of a return value. once the return has been processed, the `final_suspend` is invoked as the last thing before the coroutine is shut down. the locals of the body are destroyed before `final_suspend`, the promise and the frame are only destroyed afterwards.

## 2.4. wake me up inside
back to the example, how do we make the coroutine start executing its body? 
```cpp
struct ReturnType
{
    struct promise_type
    {
        std::suspend_always initial_suspend() noexcept { return {}; }
        ...
    };
    ...
};

...

ReturnType hello_coroutine()
{
    std::println("Hello from coroutine");
    co_return;
}

int main() {
    ReturnType c = hello_coroutine(); // prints nothing
    return 0;
}
```
the coroutine is suspended at `initial_suspend` and waits to be resumed through the `std::coroutine_handle`.

### through the return object
we know that resuming is only possible through the `std::coroutine_handle`. why not do?
```cpp
struct ReturnType
{
    struct promise_type
    {
        ReturnType get_return_object() {
            return ReturnType{
                std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_always initial_suspend() noexcept { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() {}
    };

    std::coroutine_handle<promise_type> handle;
    explicit ReturnType(std::coroutine_handle<promise_type> h) : handle(h) {}
    ReturnType(ReturnType&& o) noexcept : handle(std::exchange(o.handle, {})) {}
    ~ReturnType() { if (handle) handle.destroy(); }
    void resume() { handle.resume(); }
};

int main() {
    ReturnType c = hello_coroutine();
    c.resume(); // prints the message
}
```
it's easy but not mandatory, but must be done in some way so that all elements of the coroutine are connected:
```
             ┌────────┐                           ┌───────────┐
             │        │                           │           │
             │ Caller │                           │ coroutine ├─────────┐
             │        │                           │           │         |
             └────┬───┘                           └─────┬─────┘         |
                  │                                     │               |
                  │                                     │               |
                  │                                     │               |
            ┌─────▼──────┐                        ┌─────▼─────┐         |
            │            │                        │           │         |
  ┌ - - - - | ReturnType ├─────────┐              │ Awaitable │         |
  ❘         │            │         │              │           │         |
  ❘         └────────────┘         │              └─────┬─────┘         |
  ❘                          handle│                    │               |yield_value()
  ❘                                │                    │               |
  ❘get_return_object()             │                    │               |
  ❘                       ┌────────▼─────────┐          │               |
  ❘                   ┌───► coroutine_handle ◄──────────┘               |
  ❘                   │   └────────┬─────────┘  await_suspend()         |
  ❘                   │            │                                    |
  ❘   ::from_promise()│            │                                    |
  ❘                   │            │.promise()                          |
  ❘                   │            │                                    |
  ❘              ┌────┴────┐       │                                    |
  └ - - - - - - -│ promise ◄───────┴────────────────────────────────────┘
                 └─────────┘
```

## 2.5. getting data out of the coroutine
the user must be able to get something out of the coroutine, either through `co_yield` or `co_return`. the coroutine itself only has access to `Awaitable`s.
```cpp
Coroutine f1()
{
    ...
    co_await AwaitableAnswer{420};
}
AwaitableAnswer::AwaitableAnswer(int v) :
    value_(v) {}
```
now the data lives inside the `Awaitable`, but it must reach the caller:
```cpp
struct promise_type
{
    ...
    int value;
}
void AwaitableAnswer::await_suspend(std::coroutine_handle<promise_type> h)
{
    h.promise().value = value_;
}
```
and now the data lives inside the promise, i.e. the central intersection point. on the caller side, we can create an interface such as:
```cpp
struct Coroutine
{
    ...
    int getAnswer() { return handle.promise().value; }

};
int main()
{
    Coroutine c1 = f1();
    c1.resume(); // run until the co_await
    std::println("The answer is {}", c1.getAnswer());
}
```

## 2.6. getting data into the coroutine
the caller might want to put the coroutine to sleep at some point to provide some data to it.
```cpp
Coroutine f1()
{
    ...
    int some_data = co_await OutsideAnswer{};
}

int main()
{
    Coroutine c1 = f1();
    c1.resume(); // run until the co_await
    c1.provide(420);
}
```
the `co_await` call suspends the coroutine and returns control back to main (the coroutine must be started first, otherwise it is still waiting at `initial_suspend`). the `provide` function from the `ReturnType` stores the provided data somewhere inside the `promise_type` and wakes the coroutine. 
```cpp
void Coroutine::provide(int value)
{
    handle.promise().value = value;
    handle.resume();
} 
```
compared to the previous approach where the data was ready at `await_suspend`, `await_resume` receives no arguments, so there is no direct access to the `promise_type` when the coroutine is resumed. that is why the handle must be saved in the awaitable when we are suspending the coroutine:
```cpp
struct OutsideAnswer
{
    bool await_ready() { return false; }
    void await_suspend(std::coroutine_handle<promise_type> h)
    {
        handle = h;
    }
    int await_resume()
    {
        return handle.promise().value;
    }

    std::coroutine_handle<promise_type> handle;
};
```
the `await_resume` function can have any return type. previously we had only used void because it was not a data providing Awaitable.

## 2.7. yield_value
we can now create the fibonacci generator:
```cpp
FiboGenerator make_fibo_gen()
{
    int i1 = 1;
    int i2 = 1;
    for (;;)
    {
        co_await NewNumberAwaitable{i1};
        i1 = std::exchange(i2, i1 + i2);
    }
}
```
which looks ugly. we can make it look more readable by implementing an optional function inside our `promise_type`:
```cpp
NewNumberAwaitable yield_value(int i) {
    return NewNumberAwaitable{i};
}
```
which will let us use `co_yield i1`, which the compiler translates to `co_await promise.yield_value(i1)`. is it necessary to create the new object if we're already inside the `promise_type`? nope:
```cpp
std::suspend_always promise_type::yield_value(int i)
{
    value = i;
    return {};
}
```
instead of defining and constructing an `Awaitable`, we save the value inside the promise right away.

## 2.8. symmetric transfer
when we call `co_await`, we return the control back to the caller/resumer. in our examples so far, this has been the main function, which when necessary resumed the coroutine through the `ReturnType`.

it is also possible to hand control over directly to another **suspended** coroutine. the suspended coroutine's `await_suspend` returns the other's handle and the compiler resumes it as a tail call, i.e. the other coroutine takes the place of the first on the call stack instead of being nested on top of it. this means chains of coroutines resuming each other do not grow the stack. to do this, we need an `Awaitable` that returns the successor coroutine handle (instead of `void`) in its `await_suspend` function:

```cpp
struct promise_type
{
    ...
    std::coroutine_handle<> other;
};
std::coroutine_handle<> Transfer::await_suspend(std::coroutine_handle<promise_type> me)
{
    if (me.promise().other)
        return me.promise().other;
    return std::noop_coroutine();
}

Coroutine f1()
{
    ...
    co_await Transfer{};
}
```
this would hand over control to the other coroutine if available, otherwise `std::noop_coroutine()` is resumed, which does nothing, so control returns to the caller/resumer and `f1` stays suspended. (returning `me` instead would immediately resume `f1` itself.)

further reading: it is possible to have a `ReturnType` that is also an `Awaitable`. Coroutine A calls `co_await` on coroutine B, whose return type is directly awaitable (or has an overloaded `operator co_await` returning an `Awaitable`). its `await_suspend` stores A's `coroutine_handle` (so B can resume A when it finishes, typically by symmetric transfer from `final_suspend`) and returns B's handle to start it, and its `await_resume` returns the result back to A. this is how task types (`Task<T>`) chain coroutines. 

## 2.9. notes
* prefer `std::suspend_always` for the `final_suspend` and destroy the coroutine through its handle (e.g. in the destructor of `ReturnType`, RAII). a suspended coroutine can be inspected with `done()`, while there is no way of checking whether a coroutine that destroyed itself (`final_suspend` returning `std::suspend_never`) is still alive. if `destroy()` is not called and `final_suspend` suspends, the frame is leaked.
* the coroutine parameters are copied into the frame, references and pointers are copied as such. when the coroutine outlives the caller, they dangle. the same holds for lambda captures if the lambda itself is a coroutine.
* `ReturnType` should be move-only, since it owns the handle.
* `co_await` in a `catch` block is not allowed.
* exceptions thrown before the first suspension point inside the body are routed to `unhandled_exception`, but exceptions from `get_return_object` propagate to the caller.

# 3. standard library support
## 3.1. std::generator
in C++23 we get the generator coroutine (`<generator>`, e.g. GCC 14+):
```c++
std::generator<int> get_sequence_generator(int start, int num)
{
    for (int i = start; i < start + num; ++i)
        co_yield i;
}
int main()
{
    auto get = get_sequence_generator(0, 5);
    for (auto v : get) {
        std::print("{} press a key for next value", v);
        std::cin.ignore();
    }
}
```
