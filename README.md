![Banner](https://github.com/EliteDaMyth/JS-VM-Compiler/raw/main/banner.png)

# Let's Make a Compiler and Virtual Machine in JavaScript
This is a minimalistic and very simple implementation of a Virtual Machine and Compiler for the Brainf*ck language in JavaScript.
The aim of this project is to teach everyone that not all compilers and VM's have to be complex and huge. After reading this, Hopefully you will have an understanding of how some machines and languages work.

If you learnt anything, or think this helped you in anyway, Don't forget to leave a star! All PR's are also welcome.

- [Let's Make a Compiler and Virtual Machine in JavaScript](#lets-make-a-compiler-and-virtual-machine-in-javascript)
  - [What is a compiler?](#what-is-a-compiler)
    - [Lexical Analysis](#lexical-analysis)
    - [Parsing](#parsing)
      - [Abstract Syntax Tree](#abstract-syntax-tree)
    - [Compiling/Interpreting](#compilinginterpreting)
      - [Bytecode](#bytecode)
  - [What exactly is a Virtual Machine?](#what-exactly-is-a-virtual-machine)
- [How to run the code?](#how-to-run-the-code)
- [Contact me.](#contact-me)

## What is a compiler?
If someone asks you to name a compiler, you will probably think of a compiler like GCC, The Rust compiler, CLANG compiler, etc. We associate the word compiler with a program that takes in code and spits out an executable file.

But compilers come in all shapes and sizes and compile all kinds of things, not just programming languages, including regular expressions, database queries, and even HTML templates. I bet you use one or two compilers every day without even realizing it. That’s because the definition of “compiler” itself is actually quite loose, much more so than one would expect. Here is Wikipedia’s Definition:
> A compiler is computer software that transforms computer code written in one programming language (the source language) into another computer language (the target language). Compilers are a type of translator that support digital devices, primarily computers. The name compiler is primarily used for programs that translate source code from a high-level programming language to a lower level language (e.g., assembly language, object code, or machine code) to create an executable program.

Compilers are translators. That’s vague. And a compiler that translates high-level languages to produce executables is just one special type of compiler. The variation in the size of compilers can be huge. A compiler can be written in a few hundred lines, or a few million like the GNU Compiler Collection (GCC) which has over 15 million lines of code. We are definitely not making something that big, so what exactly are we making? We are going to make a Simple compiler, that takes the AST as the input and outputs bytecode for our VM to execute. Now, what is an AST? To know about AST's we have to learn about how a programming language works.
Every language has a few pieces:
1. Lexical analysis/Tokenizing
2. Parsing
3. Compilation/Evaluation

### Lexical Analysis
The first step sounds all fancy and stuff, but what is basically happening, Is that the code is being turned into "Tokens". For example, in our Brainf*ck Tokenizer, it takes the string of the program as an input and returns an array of tokens. I.e. if you give it the input `.+.<>-`, It will return something similar to
```js
['.', '+', '.', '<', '>', '-']
```
Except each token will actually be an Object, with certain utilities. ([See tokenizer.js](https://github.com/EliteDaMyth/JS-VM-Compiler/blob/main/tokenizer.js))
This is an important step because we can know if any non recognized characters are found in our program, and throw errors early on, before moving forward. It also makes it easier for our Parser to read the program.

### Parsing
According to Wikipedia:
> Parser is a software component that takes input data (frequently text) and builds a data structure – often some kind of parse tree, abstract syntax tree, or other hierarchical structure – giving a structural representation of the input, checking for correct syntax in the process. [...] The parser is often preceded by a separate lexical analyzer, which creates tokens from the sequence of input characters;

In simple words, A parser turns its input into a data structure that represents the input.
If you have worked in javascript before, chances are you most probably have used `JSON.parse()`. It works on basically the same principle. It takes a string as an input, and it parses it to a Javascript Object. The only difference is, in our parser, we will take an object of Tokens, then turn it into an Abstract Syntax Tree.

#### Abstract Syntax Tree

Now, You may wonder what an AST is. In most interpreters and compilers the data structure used for the internal representation of the source code is called a "syntax tree" or an "abstract syntax tree" (AST for short). The "abstract" is based on the fact that certain details visible in the source code are omitted in the AST. Semicolons, newlines, whitespace, comments, braces, bracket, and parentheses -- depending on the language and the parser these details are not represented in the AST, but merely guide the parser when constructing it.

In our case, Our AST has the following structure:
```js
AstNode {
  _valid_names_list: [
    'MoveLeft',
    'MoveRight',
    'Increment',
    'Decrement',
    'Output',
    'Input',
    'While'
  ],
  _name: 'Increment',
  _next: AstNode { // This is the Next Item in the AST
    _valid_names_list: [
      'MoveLeft',
      'MoveRight',
      'Increment',
      'Decrement',
      'Output',
      'Input',
      'While'
    ],
    _name: 'Increment',
    _next: AstNode {
      _valid_names_list: [Array],
      _name: 'Increment',
      _next: [AstNode] // This will keep going on until the end of the program.
    }
  }
}
```

The Object property `_next` is the next block of the AST. This means if there are 100 Tokens from the Lexer, there will be a depth of 100 in the AST. The last Node's `_next` property will be `null`.

### Compiling/Interpreting
This is the third and last part of any programming language. As we have read earlier, A compiler is basically a translator. In our case, We want the compiler to take our AST as an input, and output the bytecode, for the VM to Execute.
#### Bytecode
Bytecode is also known as portable code, is basically a set of instructions for the machine. It is efficient for an interpreter to interpret bytecode. Each instruction of the Bytecode consists of an Opcode and an optional number of operands. An Opcode is exactly 1 Byte wide and is the first byte in the instruction.

Our bytecode instructions are:

```js
{"op": "<>","value": x} // move memory pointer to += x (x can be negative)
{"op": "+-","value": x} // update current byte to += x (x can be negative)
{"op": "PRINT","value": x} // print current byte
{"op": "READ","value": x} // read a value to current byte
{"op":"ifjump", index: x} // set memory pointer to x, if current byte is zero
{"op":"jump", index: x} // set memory pointer to x (unconditional goto)
```
So, for example we have a program `.--<>[.]`, Our bytecode, will then look like 
```js
CompiledProgram {
  _byte_code: [
    { op: 'PRINT' },
    { op: '+-', value: -1 },
    { op: '+-', value: -1 },
    { op: '<>', value: -1 },
    { op: '<>', value: 1 },
    { op: 'ifjump', index: 9 },
    { op: 'PRINT' },
    { op: 'jump', index: 6 }
  ]
}
```

Now we know what Bytecode is, We can learn what a VM is.

## What exactly is a Virtual Machine?
When you read the term Virtual Machine, the first thing that comes to your mind would be something like VMWARE, or VirtualBox. But these are not the kind of VM's we are going to build.

What we are going to build are virtual machines that are used to implement programming languages. Sometimes they consist of just a few functions, other times they make up a few modules and on occasion, they’re a collection of classes and objects. It’s hard to pin their shape down. But that doesn’t matter. What’s important is this: they don’t emulate an existing machine. They are the machine.

In order to understand Virtual Machines, We must understand how real machines work. 
Almost all the machines you encounter in your daily life are based on the [Von Neumann architecture](https://en.wikipedia.org/wiki/Von_Neumann_architecture). 
In Von Neumann’s model, a computer has two central parts: a processing unit, which contains an arithmetic logic unit (ALU) and multiple processor registers, and a control unit with an instruction register and a program counter. Together they’re called the central processing unit, often shortened to CPU. Besides that, the computer also contains memory (RAM), mass storage (think: hard drive), and input/output devices (keyboard and display).
Here is a rough sketch of the Von Neumann architecture: 
![Von Neumann architecture](https://upload.wikimedia.org/wikipedia/commons/e/e5/Von_Neumann_Architecture.svg)

When a computer is turned on, the CPU: 
1. Fetches an instruction from memory. The program counter tells the CPU where in memory it can find the next instruction. 
2. Decodes the instruction. To identify which operation should be executed. 
3. Executes the instruction. This can mean either modifying the contents of its registers, or transferring data from the registers to memory, or moving data around in memory, or generating output, or reading input.

These 3 steps are repeated indefinitely. This is known as the fetch-decode-execute cycle. Or the instruction cycle. This is the thing also known as the "Clock" of the computer.

Now, that we know a bit about how a real computer works, We can understand about Virtual Machines.
According to the definition,
> A virtual machine is a computer built with software. It’s a software entity that mimics how a computer works.

Just like a real computer, our virtual machine also has a loop of the fetch-decode-execute cycle. Our virtual machine also has a Program counter, it also has a Stack, Memory, Pointers, etc. All made in software.

I won't go Into much details with the code here, you can look for yourself in the [vm.js file](https://github.com/EliteDaMyth/JS-VM-Compiler/blob/main/vm.js). But basically, what our virtual machine is doing, Is taking the Bytecode output from the Compiler, looping through each instruction, Changing the memory location according to the bytecode instructions, and printing the string at the current memory location when it reads the `PRINT` bytecode instruction. 


# How to run the code?
The code doesnt need any dependencies except node.js to run. Use the following commands to run the code:
```
> git pull https://github.com/EliteDaMyth/JS-VM-Compiler.git
> node testing.js
```

# Contact me. 

You can always contact me on discord via EliteDaMyth#0690. You can also create an issue on this repository if you found something which is not in place. IF you wanna join my discord server, you can find it here: [https://discord.gg/ZbQBRZ5Jnc](https://discord.gg/ZbQBRZ5Jnc)