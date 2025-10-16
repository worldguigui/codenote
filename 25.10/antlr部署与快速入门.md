# antlr 4的部署与快速入门

### 对于idea来说
针对于Intellij Idea的antlr4的安装部署十分简单, 在Settings->Plugins->marketplace中直接搜索antlr v4并安装, 随后跟着提示重启ide即可.
### 对于windows来说, 稍稍复杂一点
对于windows来说,如果你想在命令行中进行antlr4的使用的话.
1. 从antlr的官网上下载jar包--antlr-4.13.2-complete.jar
2. 把该jar包复制到D:/antlr4目录下
3. 在D:/antlr4/bat目录下创建两个批处理文件antlr4.bat和grun.bat
antlr4.bat中写入 java org.antlr.v4.Tool %*
grun.bat中写入 java org.antlr.v4.gui.TestRig %*
4. 向环境变量CLASSPATH(没有则创建)中写入两个路径, "D:/antlr4/antlr-4.13.2-complete.jar" 和 "." 
5. 向环境变量Path中写入一个路径, "D:/antlr4/bat"


当你完成这五个步骤后,在cmd中输入antlr4后,理应会出现如下的说明:
```bash
ANTLR Parser Generator  Version 4.13.2
 -o ___              specify output directory where all output is generated
 -lib ___            specify location of grammars, tokens files
 -atn                generate rule augmented transition network diagrams
 -encoding ___       specify grammar file encoding; e.g., euc-jp
 -message-format ___ specify output style for messages in antlr, gnu, vs2005
 -long-messages      show exception details when available for errors and warnings
 -listener           generate parse tree listener (default)
 -no-listener        don't generate parse tree listener
 -visitor            generate parse tree visitor
 -no-visitor         don't generate parse tree visitor (default)
 -package ___        specify a package/namespace for the generated code
 -depend             generate file dependencies
 -D<option>=value    set/override a grammar-level option
 -Werror             treat warnings as errors
 -XdbgST             launch StringTemplate visualizer on generated code
 -XdbgSTWait         wait for STViz to close before continuing
 -Xforce-atn         use the ATN simulator for all predictions
 -Xlog               dump lots of logging info to antlr-timestamp.log
 -Xexact-output-dir  all output goes into -o dir regardless of paths/package
 ```

Java虚拟机会在CLASSPATH的路径下寻找可能存在的.class文件和jar包, 第二个路径"."表示虚拟机会在执行java命令所在的目录下寻找.class文件和jar包. 
windows系统会在Path下的路径寻找可能需要执行的可执行文件. 而我们编写的.bat文件就是可执行文件的一种. 在命令行输入antlr4后, 系统会找到antlr4.bat并执行.

6. 在D:/antlr4/temp/test目录下创建Hello.g4文件, 文件内容为:
```
grammar Hello;
r : 'hello' ID ;
ID : [a-z]+ ;
WS : [ \t\r\n]+ -> skip ;
```
7. 在命令行执行 antlr4 Hello.g4 后, 便会看到当前目录下由ANTLR工具命令生成了一个由HelloParser.java和HelloLexer.java组成的,可以运行的语法识别程序, 不过, 我们还缺少一个main程序来触发这个语言识别的过程, 而ANTLR在运行库中提供了一个名为TestRig的方便的调试工具, 就是刚刚的 grun.bat
8. 我们已经有一个grun的命令, 所以键入 grun Hello r -tokens, 然后再 Hello java ctrl+z(EOF), 最终输出如下:
```
hello java
^Z
[@0,0:4='hello',<'hello'>,1:0]
[@1,6:9='java',<ID>,1:6]
[@2,12:11='<EOF>',<EOF>,2:0]
```

执行 antlr4.bat时, 其中的 %* 表示"所有参数". 所以实际执行的是java org.antlr.v4.Tool Hello.g4, 而grun命令同理.

每行输出代表了一个词法符号, 其中包含了该词法符号的所有信息. 
[@0,0:4='hello',<'hello'>,1:0] 表示 这个词法符号位于第一个位置(从0开始计数), 由输入文本的第0个到第4个位置之间的字符组成, 包含的文本内容是'hello', 词法类型是'hello', 位于输出文本的第1行, 第0个位置开始.