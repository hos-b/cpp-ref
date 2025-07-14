# range
a range is a pair of begin and end iterators:
  - they need to be comparable but not necessarily of the same type
  - technically an iterator and a sentinel
  - (almost) all standard library containers are ranges

## range algorithms
- are like STL algoirthms but better
- execute immediately: iteration is controlled internally by the algorithm
- return value improved compared to STL
- specified with concepts
- projection parameters: predicate independent of the algorithm being executed

- query
  - `find`, `find_if`, `find_if_not`
  - `find_last`, `find_last_if`, `find_last_if_not` (C++23)
  - `adjacent_find`
  - `any_of`, `all_of`, `none_of`
  - `contains`, `contains_subrage` (C++23)
  - `is_partitioned`
  - `is_sorted`, `is_sorted_until`
  - `lower_bound`, `upper_bound`, `parition_point`
  - `clamp`
  - `mismatch`
  - `starts_with`, `ends_wtih` (C++23)
- sampling
  - `copy`, `copy_if`, `copy_n`
  - `rotate_copy`, `unique_copy`
  - `stride`, `sample`, `take`
- modifiers
  - `merge`, `inplace_merge`
  - `move`, `move_backward`
  - `partition`, `partition_copy`
  - `replace`, `replace_if`, `replace_copy`, `replace_copy_if`
  - `remove`, `remove_if`, `remove_copy`, `remove_copy_if`
  - `reverse`
  - `shuffle`, `sort`
  - `shift_left`, `shift_right` (C++23)
  - `transform`, `for_each`, `for_each_n`
  - `unique`
  - `uninitialized_value_construct`, `uninitialized_copy`
- calculation
  - `count`, `count_if`
  - `fold_left`, `fold_left_first`, `fold_right`, `fold_right_last` (C++23)
  - `min`, `max`, `minmax`

### example sort with projection
instead of taking two elements of the underlying datatype and doing the comparison directly, we simply set the projection parameter
```cpp
struct stuff {
    int idx = 0;
    std::string s;
};
std::vector<stuff> vstuff = {{2, "foo"}, {1, "bar"}, {0, "baz"}};
ranges::sort(vstuff, std::less<>{}, [](auto const& val) {return val.idx});
```

## improved return values
ranges was built on the principle of not throwing away calculation. many range algorithms return a struct with iterator + information.

### find_last example
```cpp
std::string path = "/foo/bar/baz";
auto leaf = rng::find_last(path, "/") | std::views::drop(1); // `/baz` to `baz`
std::print("{}\n", std::string_view(leaf)); //baz

path = "none";
leaf = rng::find_last(path, "/") | std::views::drop(1)
std::print("empty: {}\n", std::string_view(leaf));
```
# view
a view is a non-owning range with lazy evaluation. all methods are O(1). they allow for compistion of several processing steps.
since iterations is externally driven, one can have "infinite views". it mostly delays computation until the iteration:

```cpp
std::vector<int> vi{0, 1, 2, 3, 4, 5};
auto is_even = [](int i){return i%2 == 0;};
// it only finds the first element that satisfies the predicate
ranges::filter_view evens(vi, is_even);

for (int i : evens) {
    std::print("{}\n", i);
}
```

## view algorithms
- modifiers
  - `join_view`, `join_with` (C++23)
  - `split_view`, `lazy_split_view`
  - `reverse_view`, `transform_view`
  - `cartesean_product` (C++23)
  - `zip`, `zip_transform` (C++23)
- sample
  - `drop_view`, `drop_while_view`
  - `take_view`, `take_while_view`
  - `filter_view`, `stride` (C++23)
  - `chunk`, `chunk_by` (C++23)
- adapters
  - `istream_view`
  - `keys_view`, `values_view`
  - `ref_view`, `all_view`
  - `enumerate` (C++23)
- factories
  - `iota_view`, `single_view`, `empty_view`
  - `repeat`
- convert
  - `as_const`, `as_rvalue` (C++23)

## views and const iteration
you should not pass ranges/views by `const&`. some of them like `filter_view` cache.
either pass by value or using forwarding reference.

