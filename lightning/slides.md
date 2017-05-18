<!-- $theme: gaia -->

# Generalized fold expressions

<div align="center">
<img src="velociraptor_fold.jpg" height="350">
</div>

<div align="right">
<font size="4">Photo credit: <a href="https://www.patreon.com/jonakashima">Jo Nakashima</a></font>
</div>

Jackie Kay
:zap: C++ Now 2017 :zap:

---

# C++17 fold expressions are great
```c++
template<bool... Pack>
constexpr auto and_() {
  return (... && Pack);
}
```


---

# Left fold with initial value
Credit: [Bryce's presentation from Tuesday](https://github.com/brycelelbach/cpp17_features/blob/gh-pages/cpp17_features/04_language_fold_expressions/20_print_cpp17_fold.cpp)

```c++
template <typename... Ts>                          
void print(Ts&&... ts)
{
  (std::cout << ... << std::forward<Ts>(ts)) << "\n";
}

```

---

# With comma operator

```c++
template<typename F, typename... Ts>
void for_each(F&& f, Ts&&... ts) {
  ([&f, &ts]() {
    f(ts);
  }, ...);
}
```
---

# What about return values?

---

# Generalized fold one-liner
Credit: [Barry Revzin on Stack Overflow](http://stackoverflow.com/questions/43499015/uninitialized-captured-reference-error-when-using-lambdas-in-fold-expression)

```c++
template <typename F, typename Z, typename... Xs>
auto fold_left(F&& f, Z acc, Xs&&... xs) {
  ((acc = f(acc, xs)), ...);
  return acc;
}
```
---
# Works with constexpr too

```c++
template <typename F, typename Z, typename... Xs>
constexpr auto fold_left(F&& f, Z acc, Xs&&... xs) {
  ((acc = f(acc, xs)), ...);
  return acc;
}

auto sum = [](auto x, auto y){ return x + y; };

static_assert(fold_left(sum, 1, 2, 3) == 6);
```
 
---

# Gotcha

Only works if return type of all intermediate operations is uniform. Consider this case:

```c++
auto concatenated_tuple = fold_left(
    [](auto&& x, auto&& y) {
      return std::tuple_cat(x, y);
    },
    std::make_tuple("hello world"),
    std::make_tuple(1, 2, 3),
    std::make_tuple(std::vector<float>{})
);
```

---

# Possible solution

```c++
template<typename F, typename X>
struct fold_wrapper {
  F f;
  X state;

  template<typename Arg>
  constexpr auto operator>>=(Arg&& arg) {
    auto result = f(state, arg.state);
    return fold_wrapper<F, decltype(result)>{f, result};
  }
};
template <typename F, typename... Xs>
constexpr auto fold_left(const F& f, Xs&&... xs) {
  auto result = (... >>= fold_wrapper<F, Xs>{f, xs});
  return result.state;
}
```
