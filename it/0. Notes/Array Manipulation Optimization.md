how can I make this array manipulation run faster public static long arrayManipulation(int n, List<List<Integer>> queries) { Long[] array = new Long[n]; for (int i = 0; i < n; i++) { array[i] = 0L; } Long max = 0L; for (int i = 0; i < queries.size(); i++) { int start = queries.get(i).get(0) - 1; int end = queries.get(i).get(1) - 1; int operation = queries.get(i).get(2); for (int j = start; j <= end; j++) { array[j] = array[j] + operation; } } for (int i = 0; i < n; i++) { max = Math.max(max, array[i]); } return max; } public static void main(String[] args) { int n = 10; List<List<Integer>> list = List.of( List.of(1, 5, 3), List.of(4, 8, 7), List.of(6, 9, 1) ); }

###### ChatGPT said:

ChatGPT

The current approach has a high time complexity, especially in the inner loop that iterates over the range from `start` to `end` for each query. To make this more efficient, you can use a technique known as a _difference array_, which allows us to apply the range updates in O(1)O(1)O(1) per query.

Here’s how it works:

1. For each query (start,end,operation)(start, end, operation)(start,end,operation), instead of updating each element in the range [start,end][start, end][start,end], you only update two positions in the array:
    - Add `operation` to `array[start]`.
    - Subtract `operation` from `array[end + 1]` (if `end + 1` is within bounds).
2. After all queries, accumulate the values by iterating through the array and finding the maximum.

This approach brings the time complexity down to O(n+q)O(n + q)O(n+q), where qqq is the number of queries, as we only need one loop through the array to apply all range updates.

Here’s the optimized code:

java

Copy code

`public static long arrayManipulation(int n, List<List<Integer>> queries) {     // Initialize an array with n+1 elements to handle end boundary without extra checks.     long[] array = new long[n + 1];      // Apply each query in O(1)     for (List<Integer> query : queries) {         int start = query.get(0) - 1;         int end = query.get(1);         int operation = query.get(2);          array[start] += operation;         if (end < n) {             array[end] -= operation;         }     }      // Calculate the prefix sum and find the maximum value     long max = 0;     long current = 0;     for (int i = 0; i < n; i++) {         current += array[i];         max = Math.max(max, current);     }      return max; }  public static void main(String[] args) {     int n = 10;     List<List<Integer>> list = List.of(             List.of(1, 5, 3),             List.of(4, 8, 7),             List.of(6, 9, 1)     );          System.out.println(arrayManipulation(n, list)); // Output should be the max value after all operations }`

### Explanation

1. **Difference Array Update**:
    - Instead of incrementing each index within the range [start,end][start, end][start,end], we increment only at `start` and decrement at `end + 1`.
2. **Prefix Sum**:
    - By accumulating values in the array from left to right, each position effectively reflects the cumulative effect of all previous operations in the specified ranges.
3. **Finding Max**:
    - While accumulating, we keep track of the maximum value reached.

This significantly reduces the time complexity by eliminating the need to iterate through each range individually.