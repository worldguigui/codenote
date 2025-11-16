在学习antlr4的使用时，我被建议去了解一下lambda演算，以下是我对教材内容的总结。<br/>

## 一.什么是lambda演算？
这里只进行原文的摘录和翻译。<br/>
Functions play a prominent role in describing the semantics of a programming language, since the meaning of a computer program can be considered as a function from input values to output values. In addition, functions play an essential role in mathematics, which means that much of the theory of functions, including the issue of computability, unfolded as part of mathematical logic before the advent of computers. In particular, Alonzo Church developed the lambda calculus in the 1930s as a theory of functions that provides rules for manipulating functions in a purely syntactic manner. <br/>
函数在描述编程语言的语义中扮演着重要角色，因为计算机程序的含义可以被视为一个从输入值到输出值的函数。此外，函数在数学中也扮演着关键角色，这意味着函数的许多理论，包括可计算性问题，在计算机出现之前是作为数理逻辑的一部分展开的。特别是，阿隆佐·邱奇在 20 世纪 30 年代发展了 λ 演算，作为一种函数理论，它提供了以纯句法方式操作函数的规则。<br/>

Although the lambda calculus arose as a branch of mathematical logic to provide a foundation for mathematics, it has led to considerable ramiﬁcations in the theory of programming languages. Beyond the inﬂuence of the lambda calculus in the area of computation theory, it has contributed important results to the formal semantics of programming languages: <br/>
尽管 λ 演算作为数理逻辑的一个分支出现，旨在为数学奠定基础，但它对编程语言理论产生了深远的影响。除了 λ 演算在计算理论领域的影响之外，它还为编程语言的形式语义学贡献了重要成果：<br/>

• Although the lambda calculus has the power to represent all computable functions, its uncomplicated syntax and semantics provide an excellent vehicle for studying the meaning of programming language concepts. <br/>
• 尽管 λ 演算有能力表示所有可计算函数，但其简单的语法和语义为研究编程语言概念的含义提供了一个极好的工具。<br/>

• All functional programming languages can be viewed as syntactic variations of the lambda calculus, so that both their semantics and implementation can be analyzed in the context of the lambda calculus. <br/>
• 所有函数式编程语言都可以被视为 λ 演算的句法变体，因此它们的语义和实现都可以在 λ 演算的背景下进行分析。<br/>

• Denotational semantics, one of the foremost methods of formal speciﬁcation of languages, grew out of research in the lambda calculus and expresses its deﬁnitions using the higher-order functions of the lambda calculus. <br/>
• 指称语义，语言形式规范的最重要方法之一，源于 λ 演算的研究，并使用 λ 演算的高阶函数来表达其定义。<br/>

In this chapter we take a brief but careful look at the lambda calculus, ﬁrst deﬁning it as a language and then viewing it as a computational formalism in light of its reduction rules. We end the chapter by implementing a lambda calculus evaluator in Prolog. In Chapter 10 we continue the study of functions with the goal of explaining recursive deﬁnitions. <br/>
在本章中，我们将简要而仔细地探讨 λ 演算，首先将其定义为一门语言，然后根据其归约规则将其视为一种计算形式体系。我们将在 Prolog 中实现一个 λ 演算求值器来结束本章。在第 10 章中，我们将继续研究函数，目标是解释递归定义。<br/>
## 二.lambda的语法
```
<expression> ::= <variable> ; lowercase identifiers
                | <constant> ; predefined objects
                | ( <expression> <expression> ) ; combinations
                | ( λ <variable> . <expression> ) ; abstractions.
```
由labmda演算具体语法的 BNF 规范中我们可以看到lambda有四种类型：变量、预定义常量、函数应用(组合)、函数定义(λ抽象)。<br/>
在书中为了方便讲解，预定义了几个常量：数字、add(用于加法)、mul(用于乘法)、succ(后继函数)、sqr(用于平方)<br/><br/>
我们接下来解释相关概念。<br/>

