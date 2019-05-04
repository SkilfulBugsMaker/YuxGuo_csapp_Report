# bomb_lab

-   PB17111568
-   郭雨轩

---

## 准备工作

-   在我的电脑上面，因为没有安装虚拟机，于是使用wsl子系统完成实验。实验环境Ubuntu18.04。

-   首先从网上获取到bomb的实验程序，使用`tar -zvf bomb.tar` 将其解压，随后使用`objdump -d ./bomb > bomb.s`将反汇编得到的汇编代码重定向到`bomb.s` 中，~~虽然我全程是用gdb进行的调试，但是反汇编出来的代码让我对炸弹的整体情况有了一些了解，在查看参数传递时候也有一些帮助~~。同时，通过阅读c语言的代码，对其结构更清晰 ，c代码如下：

    ``` c
    /***************************************************************************
     * Dr. Evil's Insidious Bomb, Version 1.1
     * Copyright 2011, Dr. Evil Incorporated. All rights reserved.
     *
     * LICENSE:
     *
     * Dr. Evil Incorporated (the PERPETRATOR) hereby grants you (the
     * VICTIM) explicit permission to use this bomb (the BOMB).  This is a
     * time limited license, which expires on the death of the VICTIM.
     * The PERPETRATOR takes no responsibility for damage, frustration,
     * insanity, bug-eyes, carpal-tunnel syndrome, loss of sleep, or other
     * harm to the VICTIM.  Unless the PERPETRATOR wants to take credit,
     * that is.  The VICTIM may not distribute this bomb source code to
     * any enemies of the PERPETRATOR.  No VICTIM may debug,
     * reverse-engineer, run "strings" on, decompile, decrypt, or use any
     * other technique to gain knowledge of and defuse the BOMB.  BOMB
     * proof clothing may not be worn when handling this program.  The
     * PERPETRATOR will not apologize for the PERPETRATOR's poor sense of
     * humor.  This license is null and void where the BOMB is prohibited
     * by law.
     ***************************************************************************/
    
    #include <stdio.h>
    #include <stdlib.h>
    #include "support.h"
    #include "phases.h"
    
    /*
     * Note to self: Remember to erase this file so my victims will have no
     * idea what is going on, and so they will all blow up in a
     * spectaculary fiendish explosion. -- Dr. Evil
     */
    
    FILE *infile;
    
    int main(int argc, char *argv[])
    {
        char *input;
    
        /* Note to self: remember to port this bomb to Windows and put a
         * fantastic GUI on it. */
    
        /* When run with no arguments, the bomb reads its input lines
         * from standard input. */
        if (argc == 1) {
            infile = stdin;
        }
    
        /* When run with one argument <file>, the bomb reads from <file>
         * until EOF, and then switches to standard input. Thus, as you
         * defuse each phase, you can add its defusing string to <file> and
         * avoid having to retype it. */
        else if (argc == 2) {
            if (!(infile = fopen(argv[1], "r"))) {
                printf("%s: Error: Couldn't open %s\n", argv[0], argv[1]);
                exit(8);
            }
        }
    
        /* You can't call the bomb with more than 1 command line argument. */
        else {
            printf("Usage: %s [<input_file>]\n", argv[0]);
            exit(8);
        }
    
        /* Do all sorts of secret stuff that makes the bomb harder to defuse. */
        initialize_bomb();
    
        printf("Welcome to my fiendish little bomb. You have 6 phases with\n");
        printf("which to blow yourself up. Have a nice day!\n");
    
        /* Hmm...  Six phases must be more secure than one phase! */
        input = read_line();             /* Get input                   */
        phase_1(input);                  /* Run the phase               */
        phase_defused();                 /* Drat!  They figured it out!
                                          * Let me know how they did it. */
        printf("Phase 1 defused. How about the next one?\n");
    
        /* The second phase is harder.  No one will ever figure out
         * how to defuse this... */
        input = read_line();
        phase_2(input);
        phase_defused();
        printf("That's number 2.  Keep going!\n");
    
        /* I guess this is too easy so far.  Some more complex code will
         * confuse people. */
        input = read_line();
        phase_3(input);
        phase_defused();
        printf("Halfway there!\n");
    
        /* Oh yeah?  Well, how good is your math?  Try on this saucy problem! */
        input = read_line();
        phase_4(input);
        phase_defused();
        printf("So you got that one.  Try this one.\n");
    
        /* Round and 'round in memory we go, where we stop, the bomb blows! */
        input = read_line();
        phase_5(input);
        phase_defused();
        printf("Good work!  On to the next...\n");
    
        /* This phase will never be used, since no one will get past the
         * earlier ones.  But just in case, make this one extra hard. */
        input = read_line();
        phase_6(input);
        phase_defused();
    
        /* Wow, they got it!  But isn't something... missing?  Perhaps
         * something they overlooked?  Mua ha ha ha ha! */
    
        return 0;
    }
    ```

    可以看到，所有的密码是否正确都在对应的phase_{*}中进行检查。

