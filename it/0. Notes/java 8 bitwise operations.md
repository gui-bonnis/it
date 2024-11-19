Please can you explain all kinds of java 8 bitwise operations

###### ChatGPT said:

ChatGPT

In Java 8, as in previous versions of Java, you can perform bitwise operations on integer types (byte, short, int, long) using the following bitwise operators:

1. AND (&):
    
    - Symbol: `&`
    - Description: Performs a bitwise AND operation between two integers. Each bit in the result is set to 1 if and only if both corresponding bits in the operands are 1.
    - Example:
        
        java
        
        Copy code
        
        `int a = 5;       // 0101 in binary int b = 3;       // 0011 in binary int result = a & b;  // 0001 in binary (1 in decimal)`
        
2. OR (|):
    
    - Symbol: `|`
    - Description: Performs a bitwise OR operation between two integers. Each bit in the result is set to 1 if either of the corresponding bits in the operands is 1.
    - Example:
        
        java
        
        Copy code
        
        `int a = 5;       // 0101 in binary int b = 3;       // 0011 in binary int result = a | b;  // 0111 in binary (7 in decimal)`
        
3. XOR (^):
    
    - Symbol: `^`
    - Description: Performs a bitwise XOR (exclusive OR) operation between two integers. Each bit in the result is set to 1 if the corresponding bits in the operands are different.
    - Example:
        
        java
        
        Copy code
        
        `int a = 5;       // 0101 in binary int b = 3;       // 0011 in binary int result = a ^ b;  // 0110 in binary (6 in decimal)`
        
4. NOT (~):
    
    - Symbol: `~`
    - Description: Performs a bitwise NOT operation on a single integer, inverting all its bits (0s become 1s, and 1s become 0s). Note that this is a unary operator.
    - Example:
        
        java
        
        Copy code
        
        `int a = 5;       // 0101 in binary int result = ~a;   // 1010 in binary (-6 in decimal, because Java uses two's complement representation)`
        
5. Left Shift (<<):
    
    - Symbol: `<<`
    - Description: Shifts the bits of an integer to the left by a specified number of positions. Fills in zeros on the right.
    - Example:
        
        java
        
        Copy code
        
        `int a = 5;       // 0101 in binary int result = a << 2;  // 010100 in binary (20 in decimal)`
        
6. Right Shift (>>):
    
    - Symbol: `>>`
    - Description: Shifts the bits of an integer to the right by a specified number of positions. Fills in the leftmost bits with the original sign bit (0 for positive, 1 for negative).
    - Example:
        
        java
        
        Copy code
        
        `int a = -5;      // 11111111111111111111111111111011 in binary int result = a >> 2;  // 11111111111111111111111111111110 in binary (-2 in decimal)`
        
7. Unsigned Right Shift (>>>):
    
    - Symbol: `>>>`
    - Description: Similar to the right shift operator (>>), but it fills in the leftmost bits with zeros, regardless of the sign of the number.
    - Example:
        
        java
        
        Copy code
        
        `int a = -5;      // 11111111111111111111111111111011 in binary int result = a >>> 2;  // 00111111111111111111111111111110 in binary (1073741822 in decimal)`
        

These bitwise operations can be useful for tasks such as low-level bit manipulation, setting or clearing specific bits in a value, and more. They are often used in scenarios where you need to work with binary data or perform efficient bitwise operations on integer values.