### 1.抽象、变量、预定义常量、后继函数、函数应用
  表达式 **①λn . (add n 1)** 表示整数上的后继函数，因此 **②((λn . (add n 1)) 5) = 6**。<br/>
 * 例子中的表达式①就是λ抽象，也就是函数定义；<br/>
 * n是变量，如x，y，z一样都是我们熟悉的变量；<br/>
 * "add" 和 1 是预定义常量，即如果我们不提前声明，形如“add”和“1”的两串字符是无法表示加与整数的。<br/>
 * 表达式①是整数上的后继函数，简单理解就是，求下一个数字。<br/>
 * 例子中的表达式②就是函数应用，也就是组合。根据定义所说，组合的格式为(E1 E2)，E1和E2都是表达式，我们也可以叫做元变量（metavariable）。这里的E1是一个抽象，E2是一个参数。<br/>

lambda演算式往往会有极其多的括号，以下记法约定允许我们适当地省略括号。<br/>
### 2.五条约定
一个有着较多括号的lambda表达式：**(λn . (λf . (λx . f (n f x)))) (λn . (λf . (λx . f (n f x))))**
#### 1) 大写字母和以大写字母开头的标识符将用作元变量，可以用来表示λ表达式和参数。
#### 2) 函数应用是左结合的。
E1 E2 E3 表示 ((E1 E2) E3) 而不是 (E1 (E2 E3))。
#### 3) 函数定义中 "λ<变量>" 的作用域尽可能向右延伸。
λx . E1 E2 E3 表示 (λx . (E1 E2 E3)) 而不是 ((λx . E1 E2) E3)。<br/>
需要注意的是，应用的优先级高于定义。对于表达式 λx. E1 E2，正确顺序为 λx. (E1 E2)，也就是说 (E1 E2) 是应用，它们优先结合。<br/>
#### 4) 抽象允许一个变量列表，该列表缩写了一系列 λ 抽象。
λx y z . E 表示 (λx . (λy . (λz . E)))
#### 5) 为表达式命名。
定义为 λ 表达式抽象的函数是匿名的，因此 λ 表达式本身表示该函数。作为一种记法约定，可以使用语法 **define <名称> = <表达式>** 来命名 λ 表达式。<br/>
给定**define Twice = λf . λx . f (f x)**，则有 **(Twice (λn . (add n 1)) 5) = 7**。<br/>


## 三.柯里化函数（Curried Functions）
如上所述的 λ 演算似乎只允许单变量函数。抽象机制一次只允许一个参数。<br/>
然而，许多有用的函数，例如二元算术运算，需要多个参数；例如，**sum(a,b) = a+b** 符合语法规范 **sum : NxN → N** （输入一个有序对，输出一个数。这个类型描述了一个加法函数的行为），其中 N 表示自然数。<br/>
针对这个问题，lambda演算有两种解决方案：1)用有序对Pair来接收多个参数。2)使用函数的柯里化版本。<br/>
### 1.有序对Pair
在lambda演算中的Pair是一种可以存储两个值的数据结构。既然lambda表达式只能接收一个参数，那么以一个Pair作为参数即可。<br/>
Pair有Head和Tail两种操作，Head表示取第一个值，Tail表示取第二个值。<br/>
可以通过使用预定义的 "cons" 操作（如 Lisp 中）来提供对，或者使用纯 λ 演算中的原始 λ 表达式来定义。<br/>
如<br/>
* <a,b> = cons a b；add <a,b><br/>
* λa . λb . add a b<br/>
### 2.柯里化函数
柯里化函数--将一个接收多个参数的函数，转换成一系列嵌套的、每个只接收一个参数的函数。<br/>
非柯里化版本：sum : NxN → N<br/>
柯里化版本：add : N → N → N （→ 是右结合的）<br/>
### 3.柯里化函数的优势
#### 1) 部分应用(partial application)<br/>
这是柯里化最强大的特性。你可以只提供一部分参数，得到一个新的、有用的函数。<br/>
如：<br/>
(add 1) 就是后继函数 succ。<br/>
因为 (add 1) 返回一个函数 λb . 1 + b，这个函数的作用正是“加一”。<br/>
#### 2) 与高阶函数完美配合<br/>
柯里化使得创建和组合函数变得极其方便。<br/>
如：<br/>
**define Twice = λf . λx . f (f x)**（将一个函数应用两次，上文提到过）<br/>
**(Twice sqr)** 得到一个“求四次方”的函数。<br/>
**(Twice (add 1))** 得到一个“加二”的函数。<br/>
你不需要预先定义 FourthPower 或 AddTwo，通过部分应用柯里化函数，可以即时生成它们。<br/>
### 4.柯里化与反柯里化（Curry and Uncurry）
函数的柯里化和反柯里化操作可以在 λ 演算中表示为：<br/>
```
deﬁne Curry = λf . λx . λy . f <x,y>
deﬁne Uncurry = λf . λp . f (head p) (tail p)
```
通过 Curry 和 Uncurry 操作，它们可以在柯里化形式和非柯里化形式（使用有序对的形式）之间进行转换，证明这两种表达方式是等价的。<br/>
因此，加法操作的两种版本有如下关系：<br/>
```
Curry sum = add and Uncurry add = sum
```
具体的证明过程教材中没有提到，只给出了结论。嗯，至少学到这里没有提及。<br/>

