# antlr4基础

在进行了antlr的部署后，我们开始进入到真正的学习中。<br/>
通读*The Definition ANTLR 4 Reference*(《ANTLR 4 权威指南》)这本书后，我认为主要从两方面入手，是什么和怎么用。<br/>

### 是什么

antlr是一个强大的语法分析器生成工具，可以用来读取，处理，执行和翻译结构化的文本或二进制文件。<br/>
以上是官方的定义，具体可以自行去查。除此之外，antlr遵循着一套固定的处理流程。<br/>
<br/>
输入的字符流 **->**<br/>
lexer将字符聚集为token(词法符号，包含词法类型和对应的文本两部分) **->**<br/>
parser将token按照规则，默认情况下，建造为语法分析树 **->**<br/>
通过Java程序进一步操作语法分析树<br/>

### 怎么用
通读整本书，我认为要从两方面去学习antlr，一是要学习如何编写g4语法文件，这包括g4的语法和结构；<br/>
二是要学习如何将antlr生成的语法分析树与你熟悉的语言结合起来，antlr生成的语法分析树就是一个树状结构，这要求我们用Java代码来对树进行遍历并在遍历过程中对数据进行处理。<br/>
值得注意的是，g4代表的词法语法分析过程与Java应用程序这两部分会尽可能地解耦。这就意味着针对不同语言，有不同的应用程序实现，而无需额外修改g4文件，这便是为什么我认为要从两方面去学习。<br/>

## 一.如何编写g4文件
可以像导包一样根据文件名导入g4文件，假如我们有一个Number.g4，在其他g4中我们就可以
```
// 导报是一个跟编写规则无关的语法，所以写在开头
/* antlr支持的注释与java基本一致 */
/** 当然包括doc形式的注释 */
import Number.g4
```
以避免重复的规则编写。<br/>
### 1.简单的规则
lexer是词法分析器，parser是语法分析器，它们运行都遵循着各自的规则。<br/>
举例来说的话，“我学Java”,这是一句话，主谓宾是一条语法规则；规定了什么词是主语，什么词是谓语，这就是词法规则。<br/>
其实这些概念并没有太严格的区分，不要在没必要的地方太过纠结。<br/>
语法规则习惯用小写字母开头表示，词法规则习惯用大写字母开头表示。<br/>
接下来展示语法和词法的简单定义。<br/>
<br/>

**语法规则**<br/>
stat: ID ;<br/>
json：ID '=' INT ;<br/>
**词法规则**<br/>
ID: [a-zA-Z]+ ;<br/>
INT: [1-9][0-9]* ;<br/>
<br/>
可以看到，stat匹配的是一个单独的词法组成的语法，而json匹配的是形如ID=INT的语法，其中，ID是一个任意字母组成的组合，INT是一个不以0开头的数字。<br/>
一般来说，antlr允许使用正则表达式来匹配字符。<br/>

### 2.具有多个备选分支的规则
stat: ID <br/>
    | INT <br/>
    | GENDER ;    //性别不止一个<br/>
<br/>
GENDER: 'MALE'<br/>
      | 'FEMALE' <br/>
      |           //什么都不匹配<br/>
      ;<br/>
ID: ('a'..'z' | 'A'..'Z')+ ;<br/>

<br/>
我们通常用符号 | 来表示多个备选分支, 用()来组成子规则，'a'..'z'是范围表达式，表示从a到z。<br/>
+ 表示出现一次或多次<br/>
* 表示不出现或出现多次<br/>
? 表示出现零次或一次<br/>
.* 表示贪婪匹配<br/>
.*? 表示非贪婪匹配<br/>
<br/>

### 3.带标签的规则
GENDER: 'MALE'    # Male<br/>
      | 'FEMALE'  # Female<br/>
      |           # Empty<br/>
      ;<br/>
当我们想知道规则具体匹配到哪个分支的时候，我们就可以在分支后面通过#来添加标签，将大的规则拆分为多个小的规则，每个小规则都有对应的名称。<br/>
<br/>

### 4.带递归的规则
expr: expr ('*' |'/') expr<br/>
    | INT ;<br/>
或<br/>
expr: mul<br/>
    | INT;<br/>

mul: expr ('*' |'/') expr<br/>

antlr可以处理直接左递归(前一个)，但却无法处理间接右递归(后一个)。<br/>

### 5.带fragent的规则
fragment<br/>
DIGIT: [0-9] ;<br/>

被fragment标记的词法规则，fragent会告诉antlr这条规则只能被其他规则使用，不用被分析。<br/>

### 6.带动作的规则


## 二.如何编写Java应用程序
### 1.简单的实现
```
// 这是一个简单的匹配赋值语句的语法文件
grammar Expr ;
stat: assign ;
assign: ID '=' expr ;
expr: ID '(' ')'
| INT ;
ID: [a-zA-Z]+ ;
INT: [1-9][0-9]* ;

// 这是一个简单的java程序实现
class Main{
    public static void main(String[] args) {
        ANTLRInputStream input = new ANTLRInputStream(System.in); // 接受用户输入
		ExprLexer lexer = new ExprLexer(input);
		CommonTokenStream tokens = new CommonTokenStream(lexer);
		ExprParser parser = new ExprParser(tokens);
		ParseTree tree = parser.stat();
		System.out.println(tree.toStringTree(parser));
    }
}
```
这一简单的程序遵循着stream->lexer->tokens->parser->tree这样标准的流程。<br/>
antlr的自动工具可以帮助我们生成非常多的相关类，包括上面的ExprLexer和ExprParser，接下来马上就提到的上下文对象--Context，以及第三四节的Listener和Vistor。<br/>
### 2.文法树的结构
antlr根据语法文件生成的文法树ParseTree，它派生出两个子接口，RuleNode和TerminalNode，分别表示着子树的根节点与叶子节点。<br/>
antlr的自动化工具会根据规则帮助我们生成上下文对象--Context，如StatContext，AssignContext，ExprContext。这些对象包括着词法识别到的所有信息，如在AssignContext中，包含着访问标识符节点的方法ID(),和代表着表达式的子树的方法expr()。<br/>

### 3.监听者模式遍历
当某一事件发生，绑定在其身上的监听器就会被触发，进而调用回调函数，这就是监听模式的大概思路。<br/>
antlr根据每个规则，为我们生成了对应的enter()和exit()方法，以对应的Context对象为参数，并将它们包装在默认实现的ExprBaseListener中。如enterAssign()和exitAssign()，分别在进入和离开assign时触发，并以AssignContext作为传参。<br/>


### 4.访问者模式遍历
