---
title: LaTeX伪代码
date: 2022-10-23 9:58:47
tags: LaTeX
categories: LaTeX
top_img: /img/post2/top_img.JPG
cover: /img/post2/top_img.JPG
description: 在 LaTeX 中排版算法或者伪代码
---
# LaTeX伪代码

## 介绍

在LaTeX中排版算法或者伪代码有以下几种选择：

- 使用algorithm包并选择algpseudocode OR compatible OR algorithmic其中一个包排版算法体。
- 使用algorithm与algorithmicx包并选择algpseudocode OR algcompatible OR  algpascal OR algc其中一个包排版算法。
- 使用algorithm2e包排版算法。

注意:上述不同组包不可混用，否则编译会出错。

### algorithms包

[algorithms使用文档.](http://mirror.ox.ac.uk/sites/ctan.org/macros/latex/contrib/algorithms/algorithms.pdf)

> The **algorithms** package provides two environments, **algorithmic** and **algorithm**, which are designed to be used together but may, depending on the necessities of the user, be used separately.
>
> The algorithmic environment provides an environment for describing algorithms and the algorithm environment provides a “float” wrapper for algorithms (implemented using algorithmic or some other method at the users’s option). The reason for two environments being provided is to allow the user maximum flexibility

algorithms包提供了两种环境**algorithm** 与 **algorithm**.

#### algorithm

algorithm是算法的**float warpper**，类似于table,  figure这样的们命令，使算法部分成为一个浮动体，防止它被分成两页。其官方介绍如下：

用法如下：

```tex
\begin{algorithm}
    \caption{Algorithm caption}
    \label{alg:algorithm-label}
    \begin{algorithmic}
        ... Your pseudocode ...
    \end{algorithmic}
\end{algorithm}
```

#### algorithmic

algorithm用于描述算法体。包含的基本命令如下：

```tex
\STATE <text>
\IF{<condition>} \STATE {<text>} \ELSE \STATE{<text>} \ENDIF
\IF{<condition>} \STATE {<text>} \ELSIF{<condition>} \STATE{<text>} \ENDIF
\FOR{<condition>} \STATE {<text>} \ENDFOR
\FOR{<condition> \TO <condition> } \STATE {<text>} \ENDFOR
\FORALL{<condition>} \STATE{<text>} \ENDFOR
\WHILE{<condition>} \STATE{<text>} \ENDWHILE
\REPEAT \STATE{<text>} \UNTIL{<condition>}
\LOOP \STATE{<text>} \ENDLOOP
\REQUIRE <text>
\ENSURE <text>
\RETURN <text>
\PRINT <text>
\COMMENT{<text>}
\AND, \OR, \XOR, \NOT, \TO, \TRUE, \FALSE
```

注意：LaTeX中的命令区分大小写，algorithmic包中命令都是全大写。

为了适应不同语言，所有的关键字输出形式都可以重定义，例如：

```tex
\floatname{algorithm}{Procedure}
\renewcommand{\algorithmicrequire}{\textbf{Input:}}
\renewcommand{\algorithmicensure}{\textbf{Output:}}
```

algorithmic包没有提供函数关键字的支持。

### algorithmicx

[algorithmicx使用文档.](https://mirrors.harcombe.net/tex-archive/macros/latex/contrib/algorithmicx/algorithmicx.pdf)

> The package algorithmicx itself doesn’t define any algorithmic commands, but gives a set of macros to define such a command set. You may use only algorithmicx, and define the commands yourself, or you may use one of the predefined command sets. 
>
> These predefined command sets (layouts) are: 
>
> **algpseudocode** has the same look1 as the one defined in the algorithmic package. The main difference is that while the algorithmic package doesn’t allow you to modify predefined structures, or to create new ones, the algorithmicx package gives you full control over the definitions (ok, there are some limitations — you can not send mail with a, say, \For command). 
>
> **algcompatible** is fully compatible with the algorithmic package, it should be used only in old documents. 
>
> **algpascal** aims to create a formatted pascal program, it performs automatic indentation (!), so you can transform a pascal program into an algpascal algorithm description with some basic substitution rules. 
>
> **algc** – yeah, just like the algpascal. . . but for c. . . This layout is incomplete. 
>
> To create floating algorithms you will need algorithm.sty. This file may or may not be included in the algorithmicx package. You can find it on CTAN, in the algorithmic package

algorithmicx包本身没有定义任何algorithmic命令，但是给了一些宏去定义这样的命令，使用algorithmicx包时可以选择自定义命令或者自定义区块，另外algorithmicx包预定义了一些命令集，如algpseudocode、algcompatible、algpascal、algc，这些命令集中的命令也可以修改。一般algpseudocode使用的比较多。

#### algpseudocode

基础命令如下：

Statement (\State 会换行，可以用在其他命令之前)

```tex
\State $x\gets <value>$
```

三种 if-statements:

```tex
\If{<condition>} <text> \EndIf
\If{<condition>} <text> \Else <text> \EndIf
\If{<condition>} <text> \ElsIf{<condition>} <text> \Else <text> \EndIf
```

Loops:

```tex
\For{<condition>} <text> \EndFor
\ForAll{<condition>} <text> \EndFor
\While{<condition>} <text> \EndWhile
\Repeat <text> \Until{<condition>}
\Loop <text> \EndLoop
```

Pre- and postcondition:

```tex
\Require <text>
\Ensure <text>
```

Functions

```tex
\Function{<name>}{<params>} <body> \EndFunction
\Return <text>
\Call{<name>}{<params>}
```

Comments:

```
\Comment{<text>}
```

注意：algorithmicx包中命令为只有首字母大写，且定义了函数语句。

示例：

```tex
\begin{algorithm}
    \caption{Euclid’s algorithm}\label{euclid}
    \begin{algorithmic}[1]
    \Function{Euclid}{$a,b$}\Comment{The g.c.d. of a and b}
    \State $r\gets a\bmod b$
    \While{$r\not=0$}\Comment{We have the answer if r is 0}
    \State $a\gets b$
    \State $b\gets r$
    \State $r\gets a\bmod b$
    \EndWhile\label{euclidendwhile}
    \State \textbf{return} $b$\Comment{The gcd is b}
    \EndFunction
    \end{algorithmic}
\end{algorithm}
```

结果：

![](/img/post2/al1.PNG)

### algpseudocodex

[algpseudocodex使用文档.](https://mirror.apps.cam.ac.uk/pub/tex-archive/macros/latex/contrib/algpseudocodex/algpseudocodex.pdf)

algpseudocodex包的基本使用与algorithmicx包中的algpseudocode相同。

### algorithm2e

[algorithm2e使用文档.](https://mirror.ox.ac.uk/sites/ctan.org/macros/latex/contrib/algorithm2e/doc/algorithm2e.pdf)

algorithm2e 包允许大量自定义排版算法。与 algorithmic 不同，algorithm2e 为算法提供了很多的定制选项，以适应各种用户的需求。 通常，\begin{algorithm} 和 \end{algorithm} 之间的用法是

1. Declaring a set of keywords(to typeset as functions/operators), layout controls, caption, title, header text (which appears before the algorithm's main steps e.g.: Input,Output)
2. Writing the main steps of the algorithm, with each step ending with a \;
   This may be taken in analogy with writing a latex-preamble before we start the actual document.

包加载如下：

```tex
\usepackage[options]{algorithm2e}
```

可选参数options有[Hhtbp]，使用H参数算法排版不再是浮动体，如果空间不足，前后会留出空白。

用法示例：

```tex
\documentclass{article}

\usepackage[ruled, vlined, linesnumbered]{algorithm2e}
\begin{document}
\begin{algorithm}[t]
    \caption{How to write algorithms}
    \begin{small}
        \BlankLine
        \KwData{this text}
        \KwResult{how to write algorithm with \LaTeX2e }
        initialization\;
        \While{not at end of this document}{
        read current\;
        \eIf{understand}{
        go to next section\;
        current section becomes this one\;
        }{
        go back to the beginning of current section\;
     }
    }
    \end{small}       
\end{algorithm} 
\end{document}
```

结果：

![](/img/post2/al2.PNG)

## 算法跨页展示

上述方式无论是将算法体设置为浮动体还是设置为固定位置，有时候会排版不佳，有些包中也提供跨页的解决方法。这里提供另一种方式，在引言区定义新环境breakablealgorithm：

```tex
\makeatletter
\newenvironment{breakablealgorithm}
  {% \begin{breakablealgorithm}
   \begin{center}
     \refstepcounter{algorithm}% New algorithm
     \hrule height.8pt depth0pt \kern2pt% \@fs@pre for \@fs@ruled
     \renewcommand{\caption}[2][\relax]{% Make a new \caption
       {\raggedright\textbf{\fname@algorithm~\thealgorithm} ##2\par}%
       \ifx\relax##1\relax % #1 is \relax
         \addcontentsline{loa}{algorithm}{\protect\numberline{\thealgorithm}##2}%
       \else % #1 is not \relax
         \addcontentsline{loa}{algorithm}{\protect\numberline{\thealgorithm}##1}%
       \fi
       \kern2pt\hrule\kern2pt
     }
  }{% \end{breakablealgorithm}
     \kern2pt\hrule\relax% \@fs@post for \@fs@ruled
   \end{center}
  }
\makeatother
```

将algorithm环境替换为breakablealgorithm环境即可，即为：

```tex
\begin{breakablealgorithm}
    \caption{Algorithm caption}
    \label{alg:algorithm-label}
    \begin{algorithmic}
        ... Your pseudocode ...
    \end{algorithmic}
\end{breakablealgorithm}
```

更多内容参见包手册。

## 参考文献

[1] [wikibooks LaTeX/Algorithms](https://en.wikibooks.org/wiki/LaTeX/Algorithms#Typesetting_using_the_algorithmic_package)

[2] [Algorithm tag and page break](https://tex.stackexchange.com/questions/33866/algorithm-tag-and-page-break)