## 四.lambda表达式的语义（Semantics of Lambda Expressions）
一个λ表达式的含义，就是当它所有的函数应用都执行完毕后，得到的最终结果。这个过程，叫做归约（reduction）。<br/>
下部分再深入λ表达式的归约，这部分要先了解前置知识：替换。<br/>
### 1.自由变量和绑定变量
定义：变量 v 在 λ 表达式中的一次出现，如果它在 "λv" 的作用域内，则称为绑定的；否则称为自由的。<br/>
一个变量可以在同一个 λ 表达式中既绑定又自由地出现；例如，在 **λx . y λy . y x** 中，y 的第一次出现是自由的，另外两次是绑定的。<br/>
### 2.替换与安全的替换
**E[v→E1]** 指的是通过用 λ 表达式 E1 替换 E 中变量 v 的每个自由出现而得到的 λ 表达式。<br/>
如果 E1 中的自由变量没有因为替换 **E[v→E1]** 而变成绑定的，则这种替换被称为有效的或安全的。无效的替换涉及变量捕获或名称冲突。<br/>
例如，朴素的替换 **(λx . (mul y x))[y→x]** 得到 **(λx . (mul x x))** 是不安全的，因为结果表示一个平方操作，而原始的 λ 表达式并非如此。<br/>
### 3.识别E中自由变量的操作FV
定义：表达式 E 中自由变量（自由出现的变量）的集合，记作 FV(E)。<br/>
替换的定义需要小心以避免变量捕获。我们首先需要通过定义一个列出表达式中自由出现的变量的操作来识别 E 中的自由变量。例如，**FV(λx . y λy . y x z) = {y,z}**(遵循③④)。<br/>
并遵循以下规则：<br/>
#### 1) FV(c) = ∅ for any constant
#### 2) FV(x) = {x} for any variable 
#### 3) FV(E1 E2) = FV(E 1) ∪ FV(E2)
#### 4) FV(λx . E) = FV(E) - {x} 
没有自由变量（FV(E) = ∅）的 λ 表达式 E 被称为封闭的。
### 4.替换的几种情况
#### 1) v[v→E1] = E1 for any variable v
#### 2) x[v→E1] = x for any variable x≠v
#### 3) c[v→E1] = c for any constant c
#### 4) (Erator Erand)[v→E1] = ((Erator[v→E1]) (Erand[v→E1]))
#### 5) (λv . E)[v→E1] = (λv . E)
#### 6) (λx . E)[v→E1] = λx . (E[v→E1]) when x≠v and x∉FV(E1)
#### 7) (λx . E)[v→E1] = λz . (E[x→z][v→E1]) when x≠v and x∈FV(E1),where z≠v and z∉FV(E E1)
```
Example : (λy . (λf . f x) y)[x→f y]
(λy . (λf . f x) y)[x→f y]
= λz . ((λf . f x) z)[x→f y]         by 7) since y∈FV(f y)
= λz . ((λf . f x)[x→f y] z[x→f y])  by 4)
= λz . ((λf . f x)[x→f y] z)         by 2)
= λz . (λg . (g x)[x→f y]) z         by 7) since f∈FV(f y)
= λz . (λg . g (f y)) z              by 4), 2), and 1) 
```
情况1很好理解，把变量v直接替换成E1。<br/>
情况2、3，不作替换。<br/>
情况4，可以理解为乘法分配律。对操作符和操作数分别进行替换。<br/>
情况5，变量v是绑定的，所以不受替换影响，不会改变。<br/>
情况6，变量x是绑定的，但是与目标变量v不同，所以可以安全地替换。<br/>
情况7，变量x是绑定地，且与目标变量v不同，但x在E1中是自由的，直接替换就会让 x 被外层的 λx 错误地捕获，改变其语义（x 在 E1 中从自由变成绑定）。所以需要将抽象体的绑定变量 x 重命名为一个全新的变量，例如 z。然后再进行替换。<br/>
比如说，λ表达式：**(λx. y x) [y → (f x)]**。直接替换：**(λx. f x x)**。先重命名：**(λz. y z) [y → (f x)]**。再替换：**(λz. f x z)**。<br/>

