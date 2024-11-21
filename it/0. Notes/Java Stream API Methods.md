###### ChatGPT said:

ChatGPT

Java's Stream API provides several methods that make it easier to process collections and other data sources in a functional style. Here’s a summary of the main API methods in Java Streams:

### 1. **Creating Streams**

- **`stream()`**: Creates a sequential stream from a collection.
- **`parallelStream()`**: Creates a parallel stream from a collection.
- **`Stream.of(...)`**: Creates a stream from individual elements or an array.
- **`Stream.generate(...)`**: Creates an infinite stream from a Supplier.
- **`Stream.iterate(...)`**: Creates an infinite stream with repeated application of a function.

### 2. **Intermediate Operations**

- **`filter(Predicate<? super T> predicate)`**: Filters elements based on a condition.
- **`map(Function<? super T,? extends R> mapper)`**: Transforms each element by applying a function.
- **`flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)`**: Flattens and maps elements to a new stream.
- **`distinct()`**: Removes duplicate elements based on `equals`.
- **`sorted()`** / **`sorted(Comparator<? super T> comparator)`**: Sorts elements in natural or specified order.
- **`peek(Consumer<? super T> action)`**: Applies an action for each element without altering the stream.
- **`limit(long maxSize)`**: Limits the stream to a maximum number of elements.
- **`skip(long n)`**: Skips the first `n` elements in the stream.

### 3. **Terminal Operations**

- **`forEach(Consumer<? super T> action)`**: Executes an action on each element in the stream.
- **`forEachOrdered(Consumer<? super T> action)`**: Ensures actions are applied in encounter order in parallel streams.
- **`toArray()` / `toArray(IntFunction<A[]> generator)`**: Collects elements into an array.
- **`reduce(...)`**: Combines elements into a single result.
- **`collect(Collector<? super T, A, R> collector)`**: Accumulates elements into a collection or result.
- **`min(Comparator<? super T> comparator)` / `max(...)`**: Returns the minimum or maximum element.
- **`count()`**: Counts the number of elements.
- **`anyMatch(Predicate<? super T> predicate)`**: Checks if any elements match a condition.
- **`allMatch(Predicate<? super T> predicate)`**: Checks if all elements match a condition.
- **`noneMatch(Predicate<? super T> predicate)`**: Checks if no elements match a condition.
- **`findFirst()` / `findAny()`**: Returns the first or any element, especially useful for parallel streams.

### 4. **Specialized Streams**

Java also has specialized streams for handling primitives: `IntStream`, `LongStream`, and `DoubleStream`. They provide similar methods to `Stream`, plus some additional ones specific to primitive types, like `sum()`, `average()`, etc.

### Example

Here’s a quick example that demonstrates several of these methods:

java

Copy code

`List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David", "Eve");  // Stream example List<String> result = names.stream()     .filter(name -> name.length() > 3) // Intermediate operation     .sorted()                          // Intermediate operation     .map(String::toUpperCase)          // Intermediate operation     .collect(Collectors.toList());     // Terminal operation  System.out.println(result); // Output: [ALICE, CHARLIE, DAVID]`

In this example:

- **`filter()`** filters names with length greater than 3.
- **`sorted()`** sorts the filtered names.
- **`map()`** transforms names to uppercase.
- **`collect()`** collects the result into a list.