# Allocator Comparisons & Propagation

[Back to README.md](../README.md)

*Definitions and concepts sourced from [cppreference.com](https://cppreference.com/cpp/named_req/Allocator).*

#### In page links
- [Comparing Allocators](#comparing-allocators)<br>
- [Construction](#construction)<br>
- [Propagate on Container Copy Assignment](#propagate-on-container-copy-assignment)<br>
- [Propagate on Container Move Assignment](#propagate-on-container-move-assignment)<br>
- [Propagate on Container Swap](#propagate-on-container-swap)<br>

## Comparing Allocators

[Back to Top](#allocator-comparisons--propagation)

When two allocators `a1` and `a2` compare equal, storage allocated by `a1` can be deallocated through `a2` and vice versa.
When the two allocators do not compare as equal, storage allocated by `a1` cannot be managed by `a2`.

## Construction

[Back to Top](#allocator-comparisons--propagation)

- `Allocator a1(a2);` and `Allocator a1 = a2;`
  - copy-constructs `a1` such that `a1 == a2`.  Allocators must satisfy `std::copy_constructible<Allocator>`.
  - Since `a1 == a2` this means that creating a temporary local to allocate a buffer in order to swap contents before modifying the original in order to maintain strong exception safety IS VALID if the source allocator is assigned to the temporary after allocation and swapping.

## Propagate on Container Copy Assignment

[Back to Top](#allocator-comparisons--propagation)

**`propagate_on_container_copy_assignment:`**

- `std::true_type` or derived from it if the allocator of type `A` needs to be copied when the container that uses it is copy-assigned.
- If this member is `std::true_type` or derived from it, then `A` must satisfy *CopyAssignable* and the copy operation must not throw exceptions.
- Note that if the allocators of the source and the target containers do not compare equal, copy assignment has to deallocate the target's memory using the old allocator and then allocate it using the new allocator before copying the elements (and the allocator).

```cpp
    if constexpr (std::allocator_traits<Allocator>::
                      propagate_on_container_copy_assignment::value) {
      // If propagate is true, we must copy the allocator from source to 
      // destination.

      if (destination_allocator != source_allocator) {
        // If allocators do not compare as equal, the source allocator cannot
        // manage the memory allocated by the destination allocator.  You must
        // allocate new memory for the elements in source with a copy of the
        // source allocator.  Since copied allocators are guaranteed to compare
        // equal to one another, you can safely create a local temporary
        // allocator and use that temporary allocator to allocate a new buffer
        // that can be used to copy the elements from source while preserving
        // strong exception safety guarantees.  Once data has been safely copied
        // into the new buffer, you can use the destination allocator to destroy
        // the objects that originally lived in destination's allocated memory
        // and then free destination's memory.  Once freed, assign the new buffer
        // to destination, then assign the temporary allocator to destination.

      } else {
        // If the two allocators do compare equal, then the source allocator can
        // manage the memory allocated by the destination allocator.  However,
        // since propagate is true the source allocator must still be copied 
        // over to destination.  Once copied, the allocator can manage the
        // memory already allocated in destination.

      }
    } else {
      // If propagate is false, copying of the allocator is not necessary, the
      // destination allocator will continue to manage it's object's memory.
      // Simply copy over the elements.  CLARIFICATION: Since the allocator is
      // not copied, there's no need to check for equality.  Source might not
      // compare equal to destination, but since it's not getting copied over, 
      // it doesn't matter.  Destination is still managing it's own memory, so
      // don't worry about equality comparison in this branch.

    }
```

## Propagate on Container Move Assignment

[Back to Top](#allocator-comparisons--propagation)

**`propagate_on_container_move_assignment:`**

- `std::true_type` or derived from it if the allocator of type `A` needs to be moved when the container that uses it is move-assigned.
- If this member is `std::true_type` or derived from it, then `A` must satisfy *MoveAssignable* and the move operation must not throw exceptions.
- If this member is not provided or derived from `std::false_type` and the allocators of the source and the target containers do not compare equal, move assignment cannot take ownership of the source memory and must move-assign or move-construct the elements individually, resizing its own memory as needed.

```cpp
    if constexpr (std::allocator_traits<Allocator>::
                      propagate_on_container_move_assignment::value) {
      // If propagate is true, we must move the allocator from source to
      // destination.  Simply move the allocator along with everything else from
      // source to destination.

    } else if (destination_allocator == source_allocator) {
      // If propagate is false, source's allocator can't be moved over, but in
      // this block the destination and source allocators compare equal, so they
      // can safely manage each other's memory.  Ignore the allocator, move over
      // everything else.

    } else {
      // Worst case scenario, can't transfer memory at all since the allocator
      // can't be moved and the destination allocator does not compare equal
      // with the source allocator.  You must allocate new memory with the 
      // destination allocator and move the elements into that new memory.  On
      // success, destroy destination's existing objects and free that memory,
      // then swap in the pointer from the newly allocated memory and copy over
      // everything else.
      //
      // Once that's done, you have two options depending on how consistent you
      // want to be:
      //   1. Should the source wind up in the same state as the other cases?
      //      If so, then destroy moved from elements, free the memory, and
      //      reset the invariants.
      //   2. Should source be left as is, with moved from elements still alive
      //      in allocated memory?  If that's the case, you're done.
      //
      // Currently I'm going to say option 2 because I believe there's only two 
      // cases to consider here:
      //   1. The moved from container is about to die anyway, so let the
      //      destructor destroy the elements and free the memory.
      //   2. If the container is going to be reused, that previously allocated
      //      memory can also be reused, saving an allocation.  The user just
      //      has to be aware of the fact that the elements are in a moved-from
      //      state.  This means extra documentation for the move assignment
      //      operator, but I believe this to be the correct choice at present.

  }
```

There's also a nice write up on [Stack Overflow](https://stackoverflow.com/questions/27471053/example-usage-of-propagate-on-container-move-assignment).

## Propagate on Container Swap

[Back to Top](#allocator-comparisons--propagation)

**`propagate_on_container_swap:`**

- `std::true_type` or derived from it if the allocators of type `A` need to be swapped when two containers that use them are swapped.
- If this member is std::true_type or derived from it, type `A` must satisfy Swappable and the swap operation must not throw exceptions.
- If this member is not provided or derived from `std::false_type` and the allocators of the two containers do not compare equal, the behavior of container swap is undefined.

```cpp
// Example usage.
  constexpr auto swap(Vector &other) noexcept -> void {
    if constexpr (std::allocator_traits<
                      Allocator>::propagate_on_container_swap::value) {
      using std::swap;
      swap(allocator_, other.allocator_);
    } else if constexpr (!std::allocator_traits<
                             Allocator>::is_always_equal::value) {
      contract_assert(
          allocator_ != other.allocator_ &&
          "If propagate_on_container_swap is not provided or is derived from "
          "std::false_type and the allocators of the two containers do not "
          "compare equal, the behavior of container swap is undefined.");
    }

    std::swap(data_, other.data_);
    std::swap(size_, other.size_);
    std::swap(capacity_, other.capacity_);
  }
```
