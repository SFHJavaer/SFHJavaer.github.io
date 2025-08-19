---
layout: post
title: 以太坊开发：Solidity语言入门
catalog: true
typora-root-url: 以太坊开发：Solidity语言入门
author: Futari
date: 2025-04-13 22:19:31
---

<h3 id="CZjfC">两个前提</h3>

+ 说明软件许可
+ 声明源文件版本

<h3 id="WUTBJ">常用基础数据类型</h3>

> 理解上和Java相似

+ int：整型
+ uint(不是unit) = unsigned int 无符号整型，即正整数
+ string：均为小写开头，字符串型
+ bool：布尔，逻辑运算符和Java一致（也有短路规则）
+ enum：枚举，可以和unit（0,1,2,3）按顺序转换，用的很少，用.调用
+ address：EVM地址类型，40位长度 = 20字节 0x开头代表这个是16进制串
+ bytes：定长数组类型，和unit相似有bytes1,bytes8,bytes32等等，一个字节等于一个字母，相当于基础类型的string

<h3 id="PHVYl">引用数据类型</h3>

+ 数组
+ 

<h3 id="tupNV">函数</h3>

形式

```solidity
function <function name>(<parameter types>) {internal|external|public|private} [pure|view|payable] [returns (<return types>)]
```

> 函数的访问范围必填，使用范围建议必填，不填默认可以访问state变量，消耗gas

注意参数类型和返回值类型后跟的变量，如 ：

```solidity
contract test{
  function test01(int a) external pure returns (int b){
    b = a+1;
  }
}
```

**可以看出Solidity相对于Java声明方法：**

**可以不显式的在方法体中return，而是在方法末尾直接指定返回的变量，同时参数和返回值的类型 变量形式同时起了声明的作用，返回的形式不再严格**

<h3 id="c1NPi">变量</h3>

同样函数上声明的可以认为是Java方法内声明的，而非链上的状态变量。

<h3 id="EpNyr">返回值   <font style="color:rgb(31, 35, 40);">return 和 returns</font></h3>

如果显式的return了指定的变量名，那么就不需要在函数声明中写清变量名了，只需要写类型即可，引用数据类型书写变量名也是可选的，原因是变量名既可以被外部接收体指明，也可为“命名式返回”

```solidity
contract test{
  function test01(int a) public pure returns (int <a>,bool <_bool>,unit256[3] memory _array){
    return(1, true, [uint256(1),2,5]);
  }
}
```

<font style="color:rgb(31, 35, 40);">说明：[1,2,3]</font><font style="color:rgb(31, 35, 40);">会默认为</font><font style="color:rgb(31, 35, 40);">uint8(3)</font><font style="color:rgb(31, 35, 40);">，因此</font><font style="color:rgb(31, 35, 40);">[uint256(1),2,5]</font><font style="color:rgb(31, 35, 40);">中首个元素必须强转</font><font style="color:rgb(31, 35, 40);">uint256</font><font style="color:rgb(31, 35, 40);">来声明该数组内的元素皆为此类型。数组类型返回值默认必须用memory关键字修饰</font>



**必须在方法中写逻辑，和Java必须在PSVM中运行代码一样！！！**

<font style="color:rgb(31, 35, 40);">如果函数中显式的return了，函数声明上必须有returns及返回类型规定</font>

<font style="color:rgb(31, 35, 40);"></font>

<font style="color:rgb(31, 35, 40);"></font>

<font style="color:rgb(31, 35, 40);"></font>

