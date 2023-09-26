+++
title = 'EAX ME - MEDIUM/HARD'
date = 2023-09-26T19:49:03-04:00
tags = ["cracking", "pintools", "medium", "hard", "packed"]
description = "Medium/Hard Cracking Challenge, a UPX Packed binary with additional obfuscation rendering unpacking impossible, we have to think outside of the box to trace what is happening and find the key."
draft = false
+++
Windows x86 Console application
Heavily obfuscated, presumably with author's own tool:
https://github.com/DosX-dev/ASM-Guard

Prompt:
```
EAX-Crackme! Coded by DosX
Enter the key: ABCDEFGH

 No, you are a loser :(

```

That's not very nice.
I guess we have to crack it now.

# First glance

Opening in CFF Explorer, here's the entire recoverable import table, note that these imports are common with packers:
```
Address	                Ordinal	Name	Library
000000000040B03C		LoadLibraryA	KERNEL32
000000000040B040		ExitProcess	    KERNEL32
000000000040B044		GetProcAddress	KERNEL32
000000000040B048		VirtualProtect	KERNEL32
000000000040B050		exit            msvcrt
```

And here's what we find searching for strings:
```
DOSX:0040612C aByDosxEnterThe db ' by DosX',0Ah
DOSX:0040612C                 db 'Enter the key: ',0
DOSX:00406145                 db 25h, 36h, 5Bh
DOSX:00406148                 dd 0DFEC0A5Eh, 255DFFFEh, 0A00632Ah, 0A732520h, 65655900h
```

With no xrefs obviously.

Picking it apart manually is clearly out of the question.

# Finding the relevant code

Let's start simple with API call logging with api_log so we can see what it's actually importing and using, and to figure out how the input/output is being performed.

We see some interesting things happening in the log file when searching for returns to user code:
```
...
Line 2693: C:\Windows\SysWOW64\msvcrt.dll!printf returns to 401015
Line 2765: C:\Windows\SysWOW64\msvcrt.dll!scanf returns to 401027
Line 2813: C:\Windows\SysWOW64\msvcrt.dll!strlen returns to 401033
Line 2814: C:\Windows\SysWOW64\msvcrt.dll!printf returns to 4010da
Line 2842: C:\Windows\SysWOW64\msvcrt.dll!_getch returns to 4010e2
```

If we attach to the process while it's running with x32dbg and go to address 401027, we can find the following verification logic:
```
00401022    call <JMP.&scanf>
00401027    add esp,8
0040102A    lea eax,dword ptr ss:[ebp-1]
0040102D    push eax
0040102E    call <JMP.&strlen>
00401033    add esp,4
00401036    cmp eax,6
00401039    jne eax.4010AD
0040103F    lea eax,dword ptr ss:[ebp-1]
00401042    add eax,3
00401045    movsx ecx,byte ptr ds:[eax]
00401048    cmp ecx,61						; 'a'
0040104B    jne eax.4010AD
00401051    lea eax,dword ptr ss:[ebp-1]
00401054    add eax,1
00401057    movsx ecx,byte ptr ds:[eax]
0040105A    cmp ecx,3C						; '<'
0040105D    jne eax.4010AD
00401063    movsx eax,byte ptr ss:[ebp-1]
00401067    cmp eax,49						; 'I'
0040106A    jne eax.4010AD
00401070    lea eax,dword ptr ss:[ebp-1]
00401073    add eax,4
00401076    movsx ecx,byte ptr ds:[eax]
00401079    cmp ecx,73						; 's'
0040107C    jne eax.4010AD
00401082    lea eax,dword ptr ss:[ebp-1]
00401085    add eax,5
00401088    movsx ecx,byte ptr ds:[eax]
0040108B    cmp ecx,6D						; 'm'
0040108E    jne eax.4010AD
00401094    lea eax,dword ptr ss:[ebp-1]
00401097    add eax,2
0040109A    movsx ecx,byte ptr ds:[eax]
0040109D    cmp ecx,33						; '3'
004010A0    jne eax.4010AD
004010A6    mov eax,1
004010AB    jmp eax.4010B2
004010AD    mov eax,0
004010B2    test eax,eax
004010B4    je eax.4010BF
004010BA    jmp eax.4010C9
004010BF    mov eax,eax.402056        402056:"No, you are a loser :("
004010C4    jmp eax.4010CE
004010C9    mov eax,eax.40203B        40203B:"Yees! You are the best! :)"
```

The characters are verified out of order.

The quickest way to get the correct order is just to enter '123456' as the password and step through to see which number is compared to which value.

Putting them all in order we get 'I<3asm':
```
EAX-Crackme! Coded by DosX
Enter the key: I<3asm

 Yees! You are the best! :)

```
