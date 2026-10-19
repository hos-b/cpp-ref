# Coroutines
given the impossible "user" API, it needs a separate file.
C++20 implements the _stackless_ coroutine: when suspended, control is returned to the caller (or whoever resumed the coroutine) and the local state is kept in a separate coroutine frame, which is typically heap-allocated (the allocation can be elided by the compiler). all the facilities live in the `<coroutine>` header. this is mostly a summary of Andreas Weis cppcon talk.

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
this will suspend the current coroutine (the keywords are only usable within coroutines) and return control to its caller/resumer while the read is in flight. as soon as the data becomes available, the coroutine is resumed and the execution continues right after the `co_await` expression, which evaluates to the result of the read.

## 0.2. suspend/resume
we'd rather have:
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
coroutines should be thought of as factory functions that produce the actual coroutine.

# 1. Intro
the initial call to the coroutine function returns an object of `ReturnType` to the caller. the interface of `ReturnType` is going to determine what the coroutine can do. the end user will only ever need to understand and deal with this object.

a function is a coroutine if its body contains one of the following keywords:
- `co_return`
- `co_yield`
- `co_await`

as soon as the compiler detects this, it changes the rules of how the code inside the body works. reading it as a typical function will not make sense.
the following cannot be coroutines: `main`, constructors/destructors, `constexpr` functions, and functions with a deduced return type (`auto`) or using `return` (use `co_return` instead). the coroutine's parameters are copied into the coroutine frame, so reference parameters can dangle if the referenced objects die before the coroutine finishes.

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
#include <coroutine>

struct ReturnType
{
    struct promise_type{};
};

