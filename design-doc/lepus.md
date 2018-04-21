## Animate Virtual Machine
Lynx为实时交互动画提供了一套动画脚本，脚本引擎(lepus)就是交互动画的基础。lepus是一个基于寄存器的虚拟机，其脚本语法使用了JS兼容的语法，是JS语法的子集。由于交互动画计算的特性，为了追求在解释执行的时候更快的速度，lepus尽可能精简了JS的语法。

## 语法支持
### 注释
lepus支持单行注释和多行注释，注释方式和JS语法一致。单行注释使用//开头。多行注释以 /\* 开始，以 \*/ 结尾。

### 变量
lepus支持使用`var`来申请变量，但是lepus <font color=#ff0000>**不支持** `const`, `let`关键字</font>

### 数据类型
lepus支持 **字符串**，**数字**，**布尔**，**Undefined** 类型，但是注意lepus <font color=#ff0000>**不支持** 对象和数组</font>

### 函数
lepus函数定义方式和JS一致，使用关键字`function` 


### 运算符
赋值运算符 | 算数运算符
---------- | --------
 += | +
 -= | -
  *= | *
  /= | /
  %= | %
  = | ++
    | --

### 比较
比较运算符 | 逻辑运算符
---------- | --------
 === | &&
 !== | &#124;&#124;
  \> | !
  < | 
  \>= | 
  <=| 


### If...Else
与JS一致
### Switch
与JS一致
### For
lepus 暂时只支持for语句 <font color=#ff0000>**不支持** for/in</font>
### While
与JS一致支持while和do/while循环语句
### Break
与JS一致支持break/continue

## 数学运算

## 字符串

