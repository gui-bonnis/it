how to sort this list in java List<List<Integer>> list = List.of( List.of(1, 5, 3), List.of(4, 8, 7), List.of(6, 9, 1) ); by column 1 and them 2

###### ChatGPT said:

ChatGPT

To sort the list by column 1 and then column 2, you can use Java's `Comparator` with `List::get`. Here’s how to do it:

java

Copy code

`import java.util.ArrayList; import java.util.Comparator; import java.util.List;  public class SortList {     public static void main(String[] args) {         List<List<Integer>> list = new ArrayList<>(List.of(             List.of(1, 5, 3),             List.of(4, 8, 7),             List.of(6, 9, 1)         ));                  // Sort by column 1, then by column 2         list.sort(Comparator.comparing((List<Integer> row) -> row.get(0))                             .thenComparing(row -> row.get(1)));          System.out.println(list);     } }`

### Explanation:

- `Comparator.comparing(row -> row.get(0))` sorts by the first column.
- `.thenComparing(row -> row.get(1))` adds a secondary sort by the second column if two elements are equal in the first column.

**Output**:

java

Copy code

`[[1, 5, 3], [4, 8, 7], [6, 9, 1]]`

This will sort `list` based on the specified columns. Adjust the column indices (`0` for the first column, `1` for the second) as needed for different orderings.