ReturnType hello_coroutine() {
    co_return;
}
```
this doesn't actually compile yet, because the compiler needs a few functions to be defined by the `promise_type`.

## 2.1. promise_type
`std::future` and `std::promise` are part of the C++ threading library. future is what is handed out to the caller, so that it can retrieve the result. the future has the role of the `ReturnType` in coroutines.
we also need something that the callee can put the value in, which is the promise type for threads. the coroutine promise has a similar _idea_ behind it. this object remains inside the coroutine.

unlike the threading object pair, where the producer constructs the promise on its own, the coroutine promise object is constructed automatically by the compiler inside the coroutine frame. it is the central intersection point for the coroutine code and the caller code. it determines what happens in the essential timepoints during a coroutine's lifetime, namely what happens at the startup and the completion of the coroutine and how various return values are handled, when the coroutine reaches the end of its control flow.

the compiler expects the following functions to be inside the promise type (`return_void` and `return_value` are alternatives, only one of them may exist):
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
1. `get_return_object`: has the same return type as the coroutine body. when the coroutine starts executing, a compiler implicitly calls this function. it is now up to the `promise_type` to construct the return object that will be handed back to the caller (in the basic example, `FiboGenerator` would be constructed here). the result is stored by the compiler in a local object.

    when the coroutine reaches its first suspension point (the `initial_suspend`, if it suspends, or a later `co_await`/`co_yield`) or finishes, the control flow returns to the caller, and the caller receives this object as the return value.

2. `return_void`: decides what happens when we reach an empty `co_return` statement or fall off the end of the body (which is undefined behaviour if the promise has no `return_void`). there's also a corresponding `return_value` for when the `co_return` is used in conjunction with a value, but in our example, that is not done.

3. `unhandled_exception`: called from a compiler-generated `catch` block when an exception escapes the coroutine body. it can store the exception (`std::current_exception()`), rethrow it (`throw;`) or terminate. afterwards the coroutine proceeds to `final_suspend`.

4. `initial_suspend`
5. `final_suspend`: these two are the customization points that allow us to execute some code, when the coroutine first starts executing (initial_suspend) and shortly before when the coroutine stops executing for good (final_suspend), e.g. if we return from the coroutine or the control flow falls off the end or if we exit via an exception.

`std::suspend_always` tells the compiler, that we always want to suspend at that particular point (intial, final) and hand the control flow back to the caller.
`std::suspend_never` lets the coroutine continue executing without relinquishing control.
`final_suspend` must be `noexcept`.

**Note**: `std::suspend_never` on `final_suspend()` doesn't mean that the coroutine will just run forever in a detatched state. it rather means that the coroutine will not go into suspension and that it will destroy its own frame automatically. if not chosen, we'd need to manually read back any result the coroutine might have, if any, and then destroy it manually.

back to the example: the code can actually compile now:
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
if we change the `initial_suspend` declaration from `suspend_always` to `suspend_never`, then we see the message on the standard output.

## 2.2. pausing and awaitables
An `awaitable` is a type that can be the operand of `co_await`.
```cpp
AsyncRead awaitable = async_read(socket, buffer);
auto [ec, bytes_read] = co_await awaitable;
```
calling `co_await` is an opportunity for suspension. it is the main operator that explicitly lets us pause the coroutine and return control flow back to the caller. `co_yield x` is also a suspension point (it is defined as `co_await promise.yield_value(x)`), and `initial_suspend`/`final_suspend` are implicitly awaited by the compiler. the operand may be customised via `promise_type::await_transform` or an `operator co_await` overload.

the `awaitable` controls whether we actually go into suspension and what gets executed during it. similar to how `promise_type` controls what happens during startup and returning, the `awaitable` controls what happens whether and when we go in and out of suspension.
```cpp
struct Awaitable
{
    bool await_ready();
    void await_suspend(std::coroutine_handle<promise_type>);
    void await_resume();
};
```
1. `await_ready`: decides whether we go into suspension by returning a readiness flag, i.e. `true` = the result is ready, do not suspend (`await_suspend` is skipped and `await_resume()` is called right away).
2. `await_suspend`: executed once the coroutine is considered suspended, so it is safe to hand over the coroutine handle to another thread (which may resume it even before `await_suspend` returns!). it takes as its argument the `coroutine_handle` of the coroutine that called the `co_await`. it may return `void` (stay suspended, return control back to the caller), `bool` (`true` = stay suspended, `false` = resume immediately) or a `coroutine_handle` (resume that coroutine instead, see symmetric transfer). it's templated but can also be used without a type (`<>`), s.t. it can refer to any coroutine. if we use the specialization, it means that the awaitable can only be waited upon by coroutines that use a specific `promise_type`. it is possible that we would want to leave open what coroutines `co_await` on our `awaitable`.
3. `await_resume`: this is the function that gets called **right before** the coroutine retakes the control (also when `await_ready` returned `true`). its return value is the value of the whole `co_await` expression.

### coroutine internals
`initial_suspend` and `final_suspend` must actually return an `awaitable` instance:
```cpp
struct std::suspend_always
{
    constexpr bool await_ready() const noexcept { return false; }
    constexpr void await_suspend(std::coroutine_handle<>) const noexcept {}
    constexpr void await_resume() const noexcept {}
};
```
`std::suspend_never` is the same, except that `await_ready` returns `true`.
the entire coroutine body can be thought of as:
```cpp
{
  // allocate frame, construct promise, create return object
  co_await promise.initial_suspend();   // implicit
  try {
    /* explcitly written coroutine body */
  } catch (...) {
    promise.unhandled_exception();
  }
  co_await promise.final_suspend();     // implicit, must not throw
}
```
the `co_await` on the other hand resembles:
```cpp
if (!a.await_ready()) {
    // coroutine is now suspended
    auto r = a.await_suspend(handle_of_this_coroutine);
    // r == void               -> return to the caller/resumer
    // r == bool               -> true: return to resumer; false: resume self immediately
    // r == coroutine_handle<> -> resume THAT coroutine (a tail call)
}
return a.await_resume();   // after being resumed
```
### comparison to python
an awaitable and a coroutine may be preceived as synonymous in python, but it also has both. a python awaitable is any object that defines the `__await__` method. coroutines are implicitly awaitable. async futures are awaitable but not callable:
```py
loop = asyncio.get_running_loop()
fut = loop.create_future()
loop.call_later(1, fut.set_result, "done")
await fut
```
in C++, we can also make a coroutine an awaitable by defining the three functions mentioned above. see symmetric transfer.

### coroutine_handle
defines what we can actually do inside `await_suspend`:
```cpp
template <typename Promise>
struct std::coroutine_handle
{
    void resume() const;
    void destroy() const;
    bool done() const; // true if suspended at the final suspend point
    Promise& promise() const;
    static coroutine_handle from_promise(Promise&);
    void* address() const;
    explicit operator bool() const;
};
```

1. `resume`: it resumes a **suspended** coroutine. resuming a coroutine that is not suspended (or has finished) is undefined behaviour.
2. `destroy`: destroys the coroutine frame. the other way the frame goes away is when the coroutine runs to completion and `final_suspend` does not suspend. note that `coroutine_handle` is not a smart pointer type, i.e. it won't destroy itself, and copies of the handle dangle after the frame is destroyed.
3. `done`: only valid on a suspended coroutine; `true` when it is suspended at on `final_suspend()`.

`std::coroutine_handle<>` (i.e. `coroutine_handle<void>`) is the type-erased handle; any `coroutine_handle<P>` converts to it, but it has no `promise()`.

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
we don't always need a nested type definition for the `promise_type`. we can instead specialize the `std::coroutine_traits` tempalted class to point out where the `promise_type` is. this also unlocks some new possibilities, like having a coroutine with `ReturnType = void`.

as soon as the compiler detects a coroutine, it will look at the return type, which in turn defines what the promise type is (`std::coroutine_traits<ReturnType, Args...>::promise_type`, which defaults to `ReturnType::promise_type`). the coroutine frame is allocated, the parameters are copied into it and the promise is constructed. if the promise has a constructor taking the coroutine's parameters (`T...`, passed as lvalues), that one is used, otherwise the default constructor is. afterwards, the `get_return_object` will get called which constructs the object returned to the caller. the initial suspend is then called and after that the coroutine is started and the function body starts executing. simlarly during shutdown you can either `co_return` a value using `return_value` or fall off the end, which is the same as having an empty `co_return`, in which case the `return_void` will get called. you cannot have both in the same promise type. `unhandled_exception` is called when we exit with an exception instead of a return value. once the return has been processed, the `final_suspend` is invoked as the last thing before the coroutine is shut down. 

## 2.4. wake me up inside
back to the example, how do we make the coroutine start executing its body? 
```cpp
struct ReturnType
{
    struct promise_type
    {
        std::suspend_always initial_suspend() { return {}; }
        ...
    }
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

### through the return object
we know that _explicit_ resuming is only possible through the `std::coroutine_handle::resume()`. why not do:
```cpp
struct promise_type
{
    ReturnType get_return_object() {
        return ReturnType{
            std::coroutine_handle<promise_type>::from_promise(*this)
        };
    }
};

struct ReturnType
{
    std::coroutine_handle<promise_type> handle;
    ReturnType(std::coroutine_handle<promise_type> h) : handle(h) {}
    void resume() { handle.resume(); }
};
```
a complete, working version (the `ReturnType` owns the frame and destroys it, since our `final_suspend` suspends):
```cpp
struct ReturnType
{
    struct promise_type
    {
        ReturnType get_return_object() {
            return ReturnType{std::coroutine_handle<promise_type>::from_promise(*this)};
        }
        std::suspend_always initial_suspend() noexcept { return {}; }
        std::suspend_always final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };

    explicit ReturnType(std::coroutine_handle<promise_type> h) : handle(h) {}
    ReturnType(ReturnType&& o) noexcept : handle(std::exchange(o.handle, {})) {}
    ~ReturnType() { if (handle) handle.destroy(); }
    void resume() { handle.resume(); }

    std::coroutine_handle<promise_type> handle;
};
```
there's no prescribed pattern of defining the coroutine return type, but it must be done in a way so that all elements of the coroutine are connected:
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
the user must be able to get something out of the coroutine, either through `co_yield` or `co_return`. the coroutine body has no direct access to its promise, so it communicates through `Awaitable`s (or `co_yield`/`co_return`, which go through the promise).
```cpp
ReturnType f1()
{
    ...
    co_await AwaitableAnswer{420};
}
AwaitableAnswer::AwaitableAnswer(int v) :
    value_(v) {}
```
now the data lives inside the `Awaitable`, but it must reach the caller:
```cpp
struct ReturnType
{
    struct promise_type
    {
        ...
        int value;
    }
}
...
void AwaitableAnswer::await_suspend(std::coroutine_handle<ReturnType::promise_type> h)
{
    h.promise().value = value_;
}
```
and now the data lives inside the promise, i.e. the central intersection point. on the caller side, we can create an interface such as:
```cpp
struct ReturnType
{
    ...
    int getAnswer() { return handle.promise().value; }

};
int main()
{
    ReturnType c1 = f1();
    c1.resume(); // runs the body up to the co_await (initial_suspend suspends)
    std::println("The answer is {}", c1.getAnswer());
}
```

## 2.6. getting data into the coroutine
the caller might want to put the coroutine to sleep at some point to provide some data to it.
```cpp
ReturnType f1()
{
    ...
    int some_data = co_await OutsideAnswer{};
}

int main()
{
    ReturnType c1 = f1();
    c1.resume();      // run until the co_await
    c1.provide(420);  // store the value and resume
}
```
the `co_await` call suspends the coroutine and returns control back to main. the `provide` function from the `ReturnType` stores the provided data somewhere inside the `promise_type` and wakes the coroutine. 
```cpp
void ReturnType::provide(int value)
{
    handle.promise().input_value = value;
    handle.resume();
} 
```
compared to the previous approach where the data was ready at `await_suspend`, there is no direct access to the `promise_type` when the coroutine is resumed. that is why the handle must be saved when we are suspending the coroutine:
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
        return handle.promise().input_value;
    }

    std::coroutine_handle<promise_type> handle;
};
```
the `await_resume` function can have any return type. previously we had only used void because it was not a data providing awaitable.

## 2.7. yield_value
we can now create the fibonacci generator:
```cpp
FiboGenerator makeFiboGenerator()
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
that lets us use `co_yield i1`, which the compiler translates to `co_await promise.yield_value(i1)`. is it necessary to create the new type if we're already inside the `promise_type`? nope:
```cpp
std::suspend_always promise_type::yield_value(int i)
{
    value = i;
    return {};
}
```
instead of defining and constructing an `awaitable`, we save the value inside the promise right away.

