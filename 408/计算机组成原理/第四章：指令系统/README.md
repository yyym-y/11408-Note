# 第四章：指令系统

## 索引

* [指令的基本格式](./指令的基本格式.md)
* [拓展操作码](./拓展操作码.md)
* [指令寻址](./指令寻址.md)
* [数据寻址](./数据寻址.md)
* [汇编指令快速入门](./汇编指令快速入门.md)
* [跳转指令的机器代码](./跳转指令的机器代码.md)


## 背诵单

1. ISA 的主要内容：
    > 指令类型, 指令寻址方式, 操作类型, 每种操作数的相应规定
    > 操作数类型, 操作数寻址方式, 大端存储/小端存储
    > 程序可访问的寄存器编号, 寄存器个数, 位数, 存储空间大小和编制方式
    > 指令执行的控制方式, 包括程序计数器等等
2. 定长操作码/拓展操作码(说出具体的实现过程, 以及如何拓展,拓展多少)
3. 指令寻址和数据寻址是什么
4. 数据寻址方式: 隐含寻址, 立即寻址, 直接寻址, 间接寻址, 寄存器寻址, 寄存器间接寻址, 基址寻址, 相对寻址, 变址寻址, 堆栈寻址 (说出寄存器的名称)
5. 注意计算偏移的时候, 要注意偏移量是否为补码负数
6. 条件专业判断条件及原因
    > `ZF = 0` 不相等跳转   `ZF = 1` 相等跳转
    > 大于:
    > 1. 有符号数: `ZF = 0 and SF = OF` 如果不相等, 并且符号位和负溢出相等, 那么表示大于
    > 2. 无符号数: `ZF = 0 and CF = 0`  不相等并且符号位为正说明大于
    > 小于:
    > 1. 有符号数: `ZF = 0 and SF != OF` 如果不相等, 并且符号位和负溢出不相等, 那么表示小于
    > 2. 无符号数: `ZF = 0 and CF = 1`  不相等并且符号位为负说明小于
    > 大于等于和小于等于只需要将 `ZF` 的判断去掉就可以了