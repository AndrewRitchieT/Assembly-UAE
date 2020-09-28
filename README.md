# Assembly
 汇编语言



AT&T与Inter汇编格式区别

GCC采用的是AT&T的汇编格式, 也叫GAS格式(Gnu ASembler GNU汇编器), 而微软采用Intel的汇编格式.

美国电话电报公司（AT&T）


语法上主要有以下几个不同.

## 1、寄存器命名原则

| AT&T | Intel | 说明              |
| ---- | ----- | ----------------- |
| %eax | eax   | Intel的不带百分号 |



## 2、源/目的操作数顺序

| AT&T            | Intel        | 说明                               |
| --------------- | ------------ | ---------------------------------- |
| movl %eax, %ebx | mov ebx, eax | Intel的目的操作数在前,源操作数在后 |



## 3、常数/立即数的格式

| AT&T              | Intel          | 说明                         |
| ----------------- | -------------- | ---------------------------- |
| movl $_value,%ebx | mov eax,_value | Intel的立即数前面不带$符号   |
| movl $0xd00d,%ebx | mov ebx,0xd00d | 规则同样适用于16进制的立即数 |



## 4、操作数长度标识

| AT&T         | Intel     | 说明                                              |
| ------------ | --------- | ------------------------------------------------- |
| movw %ax,%bx | mov bx,ax | Intel的汇编中, 操作数的长度并不通过指令符号来标识 |



在AT&T的格式中, 每个操作都有一个字符后缀, 表明操作数的大小. 例如:mov指令有三种形式:

movb 传送字节

movw 传送字

movl  传送双字

因为在许多机器上, 32位数都称为长字(long word), 这是沿用以16位字为标准的时代的历史习惯造成的.

如果没有指定操作数长度的话，编译器将按照目标操作数的长度来设置。比如指令“mov %ax, %bx”，由于目标操作数bx的长度为word，那么编译器将把此指令等同于“movw %ax, %bx”。同样道理，指令“mov $4, %ebx”等同于指令“movl $4, %ebx”，“push %al”等同于“pushb %al”。对于没有指定操作数长度，但编译器又无法猜测的指令，编译器将会报错，比如指令“push $4”。



## 5、寻址方式

| AT&T                                       | Intel                                           |
| ------------------------------------------ | ----------------------------------------------- |
| imm32(basepointer,indexpointer,indexscale) | [basepointer + indexpointer*indexscale + imm32) |



两种寻址的实际结果都应该是

imm32 + basepointer + indexpointer*indexscale

AT&T的汇编格式中, 跳转指令有点特殊.

直接跳转, 即跳转目标是作为指令的一部分编码的.

​    例如: jmp Label_1

间接跳转, 即跳转目标是从寄存器或存储器位置中读出的. 写法是在" * "后面跟一个操作数指示符.

​    例如: jmp *%eax 用寄存器%eax中的值作为跳转目标

​         jmp *(%eax) 以%eax中的值作为读入的地址, 从存储器中读出跳转目标



## 6、间接寻址方式

与Intel的语法比较，AT&T间接寻址方式可能更晦涩难懂一些。

| AT&T                           | Intel                          |
| ------------------------------ | ------------------------------ |
| %segreg:disp(base,index,scale) | segreg:[base+index*scale+disp] |



其中index/scale/disp/segreg全部是可选的，完全可以简化掉。

如果没有指定scale而指定了index，则scale的缺省值为1。

segreg段寄存器依赖于指令以及应用程序是运行在实模式还是保护模式下，在实模式下，它依赖于指令，而在保护模式下，segreg是多余的。

在AT&T中，当立即数用在scale/disp中时，不应当在其前冠以“$”前缀，表2.3给出其语法及几个相应的例子。



表2.3 内存操作数的语法及举例



| Intel语法                                 | AT&T语法                                   |
| ----------------------------------------- | ------------------------------------------ |
| 指令   foo,segreg:[base+index*scale+disp] | 指令    %segreg:disp(base,index,scale),foo |
| mov  eax,[ebx+20h]                        | Movl   0x20(%ebx),%eax                     |
| add   eax,[ebx+ecx*2h]                    | Addl      (%ebx,%ecx,0x2),%eax             |
| lea   eax,[ebx+ecx]                       | Leal   (%ebx,%ecx),%eax                    |
| sub   eax,[ebx+ecx*4h-20h]                | Subl   -0x20(%ebx,%ecx,0x4),%eax           |

从表中可以看出，AT&T的语法比较晦涩难懂，因为[base+index*scale+disp]一眼就可以看出其含义，而disp(base,index,scale)则不可能做到这点。

这种寻址方式常常用在访问数据结构数组中某个特定元素内的一个字段，其中，base为数组的起始地址，scale为每个数组元素的大小，index为下标。如果数组元素还是一个结构，则disp为具体字段在结构中的位移。





*arm指令如果不加上volatile防止优化，很容易就出现segment fault，因此强制禁止优化也是必须的？*



## 7、内存单元操作数

  从上面的例子可以看出，内存操作数也有所不同。在Intel的语法中，基寄存器用“［］”括起来，而在AT&T中，用“（）”括起来。  



| AT&T                | Intel            |
| ------------------- | ---------------- |
| movl   5(%ebx),%eax | mov  eax,[ebx+5] |







## 6.跳转方式

在 AT&T 汇编格式中，绝对转移和调用指令（jump/call）的操作数前要加上'*'作为前缀，而在 Intel 格式中则不需要。



AT&T:
jmp *%eax 用寄存器%eax中的值作为跳转目标
jmp *(%eax) 以%eax中的值作为读入的地址, 从存储器中读出跳转目标

Intel:不需要*作为前缀
jmp %eax
jmp (%eax)



远程转移指令和远程子调用指令的操作码，

在 AT&T 汇编格式中为 "ljump" 和 "lcall"，而在 Intel 汇编格式中则为 "jmp far" 和 "call far"，即：

AT&T：
ljump $section, $offset
lcall $section, $offset

Intel:
jmp far section:offset
call far section:offset











