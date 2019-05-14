# data lab-PB17111568-郭雨轩

## 实验简介

data lab算是csapp中最水的实验之一了，实验主要涉及到使用各种位运算的奇怪操作来实现特定的功能。

## 实验过程

### 0. 实验环境搭建

-   由于我是在wsl里面进行的实验，由于wsl不能运行32位linux程序，所以需要魔改以下Makefile，去掉`-m32`编译选项即可。

### 1. bitXor

>   */** 要求
>
>    ** bitXor - x^y using only ~ and &* 
>
>    **   Example: bitXor(4, 5) = 1*
>
>    **   Legal ops: ~ &*
>
>    **   Max ops: 14*
>
>    **   Rating: 1*
>
>    **/*

-   基本思路是，首先列出真值表，找到最小项表达式，用`&`和`~`实现`|`，因为有`~((~a)&(~b)) == a|b`，所以按照最小项表达式组合一下即可，代码如下：

    ``` c
    int bitXor(int x, int y) {
    	return ~((~(~x&y))&(~(x&~y)));
    }
    ```

### 2. tmin

>   */** 要求
>
>    ** tmin - return minimum two's complement integer* 
>
>    **   Legal ops: ! ~ & ^ | + << >>*
>
>    **   Max ops: 4*
>
>    **   Rating: 1*
>
>    **/*

-   反汇补码数中最小的一个？~~这是什么沙雕送分题…~~，结果显然是`0x80000000`

    ``` c
    int tmin(void) {
    	return (1<<31);
    }
    ```

### 3. isTmax

>   */** 要求
>
>    ** isTmax - returns 1 if x is the maximum, two's complement number,*
>
>    **     and 0 otherwise* 
>
>    **   Legal ops: ! ~ & ^ | +*
>
>    **   Max ops: 10*
>
>    **   Rating: 1*
>
>    **/*

-   最大的正补码数是`0x7fffffff`，若x为这个数字，则`~(2x+1)==0` ，那么只要返回 `!(~(2x+1))`即可，运行测试代码后发现，测试数据在`0xffffffff`不能通过。所以还需要过滤掉这种情况，那么如何过滤呢，只需要检查x是否为-1，使用`!`运算符可以将整数转换为bool类型，而bool类型其实就是一个bit的整形，所以这句的用处`!(!(x+1))`是实现检查是否为-1，最后代码如下：

    ``` c
    int isTmax(int x) {
        return (!(~(x+x+1)) & !(!(x+1)));
    }
    ```

### 4. allOddBits

>   */** 要求
>
>    ** allOddBits - return 1 if all odd-numbered bits in word set to 1*
>
>    **   where bits are numbered from 0 (least significant) to 31 (most significant)*
>
>    **   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1*
>
>    **   Legal ops: ! ~ & ^ | + << >>*
>
>    **   Max ops: 12*
>
>    **   Rating: 2*
>
>    **/*

-   要求判断所有的奇数位是否都为1，这里的奇数位指的是2的奇数次方位，其实就是数字中的偶数位，方法跟简单，由于限制使用0-0xff的数字，所以使用`0xaa`作为基础的掩码构造一个32位的掩码即可，代码如下：

    ``` c
    int allOddBits(int x) {
        int mask = 0xAA;
        mask = (mask << 8)+mask;
        mask = (mask << 16)+mask;
        return !((x&mask) + ~mask + 1);
    }
    ```

### 5. negate

>   */** 要求
>
>    ** negate - return -x* 
>
>    **   Example: negate(1) = -1.*
>
>    **   Legal ops: ! ~ & ^ | + << >>*
>
>    **   Max ops: 5*
>
>    **   Rating: 2*
>
>    **/*

-   这个题目太水了…常识题目

    ``` c
    int negate(int x) {
        return ~x+1;
    }
    ```

### 6. isAsciiDigit

>   */** 要求
>
>    ** isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')*
>
>    **   Example: isAsciiDigit(0x35) = 1.*
>
>    **            isAsciiDigit(0x3a) = 0.*
>
>    **            isAsciiDigit(0x05) = 0.*
>
>    **   Legal ops: ! ~ & ^ | + << >>*
>
>    **   Max ops: 15*
>
>    **   Rating: 3*
>
>    **/*

-   分别用上界减去x和用x减去下界，判断两个结果是否都是正数即可。

    ``` c
    int isAsciiDigit(int x) {
      	int DownBound = x + ~0x30 + 1;
      	int UpBound = 0x39 + ~x + 1;
      	return (!(DownBound>>31)) & (!(UpBound>>31));
    }
    ```

    