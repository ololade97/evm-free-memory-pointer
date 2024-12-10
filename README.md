# Understand EVM Free Memory Pointer (0x40)

The purpose of this resource is to simplify EVM free memory poionter (0x40) so that anyone can understand.

At the end of this lesson, you will know:
- What is free memory pointer (0x40)
- How to use free memory pointer
- The two use cases of mstore
- Memory position progression
- Important rules

Let's get to it.

To explain the principles as we go, I would write some simple assembly/Yul code. Don't worry if you don't understand Yul. You can pick it up from here if you're already comfortable with Solidity.

In EVM, values are stored in two places - storage or memory. Here, we are concentrating on "memory". Memory is used to store values temporarily. Here's how memory looks like:

Memory layout:

```
0x00-0x3F: Scratch space
0x40-0x5F: Free memory pointer
0x60-0x7F: Zero slot
0x80+: Free memory starts here and beyond
```

The above are the four different places values can be stored or stay in memory. Note that 0x00 and 0x60 (the first and third) are usually not used to store values in memory.

We aren't going into 0x00 (Scratch space) and 0x60 (Zero slot) here. Our main focus is on 0x40 (free memory pointer) and 0x80 (the start of free memory). Generally, these are the two positions that helps you determine where to store a value temporarily. Examples would be provided to solidify your understanding.

# Free Memory Pointer (0x40)
0x40 is 64 in decimal right? Run "chisel" then "0x40" in the terminal in foundry environment if you doubt.

That means 0x40 contains 2 bytes (32 + 32 = 64).

This is what you know about 0x40, right? For you to understand what a free memory pointer is, you need to put this aside for a while. Here's what 0x40, the free memory pointer, means.

Think of 0x40 as a bookmark. A bookmark is where you store different important things in.  Your browser bookmark that contains different important websites comes to mind?  

Now, when you need to access a website you stored in the bookmark, you first click on "bookmark" and then click on the website to access it. 

0x40 does a similar thing like a bookmark. That's why it is called a pointer - free memory pointer. 0x40 is a bookmark that has different values stored temporarily in it. When you need to access any value, you must call 0x40 (the bookmark) plus the position of the value you want to access. Here's a simple example:

```
contract MultipleValues {

    function storeMultipleValues() public pure returns (uint256 value2) {
        assembly {
            // Store multiple values
            let ptr := mload(0x40)        // ptr = 0x80 (128 in decimal)
             mstore(ptr, 100)              // First value at 0x80
             mstore(add(ptr, 0x20), 200)   // Second value at 0xA0
             mstore(add(ptr, 0x40), 300)   // Third value at 0xC0
             mstore(0x40, add(ptr, 0x60))  // Update pointer

             // Then read value and return this
             value2 := mload(add(ptr, 0x20))  // Gets 200
            }
         }
    }
```
mload is used to load or get a value from a particular position in Memory. mstore is used to store a value to the Memory. Note the "assembly { }" declaration (meaning this is an assembly code) and the use of ":=" to assign a variable. "add" means add ptr and 0x20 together. "let" is used to declare a variable.

Run the code. You would see it returns 200. To access other stored values, at the last line, change 0x20 to 0x40.

Back to 0x40. The above example stores different values to the free memory through the free memory pointer. And it updates the free memory pointer and then returns the value stored at ptr (0x40) by adding 0x20 to it. This returns 200. To access other stored values, at the last line, change 0x20 to 0x40. To access the first value stored, that is 100, change the last line to:

```
 value2 := mload(ptr)  // Gets 100
```
You might want to ask why we started by loading and storing 0x40 to the variable "ptr" and the comment in front of it says "ptr = 0x80". 

Remember the Memory Layout? 0x40 is second in the layout and the free Memory pointer (the bookmark). But where did "0x80" come from?

0x80 is the default value stored in 0x40. That is, 0x40 stores 0x80. This automatically points to 0x80 (free memory) - the fourth in the Memory layout where values are written. You might want to ask, "But mstore only store values, and not that it changes and point to the position a value can be stored at?". We will talk about this down the line.

This leads us to the first rule.

Important rule one:
Don't directly store value in 0x40. When you do, 0x40 that should serve as a pointer would be corrupted. 

You might have noticed that in the example code, 0x40 is the basis of all other values stored in the memory. And that is why we didn't store a value in it directly in the example code. Rather, we stored 0x40 and its default value (0x80) in a variable:

```
let ptr := mload(0x40) 
```
Because if we store a value directly in 0x40 like so:

```
mstore(0x40, 100)
```
0x40 won't point to 0x80 anymore. Like I said earlier, 0x80 is the starting place where you can store a value. And that's why 0x40 is pointing to the free memory 0x80.


