## 五.λ归约（Lambda Reduction）
这里有个简单的概念--**可归约式**，就是可以被归约的表达式，β-可归约式，就是可以被β归约的表达式。<br/>
### 1.α归约
作用：更改表达式中绑定变量的名称。<br/>
定义：如果 v 和 w 是变量且 E 是 λ 表达式，**λv . E ⇒ α λw . E[v→w]**（把v重命名为w）。<br/>

### 2.β归约
作用：将一个函数应用到一个参数。<br/>
定义：如果 v 是变量且 E 和 E1 是 λ 表达式，**(λv . E) E 1 ⇒ β E[v→E 1]**。我自己理解的是，相当于有一个函数f(x)=g(x)+1，β归约就是要把具体的g(x)带入，求得f(x)。在lambda表达式中叫做替换。<br/>
比如说对于表达式：**((λx. λy. add x y) 2) 3**。<br/>
①识别可归约式：**((λx. λy. add x y) 2)**。<br/>
②应用规则：**(λx. E) Arg ⇒β E[x→Arg]**。<br/>
③这里 **E = (λy. add x y)， Arg = 2**。<br/>
④结果：**(λy. add 2 y) 3**。<br/>
接下来就是以同样的步骤，继续识别可归约式、计算，直到不可β归约为止。<br/>
为了在 λ 表达式上定义"相等"关系，我们也允许 β-归约规则反向工作。<br/>
定义：反转 β-归约产生 β-抽象规则，**E[v→E1] ⇒ β (λv . E) E1**。<br/>
比如说：<br/>
正向β-归约： **(λv . E) E1 ⇒β E[v→E1]**。<br/>
这是计算的方向，将函数调用简化。<br/>
反向β-归约（β-抽象）： **E[v→E1] ⇒β (λv . E) E1**。<br/>
这是构造的方向，将表达式复杂化，包装成一个函数调用。<br/>
因为有了正向与反向的β归约，我们可以说 **(λv . E) E1** 和 **β E[v→E1]**，是相等的。<br/>
### 3.η归约
作用：去掉多余的绑定变量。<br/>
因为在lambda表达式中，两个函数如果对所有的输入都产生相同的输出，那么它们就是同一个函数。基于此，我们可以进行η归约。<br/>
定义：如果 v 是变量，E 是 λ 表达式（表示一个函数），并且 v 在 E 中没有自由出现，**λv . (E v) ⇒η E**。<br/>
比如说：<br/>
归约前：**λx. (sqr x)** （一个接收x并将其传给sqr的函数）。<br/>
归约后：**sqr** 。（sqr函数本身）<br/>

### 4.δ归约
作用：进行最终的计算。<br/>
定义：如果 λ 演算有预定义常量（即，如果它不是纯的），与那些预定义值和函数相关的规则被称为 δ 规则。这就相当于把具体的x=1值带入到f(x)=x+1函数中，然后得到了f(x)=2。<br/>
例如，**(add 3 5) ⇒ δ 8** 和 **(not true) ⇒ δ false**。<br/>

