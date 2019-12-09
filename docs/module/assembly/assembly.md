# 汇编


## 访问条件码指令

| 指令    | 同义名 | 效果                | 设置条件             |
| ------- | ------ | ------------------- | -------------------- |
| sete D  | setz   | D = ZF              | 相等/零              |
| setne D | setnz  | D = ~ZF             | 不等/非零            |
| sets D  |        | D = SF              | 负数                 |
| setns D |        | D = ~SF             | 非负数               |
| setg D  | setnle | D = ~(SF ^OF) & ZF  | 大于（有符号>）      |
| setge D | setnl  | D = ~(SF ^OF)       | 小于等于(有符号>=)   |
| setl D  | setnge | D = SF ^ OF         | 小于(有符号<)        |
| setle D | setng  | D = (SF ^ OF) \| ZF | 小于等于(有符号<=)   |
| seta D  | setnbe | D = ~CF & ~ZF       | 超过(无符号>)        |
| setae D | setnb  | D = ~CF             | 超过或等于(无符号>=) |
| setb D  | setnae | D = CF              | 低于(无符号<)        |
| setbe D | setna  | D = CF \| ZF        | 低于或等于(无符号<=) |


## 条件码寄存器

条件码寄存器描述了最近的算数或逻辑操作的属性。

CF：进位标志，最高位产生了进位，可用于检查无符号数溢出。

OF：溢出标志，二进制补码溢出——正溢出或负溢出。

ZF：零标志，结果为0。

SF：符号标志，操作结果为负。