---
layout: post
title:  "The Eldritch Birds Nest - Control Flow Graph Clumping"
date:   2021-11-03 21:22:00 +0100
author: Nia Catlin
categories: graph_layout
image: /rgatPages/blogimg/1/thumbnail_jmc_gs.png
---

Modern compilers generate a lot of code that gets poor results from force-directed graph layouts. This post discusses why and some steps being taken to improve it.

Let's start with a solved problem

#### Import Thunks

Here is an rgat testcase called ConsolePrint32

```x86
main:   mov     ebp, esp
        sub     esp, 4	
        push    -11
        call   GetStdHandle 

        mov     ebx, eax    
        push    0
        lea     eax, [ebp-4]
        push    eax
        push    (message1_end - message1)
        push    offset  message1
        push    ebx
        call    Writefile

        push    -11
        call   GetStdHandle

        mov     ebx, eax    
        push    0
        lea     eax, [ebp-4]
        push    eax
        push    (message2_end - message2)
        push    offset  message2
        push    ebx
        call    Writefile

        push    0
        call    ExitProcess 
```

That's a very flat control flow, so the graph should look really simple, right?

Here is what happens if we feed the raw trace into the force directed node layout pipeline (ie: Standard Fruchterman-Reingold) 

![Force-directed node render of consoleprint32 with jumpthunks](/rgatPages/blogimg/1/rgat_ConsolePrint32_thunks.png)

The issue here is that ```call GetStdHandle``` doesn't actually call GetStdHandle in kernel32.dll - it calls a jump thunk within the .text section ```jmp dword ptr [0x812008]``` which pulls the actual address of GetStdHandle from the .idata section and jumps to it. Every call to the same API will be linked to the same jump thunk (the faint curved grey edges in the above plot), which ruins the layout.

If the 'Hide API Thunks' option is enabled at trace launch, rgat will detect and write these thunks out of the graph - making the layout much more useful for understanding what the program is doing.

![Force-directed node render of consoleprint32 without jumpthunks](/rgatPages/blogimg/1/rgat_ConsolePrint32_nothunks.png)

#### Exploit Mitigations - Stack Check

Life gets more difficult now. Here is a simple C++ program used to test rgat's handling of blocking code. It will be compiled with standard Visual Studio defaults.

```cpp
void WaitFunction()
{
    for (int i = 0; i < 15; i++)
    {
        std::cout << "...sleep\n";
        Sleep(1);
    }
}

void InputFunction()
{
    std::cout << "Blocking on user input...\n";
    std::string test;
    std::cin >> test;
    WaitFunction();
}

int main()
{
    std::cout << "Starting wait\n";
    WaitFunction();
    std::cout << "...done\n";
    InputFunction();
    std::cout << "...done\n";
    WaitFunction();
    std::cout << "...done\n";
    return 0;
}
```

And here is how it looks with the same force directed layout applied to it:

![Zoomed out Force-directed node render of blockingapi](/rgatPages/blogimg/1/rgat_BlockingAPI_zoomedout.png)

Oof. 

Bear in mind that most of this isn't the code above, but [CRT Initialization](https://docs.microsoft.com/en-us/cpp/c-runtime-library/crt-initialization?view=msvc-160) code which you will see in some form in many Windows binaries.

Let's see what is causing the horrible star shaped tangle.

![Zoomed out Force-directed node render of blockingapi](/rgatPages/blogimg/1/rgat_BlockingAPI_stack_check.png)

The whole layout has been mangled by these three instructions

```
00C6146F | E9 2C620000           | jmp <blockingapi._RTC_CheckEsp>                             |
---
00C676A0 | 75 01                 | jne <blockingapi.esperror>                                  | stack.cpp:21
00C676A2 | C3                    | ret                                                         | stack.cpp:25
```

 Anyone can slip a jump to hidden code in there though, so ignoring it is a non-starter.


 
#### Exploit Mitigations - Intel MPX

[Gone from new compilers](https://en.wikipedia.org/wiki/Intel_MPX#Software_support), but not from our hearts/many binaries - the effect of this on control flow graphs was similar to that of stack checking in that it joins much of the code up to a small number of ```bnd``` prefixed instructions.

![Intel MPX clumping](/rgatPages/blogimg/1/rgat_BlockingAPI_MPX.png)

#### Just My Code

Seen less in release code, JMC checks litter function prologues with [calls to __CheckForDebuggerJustMyCode](https://reverseengineering.stackexchange.com/questions/27917/the-compiler-adds-a-function-call-to-user-defined-functions-what-does-the-funct)   


### Potential Workarounds

This section discusses work in progress to try and fix this problem

#### Force adjustments

One low-effort potential option is to reduce the strength of edges connecting highly connected nodes. 

<video src='https://user-images.githubusercontent.com/5470374/139589297-83b23c7d-c750-405c-8a9c-6ccdbabb8fef.mp4' controls='controls' style='max-width: 800px;'></video>

This is quite underwhelming and tends to just make a bigger mess. It would probably be more effective to have the first in and out edges treated normally and the rest just ignored, but nothing useful is likely to come from this.


#### Different plots

##### Force Directed

This could just be an issue with Fructerman Reingold. [GraphInsight](https://github.com/CarloNicolini/GraphInsight) offers a few alternative force-directed layouts of the same trace 

![GraphInsight rendering of standard FR](/rgatPages/blogimg/1/GI_blockingapi_FR.png)

![GraphInsight Binary Stress Rendering](/rgatPages/blogimg/1/GI_blockingapi_binstress.png)

![GraphInsight FR Proportional Multilevel](/rgatPages/blogimg/1/GI_blockingapi_FRpropml.png)

Some offer a little better insight into the structure of the code, but it might be that Graph Rewriting is the only option.

##### Static

It could also be true that I'm just throwing magic pretty physics algorithms at instruction traces and hoping some useful insight appears, when in reality Force Directed layouts could just be entirely inappropriate for control flow graphs.

![rgat Cylinder layout](/rgatPages/blogimg/1/rgat_BlockingAPI_JMC_GS_Cylinder.png)

rgat's Cylinder layout of the above trace is not affected in terms of how nodes are positioned, it's just clogged up with all the edges entering/leaving these routines, which is a much easier fix.



#### Graph Rewriting

Another options is to mimic how API thunks are handled, but this makes everything a lot more error prone. Duplicating 
single jumps to uninteresting code is a lot easier to deal with than trying to chop and change the graph of instrumented code being visualised.