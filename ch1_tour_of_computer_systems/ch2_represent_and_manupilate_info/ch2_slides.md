## Binary
- **The base 2 numbering system, also known as the binary system, is a mathematical system that uses only two digits: 0 and 1.** It's used in digital computers to represent data and perform calculations.

- In the binary system, each digit represents a power of 2, with the least significant digit representing 2^0, the next digit representing 2^1, and so on. For example, the binary number 1011 represents the decimal number 11 because:

`1 × 2^3 + 0 × 2^2 + 1 × 2^1 + 1 × 2^0 = 11`

- The binary system is used in computers because it's easy to implement electronically, with each digit being represented by a single transistor or logic gate.
- Binary and decimal
![image](https://github.com/Zumh/csapp2024/assets/17211423/632d3484-156d-4e86-b98f-65a47f1094df)

## Hexadecimal
- Hex or 16 base to decimal
![image](https://github.com/Zumh/csapp2024/assets/17211423/59d6ffde-c02c-40e9-9182-116374edb36a)
- **The XOR (exclusive OR) operation is a logical operation that returns a 1 if the two input bits are different, and a 0 if they are the same**. This operation is often used in computer programming to compare two values and determine if they are equal or not.

For example, the following C code uses the XOR operation to compare the values of two variables, a and b:

```c
int a = 1;
int b = 2;

if (a ^ b) {
  printf("a and b are not equal\n");
} else {
  printf("a and b are equal\n");
}
```

In this example, the XOR operation will return a 1 because the values of a and b are different. Therefore, the printf() statement will print the message "a and b are not equal".

Here is a truth table for the XOR operation:

```
A | B | A XOR B
0 | 0 | 0
0 | 1 | 1
1 | 0 | 1
1 | 1 | 0
```

As you can see from the truth table, the XOR operation returns a 1 only when the two input bits are different. Otherwise, it returns a 0.

## Boolean Algebra
![image](https://github.com/Zumh/csapp2024/assets/17211423/57d2ccac-b1a3-4b0b-9773-a957a03b6385)
