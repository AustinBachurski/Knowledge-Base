# Deducing `this`, `auto`, & `decltype(auto)`

[Back to README.md](../README.md)

*Definitions and concepts sourced from [cppreference.com](https://en.cppreference.com/).*

#### In page links
- [Deducing `this`](#deducing-this)<br>
- [auto-returning-functions](#auto-returning-functions)<br>
- [decltype(auto)](#decltypeauto)<br>

# Deducing `this`

[Back to Top](#deducing-this-auto--decltypeauto)

Deducing `this` allows your methods to determine the constness of `this` to reduce code duplication for things like `.at()`.<br>
Use `template <typename Self>` and include the first parameter `this Self &&self`.<br>
Since we're deducing constness, the method cannot be marked as `const`.<br>
Must consider the return type deduction rules if using `auto`!<br>

# `auto`-returning Functions 

[Back to Top](#deducing-this-auto--decltypeauto)

`auto` discards references and top-level cv-qualifiers and references, meaning:

```cpp
auto func() { 
  static constexpr auto value{42};
  int const *const x{&value};
  return x; // The return type of func is `int const *`!
}

auto ref_func() {
  static constexpr auto value{42};
  const int &x{value};
  return x; // The return type of ref_func is `int`!
}
```

See template type deduction rules and `std::decay` - the member typedef type is `std::remove_cv<std::remove_reference<T>::type>::type`.

Also see standards draft [expr.type]
- *If an expression initially has the type “reference to `T`” ([dcl.ref], [dcl.init.ref]), the type is adjusted to `T` prior to any further analysis; the value category of the expression is not altered.  Let `X` be the object or function denoted by the reference.  If a pointer to `X` would be valid in the context of the evaluation of the expression ([basic.fundamental]), the result designates `X`; otherwise, the behavior is undefined.*

If you need the reference and/or the cv-qualifiers to be retained, use a return type of `decltype(auto)`.

# `decltype(auto)`

[Back to Top](#deducing-this-auto--decltypeauto)

Defining the return value of a function as `decltype(auto)` will deduce the type with reference and cv-qualifiers.

```cpp
  template <typename Self>
  [[nodiscard]] constexpr auto at(this Self &&self, size_type position)
      -> decltype(auto) {
    if (position >= self.size_) {
      throw std::out_of_range(
          std::format("Vector Range Check: position (which is {}) >= "
                      "this->size() (which is {})",
                      position, self.size_));
    }
    return std::forward_like<Self>(self.data_[position]);
  }
```

>Just remember that `std::forward_like` returns a reference!

- if the value category of expression is xvalue, then decltype yields `T&&`.
- if the value category of expression is lvalue, then decltype yields `T&`.
- if the value category of expression is prvalue, then decltype yields `T`.

Note that if the name of an object is parenthesized, it is treated as an ordinary lvalue expression, thus `decltype(x)` and `decltype((x))` are often different types.
>Remember the bit about a reference type being adjusted to `T` prior to any further analysis?  Wrapping something in `()` makes it an expression.