## 开始拆弹

### phase_1

这个炸弹比较简单，在gdb中使用`disassemble phase_1` 反汇编得到代码

``` assembly
# Dump of assembler code for function phase_1:
   0x0000000000400ee0 <+0>:     sub    $0x8,%rsp
   0x0000000000400ee4 <+4>:     mov    $0x402400,%esi
   0x0000000000400ee9 <+9>:     callq  0x401338 <strings_not_equal>
   0x0000000000400eee <+14>:    test   %eax,%eax
   0x0000000000400ef0 <+16>:    je     0x400ef7 <phase_1+23>
   0x0000000000400ef2 <+18>:    callq  0x40143a <explode_bomb>
   0x0000000000400ef7 <+23>:    add    $0x8,%rsp
   0x0000000000400efb <+27>:    retq
```

其中，在进入phase_1之前，已经将用户输入的字符串的起始地址放入rdi中，第二个参数在`phase_1` 中被置为`0x402400` ，接下来调用了一个`strings_not_equal` 的函数，可以推断，这是一个字符串比较的函数，将用户输入的字符串和内存中的一个字符串进行比较，若相等则通过。在gdb中使用`x /s 0x402400` 得到phase_1为`Border relations with Canada have never been better.` 

---

### phase_2

这个炸弹也比较简单，执行`disassemble phase_2` 得到汇编代码如下：

``` assembly
# Dump of assembler code for function phase_2:
   0x0000000000400efc <+0>:     push   %rbp
   0x0000000000400efd <+1>:     push   %rbx
   0x0000000000400efe <+2>:     sub    $0x28,%rsp
   0x0000000000400f02 <+6>:     mov    %rsp,%rsi
   0x0000000000400f05 <+9>:     callq  0x40145c <read_six_numbers>
   0x0000000000400f0a <+14>:    cmpl   $0x1,(%rsp)
   0x0000000000400f0e <+18>:    je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:    callq  0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:    jmp    0x400f30 <phase_2+52>
   0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax
   0x0000000000400f1a <+30>:    add    %eax,%eax
   0x0000000000400f1c <+32>:    cmp    %eax,(%rbx)
   0x0000000000400f1e <+34>:    je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:    callq  0x40143a <explode_bomb>
   0x0000000000400f25 <+41>:    add    $0x4,%rbx
   0x0000000000400f29 <+45>:    cmp    %rbp,%rbx
   0x0000000000400f2c <+48>:    jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:    jmp    0x400f3c <phase_2+64>
   0x0000000000400f30 <+52>:    lea    0x4(%rsp),%rbx
   0x0000000000400f35 <+57>:    lea    0x18(%rsp),%rbp
   0x0000000000400f3a <+62>:    jmp    0x400f17 <phase_2+27>
   0x0000000000400f3c <+64>:    add    $0x28,%rsp
   0x0000000000400f40 <+68>:    pop    %rbx
   0x0000000000400f41 <+69>:    pop    %rbp
   0x0000000000400f42 <+70>:    retq
```

进入`phase_2` 可以看到，调用了一个`<read_six_numbers>` 的函数，字面上理解是读如六个数字。第一个数字在`%rsp`指向的位置，先检查它是否为1，

>0x0000000000400f0e <+18>:    je     0x400f30 <phase_2+52>

若为1则跳转过炸弹爆炸的函数，否则引爆炸弹。之后进入一个循环，循环的大致意思如下：

