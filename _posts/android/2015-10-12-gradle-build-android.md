# 使用gradle构建android程序

## 一、什么是构建

构建,叫 build 也好,叫 make 也行。反正就是根据输入信息然后干一堆事情,最后得 到几个产出物(Artifact)。


在Gradle走红之前,常用的构建工具是 ANT,然后又进化到 Maven。ANT 和 Maven 这两个工具其实也还算方便,现在还有很多地方在使用。但是二者都有一些缺点,所以让更 懒得人觉得不是那么方便。比如,Maven编译规则是用 XML 来编写的。XML 虽然通俗易懂,但是很难在 xml 中描述 if{某条件成立,编译某文件}/else{编译其他文件}这样有不同条件的任务。

那么，gradle是怎么解决这些问题的呢？这就需要了解gradle的特点了：

 1. Gradle 使用 Groovy 语言描述如何构建工程。Groovy 基于 Java 并拓展了 Java。 Java 程序员可以无缝切换到使用Groovy开发程序。Groovy 说白 了就是把写 Java 程序变得像写脚本一样简单。写完就可以执行,Groovy 内部会将其编译成 Java class 然后启动虚拟机来执行。

 2. 除了可以用很灵活的语言来写构建规则外,Gradle 另外一个特点就是它是一种DSL,即 Domain Specific Language,领域相关语言。什么是 DSL,说白了它是某个行业中的行话。比如 sourceSets 代表源文件的集合。那么,对使用者而言,这些行话的好处是什么呢?这就是:一句行话可以包含很多意思,而且在这个行当里的人一听就懂,不用解释。另外,基于行话,我们可以建立一个模板,使用者只要往这个模板里填必须要填的内容,Gradle 就可以非常漂亮得完成工作,得到想要的东西。

所以，学习gradle需要两方面的东西:

 * Groovy,了解 Groovy 语 言是掌握 Gradle 的基础。
 * Gradle 作为一个工具,它的行话和它“为人处事”的原则。

## 二、Groovy 介绍

Groovy是在 java平台上的、具有像Python,Ruby 和 Smalltalk 语言特性的灵活动态语言, Groovy 保证了这些特性像 Java 语法一样被Java 开发者使用。


### 2.1 初识groovy
安装好groovy环境后，就可以进行简单的测试了,创建一个test.groovy文件，其中有:

```
```

运行：

```
➜  test  groovy test.groovy
hello groovy
```
和python等脚本语言十分相似

### 2.2 关于groovy的一些简单介绍

 * Groovy 注释标记和 Java 一样,支持 或者/**/

       >Groovy 中变量定义可以使用关键字 def。注意,虽然 def 不是必须的,但是为了代码清晰,建议还是使用 def 关键字
       
 ```
 def variable1 = 1 //可以不使用分号结尾
 ```
 
 * 函数定义时,参数的类型也可以不指定。
 * 除了变量定义可以不指定类型外,Groovy中函数的返回值也可以是无类型的。
 
 ```
 def nonReturnTypeFunc(){
 }
 //如果指定了函数返回类型,则可不必加 def 关键字来定义函数 
 String getString(){
 }
 ```
 
 * 字符串处理与python十分类似：
 
 ```
 单引号''中的内容严格对应Java中的String,不对$符号进行转义: 
//输入就是 I am $dolloar
def singleQuote='I am $dolloar' 
//输出 I am one dollar
def doubleQuoteWithoutDollar = "I am one dollar"
//输出 I am 1 dolloar
三个引号'''xxx'''中的字符串支持随意换行 比如 def multieLines = ''' 
	begin
	line 2
	end '''
	
 ```

* 最后,除了每行代码不用加分号外,Groovy 中函数调用的时候还可以不加括号。

```
//可以简化为println "test"
println("test")

def String printName(name, age) {
	println "hello $name, you are 22"
}

printName "chenzhe", 24

输出：hello chenzhe, you are 22
```

### 2.3 groovy中的数据类型

Groovy 中的数据类型和java相似，有两种比较有特点的:
 * Groovy 中的容器类。

#### 2.3.1 基础数据类型

```
def int x
println x.getClass().getCanonicalName()

输出： java.lang.Integer
```

#### 2.3.2 容器数据类型

* List

```
//变量定义:List 变量由[]定义,比如
//List 由[]定义,其元素可以是任何对象 变量存取:可以直接通过索引存取,而且不用担心索引越界。如果索引超过当前链表长度,List 会自动 往该索引添加元素
//结果为101
println aList.size 
```

* map

```
//变量定义:Map 变量由[:]定义,比如
//希望key为变量
def key = "chenzhe"
def aConfusedMap=[(key1):"who am i?"]
//读取：
aMap.key
aMap["key"]
```

* Range

```
def aRange = 1..5 <==Range 类型的变量 由 begin 值+两个点+end 值表示 左边这个 aRange 包含 1,2,3,4,5 这 5 个值
println aRange.from
```

#### 2.3.3 闭包

闭包,英文叫 Closure,是 Groovy 中非常重要的一个数据类型。它代表了一段可执行的代码。其外形如下:

```
def aClosure = {//闭包是一段代码,所以需要用花括号括起来..
	println "param1 is $param1; param2 is $param2" //这是代码,最后一句是返回值,也可以使用 return,和 Groovy 中普通函数一样 
}
```

定义闭包的语法为：

```
￼def xxx = {paramters -> code} //或者
```

闭包与c/c++中的函数指针相似，调用的方法为：

```
闭包对象.call(参数) 或者更像函数指针调用的方法: 闭包对象(参数)
```

#### 2.3.4 闭包的注意点

* 省略圆括号

```
public static <T> List<T> each(List<T> self, Closure closure)

def iamList = [1,2,3,4,5] //定义一个 List
}

￼def testClosure(int a1,String b1, Closure closure){ 
	//do something
	closure()
}

//调用时可以 
testClosure 1, "hello", {
	println "i am in closure"
}
```

### 2.4 文件的读写

### 2.5 操作xml









