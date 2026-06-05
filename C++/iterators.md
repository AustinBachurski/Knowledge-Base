# Iterators

*Definitions and concepts sourced from [cppreference.com](https://en.cppreference.com/).*

## Template

> `std::iterator_category<I>` is the concept taking your iterator as template parameter `I`, where as `std::iterator_category_tag` is the iterator trait tag.<br>
 
- Replace `:TYPE:` with the iterable type.<br>
- Replace `:ITERATOR_CATEGORY:` with the std::iterator_category.<br>

```cpp
export template <typename T> class :TYPE:Iterator {
public:
  // Iterator Traits
  using iterator_category = std:::ITERATOR_CATEGORY:_tag;
  using value_type = std::remove_cv_t<T>;
  using difference_type = std::ptrdiff_t;
  using pointer = T *;
  using reference = T &;

  constexpr :TYPE:Iterator() = default;
  constexpr explicit :TYPE:Iterator(pointer ptr) : ptr_{ptr} {}

private:
  pointer ptr_{};
};

// Ensures that the iterator meets the
// requirements for the appropriate iterator tag.
static_assert(std:::ITERATOR_CATEGORY:<:TYPE:Iterator<int>>);
```

## Iterator Categories

> Each tag adds further requirements that build on the lesser tags.

| Iterator | Operators | Details |
|-|-|-|-|
| `input_or_output_iterator` | `++`, `*`                                         | The `input_or_output_iterator` concept forms the basis of the iterator concept taxonomy; every iterator type satisfies the `input_or_output_iterator` requirements. |
| `input_iterator`           | `++`, `*`                                         | The `input_iterator` concept is a refinement of `input_or_output_iterator`, adding the requirement that the referenced values can be read (via `indirectly_readable`) and the requirement that the iterator concept tag be present. ***Clarification:*** *`input_or_output_iterator` doesn't require `operator*` to be readable (think `std::back_inserter`), where as `input_iterator` requires readable.* |
| `forward_iterator`         | `++`, `*`, `==`                                   | This concept refines `input_iterator` by requiring that the iterator also models `incrementable` (thereby making it suitable for multi-pass algorithms), and guaranteeing that two iterators to the same range can be compared against each other. ***Clarification:*** *previous iterators modeled `weakly_incrementable` which does not guarantee that two copies of an iterator `a` and `b` will be equal if they are both incremented (think of a stream).* |
| `bidirectional_iterator`   | `++`, `*`, `==`, `--`                             | The concept `bidirectional_iterator` refines `forward_iterator` by adding the ability to move an iterator backward. |
| `random_access_iterator`   | `++`, `*`, `==`, `--`, `+=`, `-=`, `+`, `-`, `[]` | The concept `random_access_iterator` refines `bidirectional_iterator` by adding support for constant time advancement with the +=, +, -=, and - operators, constant time computation of distance with -, and array notation with subscripting []. ***Clarification:*** *see the concept below, spells it out very clearly.* |
| `contiguous_iterator`      | `++`, `*`, `==`, `--`, `+=`, `-=`, `+`, `-`, `[]` | The `contiguous_iterator` concept refines `random_access_iterator` by providing a guarantee that the denoted elements are stored contiguously in the memory. |

## Concept for `std::random_access_iterator`

```cpp
template< class I >
    concept random_access_iterator =
        std::bidirectional_iterator<I> &&
        std::derived_from</*ITER_CONCEPT*/<I>, std::random_access_iterator_tag> &&
        std::totally_ordered<I> &&
        std::sized_sentinel_for<I, I> &&
        requires(I i, const I j, const std::iter_difference_t<I> n) {
            { i += n } -> std::same_as<I&>;
            { j +  n } -> std::same_as<I>;
            { n +  j } -> std::same_as<I>;
            { i -= n } -> std::same_as<I&>;
            { j -  n } -> std::same_as<I>;
            {  j[n]  } -> std::same_as<std::iter_reference_t<I>>;
            // Clarification on operator[]: std::iter_reference_t<I> isn't
            // talking about the iterator itself, it's expecting a reference to
            // `T`, so j[n] should return the same type as the `reference`
            // typedef at the top of your iterator class.
```