### 5.树形结构
我们在进行lambda表达式的归约时，最重要的是寻找β可归约式，它可以是如下所示的树形结构。<br/>
<img width="207" height="108" alt="image" src="https://github.com/user-attachments/assets/0aa42c6e-a510-4b3e-ba82-91b8ca6bfca8" /><br/>

## 六.归约策略
这一节主要解决一个问题：当表达式中有多个可归约式时，我们应该先归约哪一个？<br/>
在回答这个问题之前，我们要先明确：操作 λ 表达式的主要目标是将其归约为一个"最简单的形式"，并将其视为 λ 表达式的值。<br/>
定义：如果 λ 表达式不包含 β-可归约式（在应用 λ 演算中也不包含 δ-规则），则它处于规范形式，因此无法使用 β-规则或 δ-规则进一步归约。<br/>
规范形式和归约策略的概念可以通过提出四个问题来研究：<br/>
### 1.每个 λ 表达式都能归约为规范形式吗？
答案是否定的。<br/>
这个 β-可归约式归约为自身，意味着唯一的归约路径永远继续：<br/>
```
(λx . x x) (λx . x x) ⇒β
(λx . x x) (λx . x x) ⇒β (λx . x x) (λx . x x) ⇒β …
```
### 2.归约一个特定的 λ 表达式是否有多种方法？
答案是肯定的。<br/>
我们既可以用不同的归约顺序得到相同的结果：<br/>
```
Example : (λx . λy . (add y ((λz . (mul x z)) 3))) 7 5
Path 1: (λx . λy . (add y ((λz . (mul x z)) 3))) 7 5
⇒β (λy . (add y ((λz . (mul 7 z)) 3))) 5
⇒β (add 5 ((λz . (mul 7 z)) 3))
⇒β (add 5 (mul 7 3 )) ⇒δ (add 5 21 ) ⇒δ 26
Path 2: (λx . λy . (add y ((λz . (mul x z)) 3))) 7 5
⇒β (λx . λy . (add y (mul x 3))) 7 5
⇒β (λy . (add y (mul 7 3))) 5
⇒δ (λy . (add y 21)) 5 ⇒β (add 5 21 ) ⇒δ 26
```
也可以用不同的归约顺序得到完全不同的结果：
```
Example : (λy . 5) ((λx . x x) (λx . x x))
Path 1: (λy . 5) ((λx . x x) ( λx . x x)) ⇒β 5
Path 2: (λy . 5) ((λx . x x) (λx . x x) )
⇒β (λy . 5) ((λx . x x) (λx . x x) )
⇒β (λy . 5) ((λx . x x) (λx . x x) ) …
```

