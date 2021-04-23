---
layout: post
title: "Patching a binary using r2"
date: 2020-12-22 12:00:00
categories: reversing ctfs
---

When doing binary exploitation challenges, you can’t always just trick the program to make him do what you want it to do. Sometimes, it’s easier and faster to change it directly in order to get the result we expect.

In this little tutorial, I’ll teach you how to patch an ELF binary with radare2 , an awesome reverse engineering framework that is sooo useful when playing ctfs, doing malware analysis or really anything that involves assembly at some point.

The sample I’ll use for this tutorial is from the PicoCTF 2018.

Always start by running your binary, so that you understand what’s going on.

![](/assets/images/patchingBinaryR2_1.png) 
Sooo, it prints the name of the challenge and the string “calculating key…”, then it waits for a second before to print the last line. Strange, right ? Let’s open r2 now, to see what’s happening inside.\
![](/assets/images/patchingBinaryR2_2.png)
Use the aaa command to analyze your binary. It will detect and analyze the functions, the strings and pretty much everything inside of the program, to make it easier to go through it in our analysis. Then, use the command afl to list all the functions. You’ll see the main function, which is the spine of our program, and from which we’ll see most of the functions that will be called. We have to go to that main function, now that we know it exists. To do so, we’ll seek to that place, using the letter S, as shown below:\
![](/assets/images/patchingBinaryR2_3.png)
I also printed the disas of that main function, to globally understand the flow of the program, and to see what other functions are called. With that disassembly, we can understand what the problem is: the amount of time that is given by the sym.set_timer function is too small, and the program stops its execution before that the sym.print_flag function (our objective) could be called.  
Now that we understood what the problem was, I think about two ways to patch it, in order to get the output of the print_flag function. Of course, you could use gdb and redirect the instruction pointer directly to that function, but we only want to use r2 here, because we love it, right ?  
* The first way would be to seek to the set_timer function and change directly the time that is given to the timer, which is only one second for now, and try to make it bigger.
* The second way would be to just nop out the set_timer function, so that when the instruction pointer comes to the address 0x00400845, It just won’t do anything and continue. That’s what we’re going to do.
Using a veeeeery great functionality of r2, we’ll nop everything that is on the address 0x00400845, as explained before.  
![](/assets/images/patchingBinaryR2_4.png)
Now, we have our target (0x00400845), we just have to go right here, I just printed the disas of that address to be sure it was the right one, and now it’s time to patch, but first, we have to reopen r2 in write mode ! To do so, use the command oo+, and if you don’t have any output, you’re good to go.
![](/assets/images/patchingBinaryR2_5.png)
This magic command is the wao nop I used in the second line. It writes over your opcode with nops, which is exactly what we want to do.  
To be sure everything is ok, re-print the disassembly of our address, aaaaand, it worked. Great ! Now, just quit (command q) and it’s time to see if what we did worked.
![](/assets/images/patchingBinaryR2_6.png)
Whooo ! It still takes a few seconds before the flag is printed, but it works and we get the flag !  
For further documentation, you could read [this](https://r2wiki.readthedocs.io/en/latest/options/w/wao-op/), and more globally [the documentation of r2](https://radare.gitbooks.io/radare2book/content/), it’s a great ressource if you need to get better at reverse engineering. Have fun !











