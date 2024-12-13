You can use Java Streams to create a `Map` from a stream by leveraging the `Collectors.toMap` method. Here’s an example of how to populate your map with a stream of entries:

```java
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class StreamToMapExample {
    public static void main(String[] args) {
        Map<String, String> errors = Stream.of(
                new String[]{"404", "Not Found"},
                new String[]{"500", "Internal Server Error"},
                new String[]{"403", "Forbidden"}
        ).collect(Collectors.toMap(
                entry -> entry[0],  // Key mapper: first element of the array
                entry -> entry[1]   // Value mapper: second element of the array
        ));

        // Print the map
        errors.forEach((key, value) -> System.out.println(key + ": " + value));
    }
}
```

### Explanation

1. **Input Data Format:** The `Stream.of` takes an array of `String` pairs (`String[]`) representing key-value pairs.
2. **`Collectors.toMap`:** Converts the stream to a map using a key-mapper and a value-mapper.
    - `entry -> entry[0]`: Extracts the first element of the array as the key.
    - `entry -> entry[1]`: Extracts the second element of the array as the value.

### Alternative for Simplicity

If your entries are simpler key-value pairs, you can use `Map.entry`:

```java
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class StreamToMapExample {
    public static void main(String[] args) {
        Map<String, String> errors = Stream.of(
                Map.entry("404", "Not Found"),
                Map.entry("500", "Internal Server Error"),
                Map.entry("403", "Forbidden")
        ).collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue
        ));

        // Print the map
        errors.forEach((key, value) -> System.out.println(key + ": " + value));
    }
}
```

### Notes:

- If there are duplicate keys in the stream, you'll need to provide a merge function in `toMap` to handle them.
- For immutable maps, consider `Map.ofEntries` directly:
    
    ```java
    Map<String, String> errors = Map.ofEntries(
        Map.entry("404", "Not Found"),
        Map.entry("500", "Internal Server Error"),
        Map.entry("403", "Forbidden")
    );
    ```