这两种归约策略分别叫：规范序规约和应用序归约。<br/>
**规范序归约**normal order reduction，总是首先归约最左边最外层的 β-可归约式（或 δ-可归约式）。<br/>
**应用序归约**applicative order reduction，总是首先归约最左边最内层的 β-可归约式（或 δ-可归约式）。<br/>
<br/>
**如何区分表达式的最外层和最内层呢？**
定义：λ 表达式中的 β-可归约式，如果其外部没有 β-可归约式，则是最外层的；如果其内部没有 β-可归约式，则是最内层的。<br/>
对于任何形式为 E = ((λx . B) A) 的表达式（即一个β-可归约式）：<br/>
E 在 B 或 A 中出现的任何β-可归约式的“外面”。<br/>
B 或 A 中的β-可归约式在 E 的“里面”。<br/>
就像套娃一样。<br/>
举例说明：
```
lambda表达式：(((λx . λy . (add x y)) ((λz . (succ z)) 5)) ((λw . (sqr w)) 7))
最左最外可归约式：((λx . λy . (add x y)) ((λz . (succ z)) 5))
最左最内可归约式：((λz . (succ z)) 5)
```
### 3.如果有多种归约策略，每种策略是否都会得到相同的规范形式表达式？
答案是肯定的。<br/>
Church-Rosser Theor em I : For any lambda expressions E, F, and G, if E ⇒ F and E ⇒ G, there is a lambda expression Z such that F ⇒* Z and G ⇒* Z<br/>
Church-Rosser定理Ⅰ证明λ演算具有 汇合性。<br/>
E ⇒* F 表示通过零次、一次或多次归约（α或β），可以从 E 到达 F。<br/>
定理：对于任何λ表达式 E, F, G，如果：<br/>
E ⇒* <br/>
E ⇒* G<br/>
那么存在某个λ表达式 Z，使得：<br/>
F ⇒* Z<br/>
G ⇒* Z<br/>
这个关系可以用图表表示：<br/>
<img width="110" height="88" alt="image" src="https://github.com/user-attachments/assets/021eff80-7955-4556-8270-6f5bb627bc1e" /><br/>
现在假设我们有一个表达式 E，并且它有两个不同的规范形式 N1 和 N2（即无法再归约的表达式）。<br/>
根据定义：<br/>
E ⇒* N1 （规范序归约）<br/>
E ⇒* N2 （应用序归约）<br/>
根据Church-Rosser定理，存在某个 Z 使得：<br/>
N1 ⇒* Z<br/>
N2 ⇒* Z<br/>
但是！N1 和 N2 都是规范形式，意味着它们不能再被归约。<br/>
因此唯一的可能是：<br/>
N1 ⇒* Z 必须是零步归约，所以 Z = N1<br/>
N2 ⇒* Z 必须是零步归约，所以 Z = N2<br/>
由此得出：N1 = N2<br/>
### 4.是否存在一种归约策略能保证产生规范形式表达式？
答案是否定的。<br/>
Church-Rosser Theor em II : For any lambda expressions E and N, if E ⇒ N where N is in normal form, there is a normal order reduction from E to N<br/>
Church-Rosser 定理Ⅱ指出规范序归约将产生规范形式 λ 表达式（如果存在的话）。<br/>
规范序归约可以有以下两种结果之一：<br/>
**1. 它达到一个唯一的（在 α-转换意义下）规范形式 λ 表达式。** <br/>
**2. 它永不终止。** <br/>
不幸的是，没有算法方法可以确定对于任意 λ 表达式这两种结果中的哪一种会发生。<br/>
λ表达式和图灵机已经被证明是等价的，那么，图灵机中存在的停机问题，在λ表达式中也理应存在，这也为 Church 定理Ⅱ提供了可信度。<br/>

## 七.与参数传递的关联
抽象 λx . B 是一个匿名函数，其形式参数是 x，函数体是 λ 表达式 B。<br/>
<br/>
1. 按名调用与规范序归约相同，除了在抽象（函数体）内的 λ 表达式中的可归约式不会被归约。使用按名调用时，实际参数作为未求值的表达式传递，每次引用相应的形式参数时，在执行函数的函数体中对它进行求值。通过选择最左边的可归约式来确保规范序，该可归约式始终是一个具有未求值操作数的最外层可归约式。<br/>
2. 按值调用与应用序归约相同，除了在抽象内的 λ 表达式中的可归约式不会被归约。这个限制对应于一个原则：函数体在函数被调用（在 β-归约中）之前不会被求值。应用序意味着函数的参数在函数被应用之前被求值。<br/>
```
Example : The call by value reduction of
(λx . (λf . f (succ x)) (λz . (λg . (λy . (add (mul (g y) x)) z)))) ((λz . (add z 3)) 5)
proceeds as follows:
(λx . (λf . f (succ x)) (λz . (λg . (λy . (add (mul (g y) x))) z))) ((λz . (add z 3)) 5)
⇒β (λx . (λf . f (succ x)) (λz . (λg . (λy . (add (mul (g y) x))) z))) (add 5 3 )
⇒δ (λx . (λf . f (succ x)) (λz . (λg . (λy . (add (mul (g y) x))) z))) 8
⇒β (λf . f (succ 8)) (λz . ( λg . ( λy . (add (mul (g y) 8))) z))
⇒β (λz . (λg . (λy . (add (mul (g y) 8))) z)) (succ 8 )
⇒δ (λz . (λg . (λy . (add (mul (g y) 8))) z)) 9
⇒β (λg . (λy . (add (mul (g y) 8))) 9)
```