# 3. symmetric transfer
when we call `co_await`, we _usually_ return the control back to the caller if `await_ready()` returns false. in our examples so far, this has been the main function, which when necessary resumed the coroutine through the `ReturnType`.

it is also possible to hand control over to another **suspended** coroutine. instead of returning to the caller, the first coroutine suspends and the second one is resumed as a tail call, so the stack does not grow no matter how many transfers happen. resuming through `resume()` inside `await_suspend` would nest the calls and could overflow the stack. to do this, we need an `awaitable` that returns the successor coroutine handle (instead of `void`) in its `await_suspend` function:

```cpp
struct Coroutine
{
    struct promise_type
    {
        ...
        std::coroutine_handle<> other;
    };
}
...
std::coroutine_handle<> Transfer::await_suspend(std::coroutine_handle<promise_type> me)
{
    return me.promise().other ? me.promise().other : std::noop_coroutine();
}

Coroutine f1()
{
    ...
    co_await Transfer{};
}
```
this would hand over control to the other coroutine if available, otherwise it returns `std::noop_coroutine()`, a handle whose `resume()` does nothing, so control goes back to the caller/resumer as with a plain suspension. reutrning `std::noop_coroutine()` is behaviorally the same as having `void` as a return type but with the flexibility of actually resuming another coroutine. returning `me` instead is also legal, but it immediately resumes the same coroutine.

