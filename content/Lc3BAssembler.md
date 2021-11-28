Title: Building a stand-alone Assembler for the LC3b ISA 
Date: 2020-1-6 19:11
Modified: 2020-1-6 19:11
Category: Assembler
Tags: LC3b, C
Author: AudiTT
Summary: This post details how to build an assembler for the LC3b ISA. First of many assemblers to come, so stay tuned

This post details how to build an assembler in general but we'll take a look at the Lc3b ISA. 

<h3> What is an Assembler </h3>

A computer program is essentially a sequence of instructions used to instruct the computer to perform a certain task. Since computers are built using digital logic blocks implemented through
CMOS technology, they can only understand 1's and 0's and therefore we need to remove all the layers of abstraction from a program before the computer can understand it. 

<h5> What is Abstraction </h5>

Abstraction of a subject refers to the concept of exposing only the absolute essential traits or aspects of that subject that would allow a user to exploit its usefulness despite being
potentially ignorant of its underlying complexity. For instance, the entire mechanism of driving a car is reduced to being able to operate the steering wheel, the stick shift, the brake pad 
and the acceleration pad etc through abstraction without requiring the need to understand the complexity of the underlying parts such as the engine. It would not be an exaggeration to
claim that the world runs on abstraction. Every product we use, every product a company is hoping to market requires abstraction for it to be useful. 

Similarly, programming languages abstract away all the complexities associated with programming directly in the machine language. However, since the computer can only understand machine language,
 we need to remove this abstraction. The tool that removes the abstraction from an assembly language and thereby translating it to the logic equivalent machine language is called an assembler. 


<img alt="thumbnail" height="350px" src="/assemblerAbstraction.png">


In the figure above, we have a simple assembly program (ASM code) to add -1(Base 10) to the contents of the register R0 and store it in R0. The logic equivalent machine code is shown
on the right. The binary sequence has been written in Base 16 to improve readability. Some of the major functions of an assembler include

1. Correctly translate ASM code to its logic equivalent machine code
2. Detect errors in the ASM code if any and output the appropriate error code 


<h3> Structure of LC3b Assembly Language and Program </h3>

Every program written in LC3b assembly language has the following template

```
	.ORIG Operand ; Operand is an even address
		.
		.
		.
	Instruction/Directive Operands
		.
		.
		.
	.END
```
The .ORIG directive indicates the start of the program and the .END directive indicates the end. The .ORIG directive takes an even addres as an operand and indicates the address where the first
2 bytes of the program is stored. As stated before, the assembler's job is to translate the ASM code into machine code whilst also detecting any errors. Seems like an FSM based implementation
would be ideal for this task since by definition of the language specifications, we know all the possible errors an LC3b assembly program can generate which can be used to trigger transitions
into an error state in the FSM statespace. For instance, two .ORIG directives is illegal, odd operand to .ORIG is illegal, operand to .ORIG has to be a valid address whereas two .END is allowed
and despite .END not taking an operand, it is not illegal since anything following .END is strictly speaking not part of the program. Therefore, just considering the opening and closing
directives we can setup the following FSM to vet a program.


<img alt="thumbnail" height="350px" src="/fsmHigh.png">

![Photo]({attach}fsmHigh.png)

![Photo]({attach}assemblerAbstraction.png)


<a href="https://github.com/1sand0s/Lc3B-Assembler">Github Link to the LC3b Assembler</a>