### cartesean product example
```cpp
namespace rv = std::ranges::views;
std::vector<int> v1{0, 1};
std::vector<int> v2{2, 3};
for (auto && [a, b] : rv::cartesean_product(v1, v2))
    std::print("{} {}\n", a, b);
// output:
// 0 2
// 0 3
// 1 2
// 1 3
```
### chunk_by example
```cpp
std::vector<int> vi{1, 2, 2, 3, 0, 4, 5, 2};
// [[1, 2, 2, 3], [0, 4, 5], [2]]
std::print("{}\n", v | std::views::chunk_by(ranges::less_equal{}));
```
### zip and pairwise
```cpp
std::vector v1{1, 2};
std::vector v2{'a', 'b', 'c'};
std::vector v3{3, 4, 5};

// {(1, 'a'), (2, 'b')}
std::print("{}\n", std::views::zip(v1, v2));
// {3, 8}
std::print("{}\n", std::views::zip_transform(std::multiplies(), v1, v3));
// {('a', 'b'), ('b', 'c')}
std::print("{}\n", v2 | std::views::pariwise);
// {7, 9}
std::print("{}\n", v3 | std::views::pariwise_transform(std::plus()));
```

### sum of squares
```cpp
namespace rng = std::ranges;
namespace rv = rng::views;
auto square = [](int x) { return x*x; };
auto add    = [](int x, int y) { return x+y; };

// again, no computation is done here
auto square_view = rv::iota(0)
                 | rv::transform(square)
                 | rv::take(5);
// [(0, 0), (1, 1), (2, 4), (3, 9), (4, 16)]
std::print("{}\n": sqaure_view | rv::enumerate);
int total = rng::fold_left(sqaure_view, 0, add);
// 30
std::print("{}\n": total);
```
### adjacent (C++23)
```cpp
std::vector v{1, 2, 3, 4};
auto r3 { v | std::views::adjacent<2>}; // {(1, 2), (2, 3), (3, 4)}
```

# range adaptors
these are algorithms that transform a `range` into a `view` with custom behaviour.
adaptors support the pipe operator for composition. range adaptors are declared in the `std::ranges::views` namespace.
same example with filter adaptors:
```cpp
auto square = [](int i) {return i*i;};
for (int i : vi | rng::views::filter(is_even) || rng::views::transform(square)) {
    std::print("{}\n", i);
}
```

## view and range algorithm composition

```cpp
auto print_int = [](int i) { std::print("{} ", i); }
auto is_even = [](int i) { return i%2 == 0 };

std::vector<int> vi{6, 2, 3, 4, 5, 6};
// view is defined, no calculation performed
auto after_leading_evens = rng::views::drop_while(v, is_even);

rng::for_each(after_leading_evens, print_int);
```

## std::span
a span is a non-owning view on top of a container. basically a pointer and a size.
```cpp
void print_reverse(std::span<int> si) {
    for (auto i : std::ranges::reverse_view(si))
        std::print("{} ", i);
    std::print("\n");
}

std::vector vi = {1, 2, 3, 4, 5};
print_reverse(vi);
std::span<int> si(vi);
print_reverse(si.first(2));
print_reverse(si.last(2));
```

## converting from a view to a collection (C++23)
views are non-owning. if we want to save the result to another collection, we need `std::ranges::to`:
```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};
auto squared = numbers | ranges::views::transform([](int n) { return n*n; })
                       | ranges::to<std::deque>();
std::print("{}\n", squared);
```

this also has makes container conversions much easier:
```cpp
std::vector<int> vec{11, 22, 33};

auto set1 { std::ranges::to<std::set>() };
auto set2 = vec | std::ranges::to<std::set<double>>();
std::set<double> set3 {std::from_range, vec};
```

or splitting:
```cpp
std::string lorem{"lorem ipsum dolor sit amet"};
auto words = lorem
            | std::views::split(' ')
            | std::views::transform([](const auto& v) {return std::string(std::from_range, v)})
            | std::ranges::to<std::vector>();
```

### another example with join
```cpp
namespace rv = std::ranges::views;
std::vector<std::string> vs = {"foo", "bar", "baz", ""};

std::string s = vs | rv::join(std::string_view(" -- "))
                   | std::ranges::to<std::string>();

auto is_not_empty = [](const std::string& s) { return !s.empty(); };

s = vs | rv::filter(is_not_empty)
       | rv::join('-')
       | std::ranges::to<std::string>();
```

# more to come
### range constructors and operations (C++23)
collections get range constructors and other range-based operations such as `insert_range`,
`assign_range`, `append_range`, `prepend_range`.
### custom views (C++23)
it is now possible to write you own views, albeit tedious. improvements may be coming in C++26.
