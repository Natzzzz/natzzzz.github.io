---
layout: post
title: "Core wars - your new favorite game"
date: 2021-09-01 12:00:00
categories: misc
---

Last year while playing [r2con](http://rada.re/con) ctf, i saw some people playing an obscure assembly-like game I didn't understand at all. It actually wasn't assembly, it was Redcode, and the game was called Core War. 

Here's how it works: two programs (called warriors) fight against each other in a small simulated memory space (called the core). 

Each round, the warriors execute one instruction until one of them can't play anymore, for example because its instruction pointer got corrupted by the opponent. 

At the beginning, each warrior is loaded into the core at a random address. There is no absolute address, thus you won't be able to specify a precise one, like 0x...... in the usual asm languages. The core is a sort of array, in which there is no beginning and no end. In fact, if we wanted to represent what the core looked like, it would rather look like a sort of pipe, or a sphere. For your warrior, the address where it currently is will be 0, the next one will be 1 and the previous will be -1, and so on. 

The format is <source>, <destination>, like in x64 asm. Don't forget that, fellow x86 enthusiasts. 

When you create your own warrior, there are 3 main strategies that you might use: 

- **Rock**

    Rock warriors try to take as much addresses in the middle of the core (it is statistically more likely that the opponent will spawn around the middle), hoping that it'll kill the other warrior quickly. 

- **Paper**

    Unlike the rock strategy, this one is trying to survive as much as possible, not to quickly kill its opponent. The most usual way to do this is by writing in as many addresses as it can. Remember the *spl* instruction will be very useful for this one. 

- **Scissors**

    Here's my fav one. It scans the core, trying to find occupied addresses. Once it finds an address where its opponent might be, it will bomb as much as it can around that address, hoping to kill it. Pretty cool, huh ? 

# Opcodes

As in every assembly language, you have a lot of different opcodes, and every one of them serves a different purpose. You can have a look at all of them [here](https://esolangs.org/wiki/Redcode#Instructions). Some of them are also used in assembly, like mov, add, sub, mul, div, mod, jmp, jmz (equivalent to jz in asm), jmn (=jnz), cmp and nop. 

You've certainly already used most of the above opcodes, but if you take a closer look at the full list, you'll see that the other half of these opcodes are completely unknown.

### dat

the *dat* instruction kills the current task. One of the most useful ones out there. 

### mov

The mov operation will copy the instruction from the source (operand A) to the destination (operand b).  

Have a look at a very (very) simple warrior called *The Imp*. It only consists of a single instruction: 

```assembly
;name The Imp
mov 0, 1
```

Yup, that's it, nothing more. 

The explanation will be pretty straightforward: it just copies the instruction of the address where it is (then 0) to the address just next (1). To be more clear: it copies itself into the next address, which creates an infinite loop, as the core doesn't have any beginning, nor end. 

### add, sub and mul

This will add what's on the address at 4 to what's in the address 2. 

```assembly 
add 4, 2 ; adds what's on the address at 4 to what's on the address at 2. 
sub 3, 4 ; Same thing as the add, but it's a substraction. Nothing really hard in there. 
mul 2, 1 ; oh, really ? I'm sure you guessed what this is about. 
```

### div

*div* works the same way as the above, but there's a little trick with it: if the result is a float, it will be rounded down: so if you get 3.9, it will be 3. 

Another thing to know is that you can't divide by 0. If you still do it, your program will stop its execution. Don't divide by 0, then. 

### mod

If you don't already know modulo (pretty sure it isn't the case if you're reading this), it will divide the first argument by the second, and store the remainder in the destination. 

```assembly
mod 7, 5
```

### jmp, jmz, jmn, djn

The jmp instruction is used to move to another address (remember, we only use relative addresses in redcode). The specificity is that you only need to add the <source> address. You still can put some code at the <destination>, but i guess it'll just be ignored. 

```assembly
jmp 4 ; -> jump 4 addresses after the current one
mov 2, -1 ; -> move "dat 9" to -1 -> what was jmp 4 until now
jmp -1 ; jump to the precedent address
dat 9 ; tries to kill the process at 9
```

## spl

I had never seen such an opcode before playing this game. It would be actually totally useless in other situations, because we always have full control over the instruction pointer. However, due to corewars being a turn to turn game, if you want to do two things in the same time, you'll have either to do it manually, (i.e. writing one line per process), OR use the *spl* opcode. 

The *spl* opcode is used to create separate processes, so that your code does two separate things, one at a time. 

<pre>
                                ┌───┐  ┌───► ...
                          ┌────►│spl├──┤
                  ┌───┐   │     └───┘  └───► ...
  ───── single ───┤spl├───┤
        process   └───┘   │     ┌───┐  ┌───► ...
                          └────►│spl├──┤
                                └───┘  └───► ...
</pre>

Obviously, it'll only execute one instruction at a time, but it will alternate between its different processes. 

```assembly
warrior 1 ; -> Process A
warrior 2 ; -> ...
warrior 1 ; -> Process B
warrior 2 ; -> ...
warrior 1 ; -> Process A
warrior 2 ; -> ...
```

### nop

the nop, as most of you already know it, is pretty self-explanatory: NO-Operation. It just does absolutely nothing. wow. 

Great, now you should know everything about redcode (well, actually not. There's a lot more to know, but you already know the basics)

If you'd like to know more about redcode and core wars in general, you definitely have to check the links below. 

Hope you liked that post, don't hesitate to hit me up if you have suggestions, or if i could help you in any way. 

# Ressources / references

[The beginner's guide to redcode ](https://vyznev.net/corewar/guide.html)<br>
[Koth.org's corewars for dummies ](http://www.koth.org/info/corewars_for_dummies/dummies.html)<br>
[Wikipedia - Core wars](https://en.wikipedia.org/wiki/Core_War)<br>
[Corewar.io, to practice online](https://www.corewar.io/)