## 3.1. huge example
```cpp
struct Task {
    struct promise_type {
        std::coroutine_handle<> continuation;   // who is waiting for me
        Task get_return_object() { return Task{std::coroutine_handle<promise_type>::from_promise(*this)}; }
        std::suspend_always initial_suspend() { return {}; }   // lazy start
        auto final_suspend() noexcept {
        struct F {
            bool await_ready() noexcept { return false; }
            std::coroutine_handle<> await_suspend(std::coroutine_handle<promise_type> h) noexcept {
                return h.promise().continuation;       // resume the awaiter (symmetric transfer)
            }
            void await_resume() noexcept {}
        };
            return F{};
        }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
  };

  std::coroutine_handle<promise_type> h;

  // Task is itself an awaitable:
  bool await_ready() { return false; }
  std::coroutine_handle<> await_suspend(std::coroutine_handle<> caller) {
        h.promise().continuation = caller;   // remember the caller
        return h;                            // start the child now
  }
  void await_resume() {}
};

Task child();
Task parent() { co_await child(); }   // coroutine awaiting a coroutine
```

## 2.9. notes
* prefer `std::suspend_always` for the `final_suspend` and destroy the coroutine through its handle (e.g. in the destructor of `ReturnType`, RAII). this keeps the frame alive so that `done()` and the promise (results, exceptions) can still be inspected. there is no way of checking whether a frame was already destroyed, so the handle must have a single owner. if `final_suspend` does not suspend, the frame is destroyed automatically on completion and every copy of the handle dangles. falling off the end of the body does not free the frame when `final_suspend` suspends.
* never `resume()` a coroutine that is not suspended or is done.

# 4. standard library support
## 4.1. std::generator
in C++23 we get the generator coroutine (`<generator>`):
```c++
#include <generator>
#include <iostream>
#include <print>

std::generator<int> get_sequence_generator(int start, int num)
{
    for (int i = start; i < start + num; ++i)
        co_yield i;
}
int main()
{
    auto gen = get_sequence_generator(0, 5);
    for (auto v : gen) {
        std::print("{} press a key for next value", v);
        std::cin.ignore();
    }
}
```