``` assembly
   #比较内存中地址为rsp的位置的值是否为1
   0x0000000000400f0a <+14>:    cmpl   $0x1,(%rsp) 
   #是1就跳转，不是1就顺序执行，调用爆炸函数
   0x0000000000400f0e <+18>:    je     0x400f30 <phase_2+52>
   0x0000000000400f10 <+20>:    callq  0x40143a <explode_bomb>
   0x0000000000400f15 <+25>:    jmp    0x400f30 <phase_2+52>
   # %eax <- *(%rbx-4) (在第一次循环中，这个值为1)
   0x0000000000400f17 <+27>:    mov    -0x4(%rbx),%eax
   # %eax <- 2*eax
   0x0000000000400f1a <+30>:    add    %eax,%eax
   # 若 (%rbx == %eax)，就跳过炸弹爆炸的函数，否则没事。
   0x0000000000400f1c <+32>:    cmp    %eax,(%rbx)
   0x0000000000400f1e <+34>:    je     0x400f25 <phase_2+41>
   0x0000000000400f20 <+36>:    callq  0x40143a <explode_bomb>
   # 否则就递增 %rbx，向栈底移动，再检查是否是最后一个数字（与%rbp比较）
   0x0000000000400f25 <+41>:    add    $0x4,%rbx
   0x0000000000400f29 <+45>:    cmp    %rbp,%rbx
   0x0000000000400f2c <+48>:    jne    0x400f17 <phase_2+27>
   0x0000000000400f2e <+50>:    jmp    0x400f3c <phase_2+64>
   # %rbx <- %rsp + 4 
   0x0000000000400f30 <+52>:    lea    0x4(%rsp),%rbx
   # %rbp <- %rsp +24 (联想到有6个数字，一个数字是4个字节，所以这个地址是第六个数字的地址)
   0x0000000000400f35 <+57>:    lea    0x18(%rsp),%rbp
   # 无条件跳转
   0x0000000000400f3a <+62>:    jmp    0x400f17 <phase_2+27>
```

从上面分析不难得出，输入6个数字，首项为1，公比为2。所以密码就是这个等比数列的前六项，也就是`1 2 4 8 16 32` （注意空格，盲猜是用`sscanf`读入的）。

---

### phase_3

用GDB反汇编得到的代码如下：

``` assembly
Dump of assembler code for function phase_3:
   0x0000000000400f43 <+0>:     sub    $0x18,%rsp
   0x0000000000400f47 <+4>:     lea    0xc(%rsp),%rcx
   0x0000000000400f4c <+9>:     lea    0x8(%rsp),%rdx
   0x0000000000400f51 <+14>:    mov    $0x4025cf,%esi
   0x0000000000400f56 <+19>:    mov    $0x0,%eax
   0x0000000000400f5b <+24>:    callq  0x400bf0 <__isoc99_sscanf@plt>
   0x0000000000400f60 <+29>:    cmp    $0x1,%eax
   0x0000000000400f63 <+32>:    jg     0x400f6a <phase_3+39>
   0x0000000000400f65 <+34>:    callq  0x40143a <explode_bomb>
   0x0000000000400f6a <+39>:    cmpl   $0x7,0x8(%rsp)
   0x0000000000400f6f <+44>:    ja     0x400fad <phase_3+106>
   0x0000000000400f71 <+46>:    mov    0x8(%rsp),%eax
   0x0000000000400f75 <+50>:    jmpq   *0x402470(,%rax,8)
   0x0000000000400f7c <+57>:    mov    $0xcf,%eax
   0x0000000000400f81 <+62>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f83 <+64>:    mov    $0x2c3,%eax
   0x0000000000400f88 <+69>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f8a <+71>:    mov    $0x100,%eax
   0x0000000000400f8f <+76>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f91 <+78>:    mov    $0x185,%eax
   0x0000000000400f96 <+83>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f98 <+85>:    mov    $0xce,%eax
   0x0000000000400f9d <+90>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400f9f <+92>:    mov    $0x2aa,%eax
   0x0000000000400fa4 <+97>:    jmp    0x400fbe <phase_3+123>
   0x0000000000400fa6 <+99>:    mov    $0x147,%eax
   0x0000000000400fab <+104>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fad <+106>:   callq  0x40143a <explode_bomb>
   0x0000000000400fb2 <+111>:   mov    $0x0,%eax
   0x0000000000400fb7 <+116>:   jmp    0x400fbe <phase_3+123>
   0x0000000000400fb9 <+118>:   mov    $0x137,%eax
   0x0000000000400fbe <+123>:   cmp    0xc(%rsp),%eax
   0x0000000000400fc2 <+127>:   je     0x400fc9 <phase_3+134>
   0x0000000000400fc4 <+129>:   callq  0x40143a <explode_bomb>
   0x0000000000400fc9 <+134>:   add    $0x18,%rsp
   0x0000000000400fcd <+138>:   retq
```

