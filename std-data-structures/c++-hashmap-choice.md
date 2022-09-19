# C++ HashMap Choice

## Tip of the Week #183: A Treasure of Maps

__

Maps are very common data structures, but how should we choose which of the many available map types to use in a given situation? This tip aims to give a clear answer for most cases.

`absl::flat_hash_map` is a good choice of map for most situations, and should be considered the default option. There are situations where another choice is needed or preferred; using a choice other than `absl::flat_hash_map` generally warrants a comment to tell maintainers why the choice was made. Factors that might lead to a different choice include needs for pointer stability, sorted ordering, or a stable iteration order, as well as whether the map is written just once or updated over time.

Let's define our terms and give a decision tree.

### Definitions

_Pointer stability_: a map is said to have pointer stability (equivalently, "reference stability") if a pointer (or reference) to a key or value in a map is guaranteed to remain valid for the lifetime of the map. \[^1]

_Stable iteration order_: a map is said to have stable iteration order if the order of its elements is consistent across different instances of a container and different runs of the same binary. \[^2]

### Order of Preferences

1. Prefer a hashed container unless sorted order is needed. go/hashmaps is authoritative in this case. In brief:
   1. Use `absl::flat_hash_map` unless there is a reason why it doesn't work well in your situation.
   2. Is pointer-stability of values (but not keys) needed?
      * Use `absl::flat_hash_map<Key, std::unique_ptr<Value>>`.
   3. Is pointer-stability of keys needed?
      * Use `absl::node_hash_map`, which provides pointer-stability for both keys and values.
   4. Is stable iteration order required?
      * Use `gtl::linked_hash_map`.
   5. Are all of the values compile-time constants, and can the container's type be deduced instead of being spelled out?
      * Consider using `gtl::fixed_flat_map_of`
2. If sorted order is required, consider an ordered container:
   1. Use `absl::btree_map` unless there is a reason why it doesn't work well in your situation.
   2. Is the map written once and then read many times? A "flat" map (backed by a contiguous array) can provide better performance and use less memory.
      1. Are runtime values required?
         * Use `gtl::flat_map`.
      2. Does the code need to spell out the type?
         * Use `gtl::flat_map`.
      3. Would compile-time (`constexpr`) initialization be too costly?
         * Use `gtl::flat_map`.
      4. Are all of the values compile-time constants, and can the container's type be deduced instead of being spelled out?
         * Consider using `gtl::fixed_flat_map_of`
   3. Is the map written more than once?
      1. Is pointer-stability of values (but not keys) needed?
         * Use `absl::btree_map<Key, std::unique_ptr<Value>>`.
      2. Is pointer-stability of keys needed?
         * Use `std::map`, which provides pointer-stability for both keys and values.

The astute reader (hello, astute reader) may have noticed that `std::unordered_map` is not mentioned above, and neither is the pre-standard `hash_map`. New code should never use `hash_map`, and should use `std::unordered_map` only if it is unable to use Abseil's more optimized alternatives.

NOTE: These recommendations also apply, _mutatis mutandis_, for sets.

### Parting Thoughts

> I usually make this point: even when the choice is not about runtime efficiency, it's documenting your intention for the future maintainer. "I need these unique and sorted" and "I need these unique" are two different statements -- please take a moment, and make the right one. - aemelianov@

Historically, ordered containers were sometimes used to make life easier when writing code. Now that `absl::Hash` is available, providing high-quality hashing for your types is straightforward, allowing us to improve performance and maintainability by preferring hashed containers. Just as `std::vector` should be the default option for sequences, so `absl::flat_hash_map` (and `absl::flat_hash_set`) should now be used for most associative containers.

While it aims to cover most situations, the decision tree given in this tip does not describe every situation. If measurements show that a different container would work better in your application then go ahead and use it, leaving a comment to document the choice for future readers.

