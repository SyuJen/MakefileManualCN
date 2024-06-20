# GNU make 手册 个人中译版

Copyright (C)  2024  JohnSyu.
Permission is granted to copy, distribute and/or modify this document
under the terms of the GNU Free Documentation License, Version 1.3
or any later version published by the Free Software Foundation;
with no Invariant Sections, no Front-Cover Texts, and no Back-Cover
Texts.  A copy of the license is included in the section entitled ``GNU
Free Documentation License''.

# 1 *make* 概述

*make* 实用程序自动确定大型程序的哪些部分需要重新编译，并发出重新编译它们的命令。本手册描述了由 Richard Stallman 和 Roland McGrath 实现的 GNU *make*。自3.76版本以来的开发一直由 Paul D.Smith 负责。

GNU *make*符合 IEEE 标准 1003.2-1992(POSIX.2)的第6.2节。

我们的例子显示了 C 程序，因为它们是最常见的，但您可以将 *make* 与任何可以使用 shell 命令运行的编译器的编程语言一起使用。事实上，*make* 并不局限于程序。您可以使用它来描述其中某些文件必须在其他文件更改时自动从其他文件更新的一切任务。

**准备和运行 *make***

要准备使用 *make*，必须编写一个名为 *makefile* 的文件，该文件描述程序中文件之间的关系，并提供更新每个文件的命令。在程序中，通常从目标文件(object files)更新可执行文件(executable file)，而目标文件又是通过编译源文件生成的(source files)。

一旦存在合适的makefile，每次更改一些源文件时，这个简单的shell命令：

```sh
make
```

足以执行所有必要的重新编译。*make* 程序使用 *makefile* 数据库和文件的上次修改时间来决定哪些文件需要更新。对于这些文件中的每一个，它都会发布数据库中记录的配方(recipe)。

您可以提供命令行参数给 *make*，来控制应重新编译哪些文件或如何重新编译。请参阅 [9 How to Run make](https://www.gnu.org/software/make/manual/make.html#Running)。

## 1.1 How to Read This Manual
（暂略）

## 1.2 Problems and Bugs
（暂略）

# 2 An Introduction to Makefiles
你需要一个名为 *makefile* 的文件来告诉 *make* 该做什么。通常，*makefile* 会告诉 *make* 如何编译和链接程序。

在本章中，我们将讨论一个简单的 *makefile*，该文件描述如何编译和链接由8个C源文件和3个头文件组成的文本编辑器。*makefile* 还可以告诉 *make* 如何在明确要求时运行其他命令（例如，作为清理操作删除某些文件）。要查看生成文件的更复杂示例，请参阅 [Appendix C Complex Makefile Example](https://www.gnu.org/software/make/manual/make.html#Complex-Makefile)。

当 *make* 重新编译编辑器时，必须重新编译每个更改过的C源文件。如果头文件发生了更改，则必须重新编译包含该头文件的每个C源文件，以确保安全。每次编译都会生成一个与源文件相对应目标文件。最后，如果重新编译了任何源文件，则所有目标文件，无论是新创建的还是从以前的编译中保存的，都必须链接在一起，以生成新的可执行编辑器（译者注，这里原文就是 new executable editor，虽然我感觉 editor 这个词怪怪的）。

## 2.1 一条规则的样式

一个简单的 *makefile* 由以下形状的“规则(rule)”组成：

```makefile
target ... : prerquisited ...
    recipe
    ...
    ...
```
目标(target)通常是程序生成的文件的名称；目标可以是可执行文件或目标文件。目标也可以是要执行的操作的名称，例如“clean”，参见 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)

先决条件(prerequisites)是用作创建目标的输入的文件。一个目标通常依赖于多个文件。

配方(recipe)是使执行的操作。一个配方可能有多个命令，要么在同一行，要么每个命令在自己的行。**请注意**：您需要在每个配方行的开头放置一个制表符！这是一个引起粗心的模糊。如果您更喜欢用制表符以外的字符作为配方的前缀，您可以将 `.RECIPEPREFIX` 变量设置为备用字符（请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)）。

（译者注：在本文中将 recipe 翻译为配方，之前想将其翻译为“指令”，但和 directive 的翻译会发生重合。由于文档内容较多，校对复杂，部分位置还是保留了将 recipe 翻译为 指令 的）。

通常，配方位于具有先决条件的规则中，如果任何先决条件发生更改，则用于创建目标文件。但是，为目标指定配方的规则不需要具有先决条件。例如，包含与目标 “clean” 关联的删除命令的规则没有先决条件。

然后，规则解释了如何以及何时重新制作特定文件，这个文件就是特定规则的目标。*make* 在先决条件上执行配方，来创建或更新目标。规则还可以解释如何以及何时执行操作。请参阅 [4 Writing Rules](https://www.gnu.org/software/make/manual/make.html#Rules)。

makefile 可能包含除规则之外的其他文本，但简单的 makefile 只需要包含规则。规则可能看起来比此模板中显示的要复杂一些，但都或多或少地符合模式。

## 2.2 一个简单的 Makefile

这是一个简单的 makefile，它描述了一个名为 edit 的可执行文件依赖于八个目标文件，而这些目标文件又依赖于八个C源文件和三个头文件。

在此示例中，所有 C 文件都包含 *defs.h*，但只有那些定义编辑命令的文件才包括 *command.h*，只有更改编辑器缓冲区的低级文件才包括 *buffer.h*。

示例：
```makefile
edit : main.o kbd.o command.o display.o \
       insert.o search.o files.o utils.o
        cc -o edit main.o kbd.o command.o display.o \
                   insert.o search.o files.o utils.o

main.o : main.c defs.h
        cc -c main.c
kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
insert.o : insert.c defs.h buffer.h
        cc -c insert.c
search.o : search.c defs.h buffer.h
        cc -c search.c
files.o : files.c defs.h buffer.h command.h
        cc -c files.c
utils.o : utils.c defs.h
        cc -c utils.c
clean :
        rm edit main.o kbd.o command.o display.o \
           insert.o search.o files.o utils.o
```

我们使用反斜杠/换行符将每个长行分成两行；这就像使用一条长线，但更容易阅读。参阅 [3.1.1 Splitting Long Lines](https://www.gnu.org/software/make/manual/make.html#Splitting-Lines)。

要使用此 makefile 创建名为 edit 的可执行文件，请键入(译者注，在终端里键入)：

```sh
make
```

要使用此 makefile 从目录中删除可执行文件和所有目标文件，请键入：

```sh
make clean
```

在示例 makefile 中，目标包括可执行文件 “edit”，以及目标文件 “main.o” 和 “kbd.o”。先决条件是诸如 “main.c” 和 “defs.h” 之类的文件。事实上，每个 “.o” 文件既是目标又是先决条件。配方包括 “`cc -c main.c`” 和 “`cc -c kbd.c`”。

当目标是文件时，如果其任何先决条件发生更改，则需要重新编译或重新链接它。此外，应首先更新本身自动生成的任何先决条件。在此示例中，edit 依赖于八个目标文件中的每一个; 目标文件 main.o 依赖于源文件 main.c 和头文件 defs.h。

配方会跟在包含目标和先决条件的每一行之后的行。这些配方说明了如何更新目标文件。制表符（或变量 `.RECIPEPREFIX` 指定的任何字符；请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)）必须出现在配方中每一行的开头，以将配方与 makefile 中的其他行区分开来。（请记住，make 对配方的工作原理一无所知。由您提供将正确更新目标文件的配方。make 所做的就是在需要更新目标文件时执行您指定的配方。）

目标 “clean” 不是一个文件，而仅仅是一个操作的名称。由于您通常不想执行此规则中的操作，因此 “clean” 不是任何其他规则的先决条件。因此，除非您明确告诉它，否则 make 永远不会对它做任何事情。请注意，此规则不仅不是先决条件，它也没有任何先决条件，因此该规则的唯一目的是运行指定的配方。不引用文件而只是操作的目标称为*伪目标*。有关此类目标的信息，请参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)。请参阅 [5.5 Errors in Recipes](https://www.gnu.org/software/make/manual/make.html#Errors)，了解如何使make 忽略来自 rm 或任何其他命令的错误。

## 2.3 make 执行 Makefle 的过程

默认情况下，make 从第一个 target 开始（不是名称以“.”开头的 target，除非它们还包含一个或多个 ‘/’）。这称为默认终点目标（_default goal_）。（终点目标是 make 努力最终更新的目标。您可以使用命令行（请参阅 [9.2 Arguments to Specify the Goals](https://www.gnu.org/software/make/manual/make.html#Goals)）或使用 `.DEFAULT_GOAL` 特殊变量（请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)）覆盖此行为。）

在上一节的简单示例中，默认终点目标是更新可执行程序 edit；因此，我们将该规则放在首位。

因此，当您发出如下的命令时：

```sh
make
```

make 读取当前目录中的 makefile 并从处理第一条规则开始。在示例中，此规则用于重新链接 edit；但是在 make 可以完全处理此规则之前，它必须处理 edit 所依赖的文件(在本例中是目标文件)的规则。这些目标文件中的每一个都根据自己的规则进行处理。这些规则说通过编译对应的源文件来更新每个“.o”文件的。如果源文件或任何作为先决条件命名的头文件比目标文件更近期，或者目标文件不存在，则必须进行重新编译。

处理其他规则是因为它们的目标作为终点目标的先决条件出现。如果其他规则不被终点目标（或重点目标所依赖的任何东西，等等）所依赖，则不处理该规则，除非您告诉 make 这样做（例如使用 `make clean` 等命令）。

在重新编译目标文件之前，make 会考虑更新其先决条件、源文件和头文件。示例中的 makefile 没有指定要为它们做的任何事情——“.c” 和 “.h” 文件不是任何规则的目标——所以 make 不会为这些文件做任何事情。但是 make 会根据它们自己的规则更新自动生成的 C 程序，比如 Bison 或 Yacc 制作的程序。

重新编译需要的目标文件后，make 决定是否重新链接 edit。如果文件 edit 不存在，或者任何目标文件比 edit 更新，则必须这样做。如果目标文件刚刚重新编译，它现在比 edit 更新，因此 edit 被重新链接。

因此，如果我们更改文件 insert.c 并运行 make，make 将编译该文件以更新 insert.o，然后链接 edit。如果我们更改文件 command.h 并运行 make，make 将重新编译目标文件 kbd.o、command.o 和 file.o，然后链接文件 edit。

## 2.4 使用变量简化 makefile

在我们的示例中，我们必须在 edit 规则中列出所有目标文件两次（此处重复）：

```makefile
edit : main.o kbd.o command.o display.o \
              insert.o search.o files.o utils.o
        cc -o edit main.o kbd.o command.o display.o \
                   insert.o search.o files.o utils.o
```

这种重复很容易出错；如果一个新的目标文件被添加到系统中，我们可能会将其添加到一个列表中而忘记另一个列表。我们可以通过使用变量来消除风险并简化 makefile。变量允许单次定义文本字符串，然后在多处替换。参阅 [6 How to Use Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables)。

每个 makefile 的标准做法是有一个名为 *objects*, *OBJECTS*, *objs*, *OBJS*, *obj* 或 *OBJ* 的变量，它是所有目标文件名的列表。我们将在 makefile 中用这样的行定义这样一个变量 *objects*：

```makefile
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o
```

然后，我们要放置目标文件名列表的每个地方，我们可以通过写入 `$(objects)` 来替换变量的值，参阅 [6 How to Use Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables)。

以下是当您为目标文件使用变量时完整的简单 makefile 的外观：

```makefile
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)
main.o : main.c defs.h
        cc -c main.c
kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
insert.o : insert.c defs.h buffer.h
        cc -c insert.c
search.o : search.c defs.h buffer.h
        cc -c search.c
files.o : files.c defs.h buffer.h command.h
        cc -c files.c
utils.o : utils.c defs.h
        cc -c utils.c
clean :
        rm edit $(objects)
```

## 2.5 让 make 推断 recipes

没有必要详细说明编译单个C源文件的方法，因为 `make` 可以弄清楚它们：它有一个隐含的规则，可以使用 `cc -c` 命令从相应命名的 .c 文件中更新 .o 文件。例如，它将使用配方 `cc -c main.c -o main.o` 将 main.c 编译为 main.o。因此，我们可以从目标文件的规则中省略这些方法。请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)。

当以这种方式自动使用 “.c” 文件时，它也会自动添加到先决条件列表中。因此，只要我们省略配方，我们就可以从先决条件中省略 “.c” 文件。

这是整个示例，其中包含这两个更改，以及上面建议的变量 *objects*：

```makefile
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
        rm edit $(objects)
```

这就是我们在实际编写 makefile 的方式。（与 “clean” 相关的复杂情况在 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets) 和 [5.5 Errors in Recipes](https://www.gnu.org/software/make/manual/make.html#Errors) 中描述。

因为隐式规则非常方便，所以它们很重要。你会看到它们经常被使用。

## 2.6 Makefile的另一种形式

当 makefile 的目标文件 **仅由隐式规则创建时**，可以使用 makefile 的另一种风格。在这种 makefile 风格中，可以按 prerequisites 而不是 target 对条目进行分组。

```makefile
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h
```

这里 *defs.h* 作为所有目标文件的先决条件给出；*command.h* 和 *buffer.h* 是它们列出的特定目标文件的先决条件。

这是否更好是一个品味问题：它更紧凑，但有些人不喜欢它，因为他们发现将每个目标的所有信息放在一个地方更清晰。

## 2.7 清理目录的规则

编译程序并不是您编写规则唯一想要做的事情。Makefile 通常会告诉您除了编译程序之外如何做其他一些事情：例如，如何删除所有目标文件和可执行文件，以便目录是“干净的”。

以下是我们如何编写一个 make 规则来清理我们的示例编辑器：

```makefile
clean:
        rm edit $(objects)
```

在实践中，我们可能希望以更复杂的方式编写规则来处理意料之外的情况。我们会这样做：

```makefile
.PHONY : clean
clean :
        -rm edit $(objects)
```

这可以防止 make 被称为 *clean* 的实际文件混淆，并导致它在 `rm` 出现错误的情况下继续运行。参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets) 和 [5.5 Errors in Recipes](https://www.gnu.org/software/make/manual/make.html#Errors)。

像这样的规则不应该放在 makefile 的开头，因为我们不希望它默认运行！因此，在示例 makefile 中，我们希望重新编译编辑器的 *edit* 规则保持为默认目标。

由于 *clean* 不是 *edit* 的先决条件，如果我们给出不带参数的命令 “`make`”，则此规则根本不会运行。为了使规则运行，我们必须键入 “`make clean`”。请参阅 [9 How to Run make](https://www.gnu.org/software/make/manual/make.html#Running)。

# 3 Writing Makefiles

来自数据库中告诉 *make* 如何重新编译系统的信息被称为 makefile。

## 3.1 Makefiles 包含的内容

- explicit rules
    显式规则说明何时以及如何重新制作一个或多个文件，称为规则的 _targets_。它列出了 _target_ 所依赖的其他文件，称为目标的 _prerequisites_ ，还可以提供用于创建或更新 _target_ 的 _recipe_ 。[4 Writing Rules](https://www.gnu.org/software/make/manual/make.html#Rules)
- implicit rules
    一个隐式规则说明何时以及如何根据文件的名称重新制作一类文件。它描述了目标如何依赖于名称与target相似的文件，并给出了创建或更新此类target的recipe。[10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules).
- variable definitions
- directives
    指令是让make在读取makefile时执行特殊操作的指令。这包括：
    * 读取另一个makefile。 [3.3 Including Other Makefiles](https://www.gnu.org/software/make/manual/make.html#Include)
    * 决定（基于变量的值）是使用还是忽略makefile的一部分。 [7 Conditional Parts of Makefiles](https://www.gnu.org/software/make/manual/make.html#Conditionals)
    * 从包含多行的逐字字符串中定义变量。 [6.8 Defining Multi-Line Variables](https://www.gnu.org/software/make/manual/make.html#Multi_002dLine)
- comments
    makefile的一行中的'#'开始注释。它和该行的其余部分被忽略，除了未被另一个反斜杠转义的尾随反斜杠将跨多行继续注释。
    recipe中的注释被传递给shell，就像任何其他recipe文本一样。shell决定如何解释它：这是否是注释取决于shell。
    在 `define` 指令中，注释在定义变量时不会被忽略，而是保持变量值不变。当变量展开时，它们将被视为make注释或recipe文本，具体取决于评估变量的上下文。
- [3.1.1 Splitting Long Lines](https://www.gnu.org/software/make/manual/make.html#Splitting-Lines)

### 3.1.1 Splitting Long Lines

Makefile使用“基于行”的语法，其中换行符是特殊的，并标记语句的结尾。GNU make 对语句行的长度没有限制，最高可达计算机中的内存量。

但是，如果不换行或滚动，很难读取太长而无法显示的行。因此，您可以通过在语句中间添加换行符来格式化 makefile 以提高易读性：您可以通过使用反斜杠（`\`）字符转义内部换行符来做到这一点。在需要区分的地方，我们将以换行符结尾的单行（无论它是否被转义）称为“物理行”，而将一个完整的语句，包括所有转义的换行符，直到第一个非转义的换行符，称为“逻辑行”。

处理反斜杠及换行符组合的方式取决于语句是 recipe 行还是非recipe行。处理 recipe 行中的反斜杠及换行符组合将在后面讨论，见[5.1.1 Splitting Recipe Lines](https://www.gnu.org/software/make/manual/make.html#Splitting-Recipe-Lines)

在recipe行之外，反斜杠及换行符组合将转换为单个空格字符。完成后，反斜杠及换行符组合**周围的所有空格都将压缩为单个空格**：这包括反斜杠之前的所有空格、“反斜杠-换行符”之后行首的所有空格以及任何连续的“反斜杠-换行符”组合。

如果特殊目标 `.POSIX` 被定义，那么反斜杠及换行符组合处理会稍微修改以符合 POSIX.2：首先，不删除反斜杠前面的空格，其次，不压缩连续的反斜杠/换行符。

#### 不添加空格的拆分

如果您需要拆分一行但不希望添加任何空格，您可以利用一个微妙的技巧：使用美元符号、反斜杠和换行符三个字符：

```makefile
var := one$\
       word
```

make删除反斜杠及换行符并将以下行压缩为一行后，这相当于：

```makefile
var := one$ word
```

然后 make 将执行变量展开。变量引用'`$ `'指的是一个不存在的单字符名称“ ”（空格）的变量，因此展开为空字符串，给出一个最终赋值，相当于：

```makefile
var := oneword
```

## 3.2 Makefile文件的名称

默认情况下，当 make 查找 makefile 时，它会按顺序尝试以下名称：*GNUmakefile*、*makefile* 和 *Makefile*。

通常，您应该将 makefile 称为 *makefile* 或 *Makefile*。（我们推荐 *Makefile*，因为它出现在目录列表的开头附近，就在 *README* 等其他重要文件附近。）检查的第一个名称 *GNUmakefile* 对大多数 makefile 都不推荐。如果您有一个特定于 GNU make 的 makefile，并且不会被其他版本的 make 理解，您应该使用这个名称。其他 make 程序查找 *makefile* 和 *Makefile*，但不查找 *GNUmakefile*。

如果 make 没有找到这些名称，则它不使用任何 makefile。然后您必须使用命令参数指定一个目标，make 将尝试弄清楚如何仅使用其内置的隐式规则重新制作它。请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)。

如果您想为 makefile 使用非标准名称，您可以使用 “`-f`” 或 “`--file`” 选项指定 makefile 名称。参数 “`-f name`” 或 “`--file=name`” 告诉 make 将文件 *name* 作为 makefile 读取。如果您使用多个 “`-f`”或“`--file`” 选项，您可以指定多个 makefile。所有 makefile 都按照指定的顺序有效地连接在一起。如果您指定“`-f`”或“`--file`”，则不会自动检查默认 makefile 名称*GNUmakefile*、*makefile* 和 *Makefile*。

## 3.3 包含其它 Makefile

`include` 指令告诉 make 暂停读取当前 makefile，并在继续之前读取一个或多个其他 makefile。该指令是 makefile 中的一行，如下所示：

```makefile
include filenames…
```

`filenames` 可以包含 shell 文件名模式。如果 `filenames` 为空，则不包含任何内容，也不会打印错误。

行首可以出现额外的空格，但会被忽略，但第一个字符不能是制表符（或 `.RECIPEPREFIX` 的值）——如果该行以制表符开头，它将被视为配方行。`include ` 和文件名之间、以及文件名之间需要空格；在那里和指令末尾忽略额外的空格。以 “`#`”开头的注释允许出现在行尾。如果文件名包含任何变量或函数引用，它们将被展开。参阅 [6 How to Use Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables)

例如，如果您有三个 *.mk* 文件，*a.mk*、*b.mk* 和 *c.mk*，并且 `$(bar)` 扩展为 *bish bash*，则以下表达式

```makefile
include foo *.mk $(bar)
```

等效于

```makefile
include foo a.mk b.mk c.mk bish bash
```

当 makefile 处理一个 `include` 指令时，它会暂停读取 容器makefile，并依次从每个列出的文件中读取。当完成后，make 恢复读取指令出现的 makefile。

使用 `include` 指令的一种情况是，由多个目录中的单独 makefile 处理的多个程序需要使用一组通用的变量定义 (请参阅 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting)) 或模式规则 (请参阅 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules))。

另一种情况是当您想从源文件自动生成先决条件时；先决条件可以放在 主makefile 包含的文件中。这种做法通常比传统上对其他版本的 make 进行的以某种方式将先决条件附加到 主makefile 末尾的做法更清晰。请参阅 [4.14 Generating Prerequisites Automatically](https://www.gnu.org/software/make/manual/make.html#Automatic-Prerequisites)。

如果指定的名称不以斜杠开头（或者在使用 MS-DOS/MS-Windows 路径支持时编译的 GNU Make下，以驱动器号和冒号开头），并且在当前目录中找不到该文件，则搜索其他几个目录。首先，搜索您使用 “`-I`” 或 “`include-dir`” 选项指定的任何目录（请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/make.html#Options-Summary)）。然后按以下顺序搜索以下目录（如果存在）：`prefix/include`(通常是 `/usr/local/include`) `/usr/gnu/include`, `/usr/local/include`, `/usr/include`. 

`.INCLUDE_DIRS` 将包含 make 将搜索包含文件的当前目录列表。参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)

您可以通过在命令行中添加带有特殊值 `-` 的命令行选项 `-I`（例如 `-I-`）来避免在这些默认目录中进行搜索。这将导致 make 忘记任何已设置的包含目录，包括默认目录。

如果在这些目录中找不到被包含的 makefile，并不会产生立即致命的错误；包含 `include` 的 makefile 的处理仍在继续。一旦读取完 makefile，make 将尝试重制任何过期或不存在的 makefile。请参阅 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/make.html#Remaking-Makefiles)。只有在找不到重制 makefile 的规则或找到规则但配方失败后，make 才会将丢失的 makefile 诊断为致命错误。

如果您想让 make 简单地忽略不存在或无法重新制作的 makefile，并且没有错误消息，请使用 `-include` 指令而不是 `include` ，如下所示：

```makefile
-include filenames…
```

这在各方面都类似于 `include`，除了在任何 *filenames*（或任何文件名的任何先决条件）不存在或无法重新制作的情况下，没有错误（甚至没有警告）。

为了与其他一些 make 实现兼容，`sinclude` 是 `-include` 的另一个名称。

## 3.4 变量 `MAKEFILES`

如果定义了环境变量 `MAKEFILES`，make 会将其值视为要在其他文件之前读取的额外 makefile 的名称列表（以空格分隔）。这与include指令的工作原理非常相似：为找到这些文件将搜索许多不同的目录（请参阅 [3.3 Including Other Makefiles](https://www.gnu.org/software/make/manual/make.html#Include)）。此外，默认终点目标永远不会从这些 makefile 之一（或它们包含的任何 makefile）中获取，如果找不到 `MAKEFILES` 中列出的文件，并不会出错。

`MAKEFILES` 的主要用途是在 make 的递归调用之间进行通信（请参阅 [5.7 Recursive Use of make](https://www.gnu.org/software/make/manual/make.html#Recursion)）。在顶层调用 make 之前设置环境变量通常是不可取的，因为通常最好不要从外部弄乱 makefile。但是，如果您在没有特定 makefile 的情况下运行 make，`MAKEFILES` 中的 makefile 可以做一些有用的事情来帮助内置的隐式规则更好地工作，例如定义搜索路径（请参阅 [4.5 Searching Directories for Prerequisites](https://www.gnu.org/software/make/manual/make.html#Directory-Search)）。

一些用户倾向于在登录时自动在环境中设置 `MAKEFILES`，并对makefile进行编程以期望这样做。这是一个非常糟糕的主意，因为如果由其他人运行，这样的 makefile 将无法工作。在 makefile 中编写显式 `include` 指令要好得多。请参阅 [3.3 Including Other Makefiles](https://www.gnu.org/software/make/manual/make.html#Include)。

## 3.5 Makefiles 是怎样被读取的

有时 makefile 可以从其他文件重制，例如 RCS 或 SCCS 文件。如果可以从其他文件重制 makefile，您可能希望 make 获取最新版本的 makefile 以读取。

为此，在读入所有 makefile 后，make 将按照处理它们的顺序将每个文件视为终点目标，并尝试更新它。如果启用了并行构建（参见 [5.4 Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel)），则 makefile 也将并行重建。

如果 makefile 有一个规则说明如何更新它（在该 makefile 或另一个 makefile 中找到），或者如果隐式规则适用于它（请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)），则会在必要时更新它。检查完所有 makefile 后，如果确实有更改，make 会清空记录，然后再次读取所有 makefile。（它还会尝试再次更新每个 makefile，但通常不会再次更改它们，因为它们已经是最新的。）每次重新启动都会导致特殊变量 `MAKE_RESTARTS` 被更新（请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)）。

如果您知道一个或多个 makefile 不能重做，也许是出于效率原因，您想阻止 make 对它们执行隐式规则搜索，您可以使用任何防止隐式规则查找的正常方法来执行此操作。例如，您可以编写一个以该 makefile 为目标的显式规则和一个空配方（请参阅 [5.9 Using Empty Recipes](https://www.gnu.org/software/make/manual/make.html#Empty-Recipes)）。

如果 makefile 指定了一个双冒号规则来重做一个有配方但没先决条件的文件，该文件将始终被重做（参见 [4.13 Double-Colon Rules](https://www.gnu.org/software/make/manual/make.html#Double_002dColon)）。对于 makefile，一个有配方但没有先决条件的双冒号规则的 makefile 将在每次 make 运行时被重做，然后在 make 重新启动后再次读取makefile。这将导致一个无限循环：make 将不断地重做 makefile 并重新启动，而不再做任何其他事情。因此，**为了避免这种情况，make 不会尝试重做被指定为有配方但没先决条件的双冒号规则目标的 makefile。**

伪目标（请参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)）具有相同的效果：它们永远不会被视为最新的，因此标记为伪的被包含文件会导致 make 不断重新启动。**为了避免这种情况，make 不会尝试重新制作标记为伪的 makefile**。

您可以利用这一点来优化启动时间：如果您知道您的 Makefile 不需要重新制作，您可以通过添加以下内容来防止 make 尝试重新制作它：

```makefile
.PHONY: Makefile
```

或：

```makefile
Makefile:: ;
```

如果您没有使用 “`-f`” 或 “`--file`” 选项指定要读取的任何 makefile，make 将尝试默认 makefile 名称；请参阅 [3.2 What Name to Give Your Makefile](https://www.gnu.org/software/make/manual/make.html#Makefile-Names)。与使用 “`-f`” 或 “`--file`” 选项显式请求的 makefile 不同，make 不确定这些 makefile 是否应该存在。但是，如果默认 makefile 不存在，但可以通过运行 make 规则来创建，您可能希望运行规则以便可以使用 makefile。

因此，如果默认 makefile 都不存在，make 将尝试创建它们中的每一个，直到它成功创建一个，或者它用完了可以尝试的名称。请注意，如果 make 找不到或创建任何 makefile，这不是错误；makefile 并不总是必要的。

当您使用 “`-t`” 或 “`--touch`” 选项（请参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/make.html#Instead-of-Execution)）时，您不希望使用过时的 makefile 来决定要 touch 哪些目标。因此，**“`-t`” 选项对更新 makefile 没有影响**；即使指定了 “`-t`”，它们也会真正更新。同样，“`-q`”（或“`--question`”）和 “`-n`”（或“`--just-print`”）不会阻止 makefile 的更新，因为过时的 makefile 会导致其他目标的错误输出。因此，“`make -f mfile -n foo`” 将更新 *mfile*，读取它，然后打印配方以更新 foo 及其先决条件，而不运行它。为 foo 打印的配方将是 *mfile* 更新内容中指定的配方。

但是，**有时您可能实际上希望阻止更新 makefile。您可以通过在命令行中将此 makefile 指定为终点目标以及将它们指定为 makefile 来执行此操作**。当 makefile 名称被明确指定为终点目标时，选项 '`-t`' 等确实适用于它们。

因此，“`make -f mfile -n mfile foo`” 将读取makefile *mfile*，打印更新它所需的配方，而不实际运行它，然后打印更新 foo 所需的配方，而不运行它。foo 的配方将是由 *mfile* 的现有内容指定的配方。

## 3.6 覆盖另一个 Makefile 中的一部分

有时拥有一个与另一个 makefile 基本相同的 makefile 很有用。您通常可以使用 '`include`' 指令将一个 makefile 包含在另一个中，并添加更多目标或变量定义。但是，两个 makefile 为同一目标提供不同的配方是无效的。但是还有另一种方法。

在 容器makefile（想要包含另一个 makefile 的那个）中，您可以使用 *match-anything* 模式规则（[10.5.5 Match-Anything Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Match_002dAnything-Rules)）来说明，要根据从 容器makefile 获取的信息无法重制的任何目标，make应该查看另一个makefile。有关模式规则的更多信息，请参阅 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules)。

例如，如果您有一个名为 *Makefile* 的 makefile，它说明如何制作目标 “foo”（和其他目标），您可以编写一个名为 *GNUmakefile* 的 makefile，其中包含：

```makefile
foo:
    frobnicate > foo

%: force
    @$(MAKE) -f Makefile $@
force: ;
```

如果你键入 `make foo`，make 会找到 *GNUmakefile*，阅读它，看到要制作 foo，它需要运行配方 '`frobnicate > foo`'。如果你键入 `make bar`，make 将无法在 *GNUmakefile* 中制作 bar，因此它将使用模式规则中的配方：'`make -f Makefile bar`'。如果 *Makefile* 提供了更新 bar 的规则，make 将应用该规则。对于 *GNUmakefile* 没有说明如何制作的任何其他目标也是同样如此。

(译者注，在这里不用深究 `@` 字符的意义，具体解释在 [5.2 Recipe Echoing](https://www.gnu.org/software/make/manual/make.html#Echoing)
`$(MAKE)` 的解释在 [5.7.1 How the MAKE Variable Works](https://www.gnu.org/software/make/manual/make.html#MAKE-Variable))

它的工作方式是因为模式规则是只有一个 “`%`”，它匹配任何目标。该规则指定了一个先决条件 *force*，以保证即使目标文件已经存在，配方也会运行。我们给目标 *force* 一个空配方，以防止 make 搜索隐式规则来构建它——否则它将对 *force* 本身应用相同的 *match-anything* 规则，并创建一个先决条件循环！


## 3.7 `make` 读取 Makefile 的方式

GNU make的工作分为两个不同的阶段。在第一阶段，它读取所有 makefile 和被包含的 makefile，并内化所有变量及其值、隐式和显式规则，并构建所有目标及其先决条件的依赖关系图。在第二阶段，make 使用这些内化数据来确定哪些 targets 需要更新，并运行更新它们所需的 recipe。

理解这种两阶段方法很重要，因为它直接影响变量和函数展开的方式；这通常是编写 makefile 时一些混乱的根源。下面是 makefile 中不同结构的摘要，以及结构的每个部分发生展开的阶段。

如果展开发生在第一阶段，我们说它是立即的：make 将在分析makefile 时展开构造的那一部分。如果展开不是立即的，我们说展开是延迟的。延迟构造部分的展开被延迟到展开被使用时：当它在立即上下文中被引用时，或者当它在第二阶段被需要时。

您可能还不熟悉其中的一些结构。您可以在以后的章节中熟悉本节时参考它们。

### 变量赋值

变量定义被解析如下：

```makefile
immediate = deferred
immediate ?= deferred
immediate := immediate
immediate ::= immediate
immediate :::= immediate-with-escape
immediate += deferred or immediate
immediate != immediate

define immediate
  deferred
endef

define immediate =
  deferred
endef

define immediate ?=
  deferred
endef

define immediate :=
  immediate
endef

define immediate ::=
  immediate
endef

define immediate :::=
  immediate-with-escape
endef

define immediate +=
  deferred or immediate
endef

define immediate !=
  immediate
endef
```

对于追加运算符 ‘`+=`’, 如果变量先前被设置为简单变量 (‘`:=`’ 或 ‘`::=`’), 则右侧被认为是立即的，否则被认为是延迟的。

对于 *immediate-with-escape* 运算符 ‘`:::=`’, 右侧的值立即展开，但随后转义（即展开结果中的所有 `$` 实例都替换为 `$$`）。

对于 shell 赋值运算符 ‘`!=`’, 右侧被立即计算并交给 shell。结果存储在左侧命名的变量中，该变量被视为递归扩展变量（因此将在每个引用上重新计算）。

### 条件指令

条件指令会立即被解析。例如，这意味着自动变量不能在条件指令中使用，因为在调用该规则的配方之前不会设置自动变量。如果您需要在条件指令中使用自动变量，您必须将条件移动到配方中并改用 shell 条件语法。

### 规则定义

无论形式如何，规则总是以相同的方式展开：

```makefile
immediate : immediate ; deferred
        deferred
```

即目标和先决条件部分立即展开，用于构建目标的配方总是延迟展开。显式规则、模式规则、后缀规则、静态模式规则和简单先决条件定义都是如此。

## 3.8 解析 Makefile 的方式

GNU逐行解析 makefile。解析使用以下步骤进行：

1. 读取完整的逻辑行，包括反斜杠转义行。（参阅 [3.1.1 Splitting Long Lines](https://www.gnu.org/software/make/manual/make.html#Splitting-Lines)）。
2. 移除注释（参阅 [3.1 What Makefiles Contain](https://www.gnu.org/software/make/manual/make.html#Makefile-Contents)）。
3. 如果该行以配方前缀字符开头，并且我们处于规则上下文中，则将该行添加到当前配方并读取下一行（参阅 [5.1 Recipe Syntax](https://www.gnu.org/software/make/manual/make.html#Recipe-Syntax)）。
4. 展开出现在*立即展开*上下文中的行元素（参阅 [3.7 How make Reads a Makefile](https://www.gnu.org/software/make/manual/make.html#Reading-Makefiles)）。
5. 扫描该行以查找分隔符，例如 '`:`' 或 '`=`'，以确定该行是宏赋值还是规则（参阅 [5.1 Recipe Syntax](https://www.gnu.org/software/make/manual/make.html#Recipe-Syntax)）。
6. 内化产生的操作并读取下一行。

这样做的一个重要结果是，如果宏只有一行长，它可以展开为整个规则。这将生效：

```makefile
myrule = target : ; echo built

$(myrule)
```

但是，这将不起作用，因为 make 在展开后不会重新拆分行：

```makefile
define myrule
target:
        echo built
endef

$(myrule)
```

上面的 makefile 导致定义了一个带有先决条件 'echo' 和 'build' 的目标 'target'，就好像 makefile 包含 `target: echo built`，而不是一个带有配方的规则。在展开完成后仍然存在于一行中的换行符像正常的空格一样被忽略。

为了正确展开多行宏，您必须使用 `eval` 函数：这会导致 make 解析器在展开宏的结果上运行（参阅 [8.10 The eval Function](https://www.gnu.org/software/make/manual/make.html#Eval-Function)）

## 3.9 第二次展开

之前我们了解到 GNU make 分两个不同的阶段工作：读入阶段和目标更新阶段（请参阅 [3.7 How make Reads a Makefile](https://www.gnu.org/software/make/manual/make.html#Reading-Makefiles)）。GNU make 还能够为 makefile 中定义的部分或全部目标启用先决条件（仅）的第二次展开。为了进行第二次展开，必须在使用此功能的第一个先决条件列表之前定义 `.SECONDEXPANSION`。

如果 `.SECONDEXPANSION` 被定义，那么当 GNU make 需要检查目标的先决条件时，先决条件会被第二次展开。在大多数情况下，这种二次展开不会有任何效果，因为所有变量和函数引用都将在 makefile 的初始解析期间被展开。为了利用解析器的二次展开阶段，那么，有必要转义 makefile 中的变量或函数引用。在这种情况下，第一次展开只是取消转义引用，但不会展开它，展开留给二次展开阶段。例如，考虑这个 makefile：

```makefile
.SECONDEXPANSION:
ONEVAR = onefile
TWOVAR = twofile
myfile: $(ONEVAR) $$(TWOVAR)
```

在第一个展开阶段之后，目标 `myfile` 的先决条件列表将是 *onefile* 和 `$(TWOVAR)`；第一个变量引用 `ONEVAR`（未转义的）被展开，而第二个变量引用（转义的）仅仅是取消转义，不被识别为变量引用。现在在二次展开期间，第一个单词再次展开，但由于它不包含变量或函数引用，它的值仍然是 *onefile*，而第二个单词现在是对变量 `TWOVAR` 的正常引用，它被展开为值 *twofile*。最终结果是有两个先决条件，*onefile* 和 *twofile*。

显然，这不是一个非常有趣的情况，因为简单地让两个变量都出现在先决条件列表中，而不是转义，就可以更容易地获得相同的结果。如果变量被重置，一个区别就很明显了；考虑这个例子：

```makefile
.SECONDEXPANSION:
AVAR = top
onefile: $(AVAR)
twofile: $$(AVAR)
AVAR = bottom
```

在这里，*onefile* 的先决条件将立即展开，并解析为值 *top*，而 *twofile* 的先决条件将不会完全展开，直到二次展开并产生值 *bottom*。

这稍微更令人兴奋一些，但是只有当您发现二次展开总是发生在该目标的自动变量范围内时，此功能的真正力量才会变得明显。这意味着您可以在第二次展开期间使用变量，如 `$@`, `$*`, 等，它们将有其期望值，就像在配方中一样。您所要做的就是通过转义 `$` 来推迟展开。此外，显式和隐式（模式）规则都会发生二次展开。了解了这一点，此功能的可能用途急剧增加。例如：

```makefile
.SECONDEXPANSION:
main_OBJS := main.o try.o test.o
lib_OBJS := lib.o api.o

main lib: $$($$@_OBJS)
```

在这里，在初始扩展之后，目标 *main* 和 *lib* 的先决条件都是 `$($@_OBJS)`。在二次扩展期间，变量 `$@` 被设置为目标的名称，因此目标 *main* 的展开将产生 `$(main_OBJS)` 或 `main.o try.o test.o`，而目标 *lib* 的二次展开将产生 `$(lib_OBJS)` 或 `lib.o api.o`。

您也可以在此处混合函数，只要它们被正确转义：

```makefile
main_SRCS := main.c try.c test.c
lib_SRCS := lib.c api.c

.SECONDEXPANSION:
main lib: $$(patsubst %.c,%.o,$$($$@_SRCS))
```

此版本允许用户指定源文件而不是目标文件，但生成的先决条件列表前面示例相同。

在二次展开阶段中对自动变量的评估，尤其是对目标名称变量 `$$@` 的评估，与在配方中的评估类似。然而，对于不同类型的规则定义来说，有一些细微的区别和“角落情况”。下面描述了使用不同自动变量的微妙之处。

### 显式规则的二次展开

在显式规则的二次展开期间，`$$@` 和 `$$%` 分别被计算为目标的文件名、目标成员名（当目标是存档成员时）。`$$<` 变量被计算到此目标的第一个规则中的第一个先决条件。`$$^` 和 `$$+` 计算到已为同一目标出现的所有先决条件规则的列表（`$$+` 有重复，`$$^` 没有）。以下示例将有助于说明这些行为：

```makefile
.SECONDEXPANSION:

foo: foo.1 bar.1 $$< $$^ $$+    # line #1

foo: foo.2 bar.2 $$< $$^ $$+    # line #2

foo: foo.3 bar.3 $$< $$^ $$+    # line #3
```

在第一个先决条件列表中，所有三个变量 (`$$<`, `$$^`, 和`$$+`）都展开为空字符串。在第二个中，它们的值分别为 `foo.1`、`foo.1 bar.1`和 `foo.1 bar.1`。在第三个中，它们的值分别为`foo.1`、`foo.1 bar.1 foo.2 bar.2` 和 `foo.1 bar.1 foo.2 bar.2 foo.1 bar.1 foo.1 bar.1 bar.1`。

规则会按照 makefile 顺序进行二次展开，除了带有配方的规则总是最后评估。

变量 `$$?` 和 `$$*` 不可用并展开为空字符串。

### 静态模式规则的二次展开

静态模式规则的二次展开规则与上面的显式规则相同，但有一个例外：对于静态模式规则，变量 `$$*` 被设置为模式*词干*(stem)。与显式规则一样，`$$?` 不可用并展开为空字符串。

### 隐式规则的二次展开

当 make 搜索隐式规则时，它会替换词干，然后对具有匹配目标模式的每个规则执行二次展开。自动变量的值以与静态模式规则相同的方式派生。例如：

```makefile
.SECONDEXPANSION:

foo: bar

foo foz: fo%: bo%

%oo: $$< $$^ $$+ $$*
```

当对目标 foo 尝试隐式规则时，`$$<` 展开为 `bar`，`$$^` 展开为 `bar boo`，`$$+` 也展开为 `bar boo`，`$$*` 展开为 `f`。

请注意，如 [10.8 Implicit Rule Search Algorithm](https://www.gnu.org/software/make/manual/make.html#Implicit-Rule-Search) 中所述，目录前缀(D) 被附加（扩展后）到先决条件列表中的所有模式。作为一个例子：

```makefile
.SECONDEXPANSION:

/tmp/foo.o:

%.o: $$(addsuffix /%.c,foo bar) foo.h
        @echo $^
```

在二次展开和目录前缀重建之后，打印的先决条件列表会是 `/tmp/foo/foo.c /tmp/bar/foo.c foo.h`。如果您对此重建不感兴趣，您可以在先决条件列表中使用 `$$*` 而不是 `%`。

# 4 Writing Rules
规则(*rule*)会出现在makefile中，并说明何时以及如何重新创建某些文件（即该规则的目标(*target*)，通常每个规则只有一个目标）。它会列出作为目标的先决条件(*prerequisites*)的其他文件，以及用于创建或更新目标的指令(*recipe*)。

rule 的顺序的唯一作用，是指定默认终点目标（*default goal*，如果您没有另行指定目标，make要运行的目标）（译者注，这里的原文是"default goal: the target for make to consider, if you do not otherwise specify one"，是本人将 consider 翻译成 运行）。第一个 makefile 中第一个 rule 的第一个 target 是默认目标。有两个例外：以句点开头的目标不是默认目标，除非它还包含一个或多个斜杠, ‘/’; 定义模式规则的目标对默认目标没有影响。（请参阅 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules)。）

因此，我们编写makefile时，通常会将第一条 rule 作为是编译整个程序或 makefile 描述的所有程序的规则（通常将其称为“all”）。请参阅 [9.2 Arguments to Specify the Goals](https://www.gnu.org/software/make/manual/make.html#Goals)。
## 4.1 规则示例

``` Makefile
foo.o : foo.c defs.h       # module for twiddling the frobs
	cc -c -g foo.c
```
目标是foo. o，先决条件是 foo.c 和 defs.h。它在指令中有一个执行命令：“cc -c -g foo.c”。该指令以一个缩进符开头，以将其标识为指令。

这条规则说两件事：
- 如何判定 foo. o 是否过期：如果它不存在，或者 foo.c 或 defs.h 比它更新，它就过期了。
- 如何更新 foo. o 文件：按规定运行 cc 。该指令没有明确提到 defs.h，但我们假设 foo.c 包含它，这就是将 defs.h 添加到先决条件中的原因。
## 4.2 Rule的语法

``` Makefile
targets : prerequisites
        recipe
        …
```
或者
``` Makefile
targets : prerequisites ; recipe
	recipe
    …
```

目标(*target*)是以空格分隔的文件名。可以使用通配符（见 [4.4 Using Wildcard Characters in File Names](https://www.gnu.org/software/make/manual/make.html#Wildcards)），名称的形式为`a(m)`的代表存档文件 a 中的成员 m（见 [11.1 Archive Members as Targets](https://www.gnu.org/software/make/manual/make.html#Archive-Members)）。通常每个 rule 只有一个 target，但偶尔也有更多 targets（见 [4.10 Multiple Targets in a Rule](https://www.gnu.org/software/make/manual/make.html#Multiple-Targets)）。

指令(*recipe*)行以制表符（或 `.RECIPEPREFIX` 变量值中的第一个字符；见 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)）开头。第一个指令行可能会以带有制表符的形式出现在先决条件之后的行中，也可能以带有分号的形式出现在同一行中。无论哪种方式，效果都是相同的。指令的语法还有其他差异。请参阅  [5 Writing Recipes in Rules](https://www.gnu.org/software/make/manual/make.html#Recipes)。

因为美元符号(`$`)用于开始创建变量引用，所以如果您真的想在 target 或 prerequisite 中使用美元符号，您必须编写两个, 即`$$` (见 [6 How to Use Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables)）。如果您启用了二次展开（请参阅 [3.9 Secondary Expansion](https://www.gnu.org/software/make/manual/make.html#Secondary-Expansion)）并且您想在 prerequisite列表 中使用字面上的美元符号，您必须编写四个美元符号 (`$$$$`).

您可以“反斜杠+换行符"来拆分长行，但这不是必需的，因为 make 对 makefile 中的行长度没有限制。

规则告诉 make 两件事：目标何时过期，以及如何在必要时更新它们。

过时的标准是根据先决条件中的项来指定的，这些先决条件由用空格分隔的文件名组成（其中，通配符或存档成员（见 [11 Using make to Update Archive Files](https://www.gnu.org/software/make/manual/make.html#Archives)）是允许出现的）。如果目标不存在，或者如果它比先决条件中任一项旧（通过比较最后修改时间），则目标文件已过时。其思想是目标文件的内容是根据先决条件中的信息计算而来，因此如果任何先决条件发生变化，现有目标文件的内容就不一定有效。

如何更新由 recipe 指定。这是要由 shell 执行的一行或多行命令（通常是“sh”），但有一些额外的特征（请参阅  [5 Writing Recipes in Rules](https://www.gnu.org/software/make/manual/make.html#Recipes)）。

## 4.3 Prerequisite 的类型

GNU make 可以理解两种不同类型的先决条件：普通先决条件(normal prerequisites)和仅声明顺序的先决条件(order-only prerequisites)。一个 normal prerequisites 做出两种声明：首先，它强制规定了调用 recipe 的顺序：在执行目标的指令之前，将先完成目标的所有先决条件的指令。其次，它强加了一种依赖关系：如果任何先决条件比目标更新，则该目标被视为过时，必须重新构建（通常，这正是您想要的：如果更新了目标的先决条件，那么也应该更新目标。）。

有时，您可能希望确保先决条件在目标之前生成，但如果先决条件已更新，则不必强制更新目标。仅声明顺序的先决条件(order-only prerequisites)用于创建这种类型的关系。可以通过在先决条件列表中放置管道符号（|）来指定仅限顺序的先决条件：管道符号左侧的任何先决条件都是正常的；右侧的先决条件用于仅声明顺序：

``` Makefile
targets : normal-prerequisites | order-only-prerequisites
```

普通先决条件部分可以是空的。此外，您仍然可以为同一目标声明多行先决条件：它们会被适当地附加（普通先决条件被附加到普通先决条件的列表中；仅声明顺序的先决条件则被附加到仅声明顺序的先决条件的列表中）。请注意，如果您将同一文件声明为 normal 和 order-only ，则 normal 优先。

在确定 target 是否过期时，从不检查 order-only prerequisites；即使 order-only prerequisites 被标记为假(phony)（请参阅 [4.6 phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)）也不会导致重建目标。

假设，您的目标将被放置在一个单独的目录中，而在运行make之前，该目录可能不存在。在这种情况下，您希望在将任何目标放入目录之前创建目录，但由于每当添加、删除或重命名文件时，目录上的时间戳都会发生变化，因此我们当然不希望在目录的时间戳发生变化时重新生成所有目标。管理此问题的一种方法是使用仅声明顺序的先决条件：使目录成为所有目标的仅声明顺序的先决条件：

``` Makefile
OBJDIR := objdir
OBJS := $(addprefix $(OBJDIR)/,foo.o bar.o baz.o)

$(OBJDIR)/%.o : %.c
        $(COMPILE.c) $(OUTPUT_OPTION) $<

all: $(OBJS)

$(OBJS): | $(OBJDIR)

$(OBJDIR):
        mkdir $(OBJDIR)
```

（译者注，如果你看过 ST 工程师写的 makefile，你会发现他们使用过 order-only-prerequisites ）

现在，将在生成任何 “.o” 之前运行创建 objdir 目录的规则，但不会因为objdir的目录时间戳的更改导致重新生成任何 “.o” ，。


## 4.4 在文件名中使用通配符

可以使用通配符以单个文件名指定多个文件。make中的通配符为`*`、`？`和 `[…]`，和 Bourne shell 中的一样。例如，`*.c` 指定名称以“.c”结尾的所有文件（在工作目录中）的列表。

如果一个表达式与多个文件匹配，则将对结果进行排序。2但是，不会对多个表达式进行全局排序。例如，\*.c \*.h 将列出排序后的所有名称以“.c”结尾的文件，然后是排序后的所有名称以”.h“结尾的文件。

文件名开头的字符 `~` 也具有特殊意义。如果是单独的，或者后面跟一个斜线，它表示您的主目录。例如，`~/bin` 展开为 `/home/you/bin`。如果 `~` 后面跟着一个单词，则该字符串表示用该单词命名的用户的主目录。例如 `~john/bin` 展开为 `/home/john/bin` 。在没有每个用户主目录的系统（如MS-DOS或MS Windows）上，可以通过设置环境变量 HOME 来模拟此功能。

通配符展开由 make 在目标和先决条件中自动执行。在 recipe 中，shell 负责通配符展开。在其他情况下，只有当您使用 *wildcard* 函数明确请求通配符展开时，才会进行通配符展开。

通配符的特殊意义可以通过在其前面加一个反斜杠来关闭。因此，`foo\*bar`将引用一个特定的文件，该文件的名称由“foo”、星号和“bar”组成。

### 4.4.1 通配符示例

### 4.4.2 使用通配符的缺陷

### 4.4.3 `wildcard` 函数

通配符展开在规则(rules)中自动发生，但在**设置变量时或在函数的参数中**通常不会发生。此时则需要使用 `wildcard` 函数。

```
$(wildcard pattern…)
```

此字符串在makefile中的任何位置使用，将替换为与给定 *pattern* 之一匹配的现有文件名的以空格分隔的列表。如果没有现有文件名与 *pattern* 匹配，则 `wildcard` 函数的输出中省略该 *pattern*。请注意，这与在规则中不匹配的通配符的行为不同，在规则中，它们被逐字使用而不是忽略（请参阅使用通配符的陷阱）。

与规则中的通配符展开一样，`wildcard` 函数的结果是排序的。但是，每个单独的表达式都是单独排序的，因此 `$(wildcard *.c *.h)` 将先排序所有匹配 '.c' 的文件，然后是所有匹配 '.h' 的文件。

## 4.5 Searching Directories for Prerequisites

对于大型系统，通常需要将源代码放在与二进制文件分开的目录中。_make_ 的目录搜索功能通过自动搜索几个目录来找到一个先决条件，从而方便了这一点。在目录之间重新分发文件时，不需要更改单个规则，只需要更改搜索路径。

### 4.5.1 `VPATH`: 所有先决条件的搜索路径

_make_ 变量 `VPATH` 的值指定 _make_ 应搜索的目录列表。大多数情况下，目录应包含不在当前目录中的先决条件文件；然而，_make_ 使用 `VPATH` 作为规则的先决条件和目标的搜索列表。

因此，如果当前目录中不存在列为目标或先决条件的文件，_make_ 会在 `VPATH` 中列出的目录中搜索具有该名称的文件。如果在目录列表中找到一个文件，则该文件可能会成为先决条件（见下文）。然后，规则可以指定先决条件列表中文件的名称，就好像它们都存在于当前目录中一样。请参阅[ 4.5.4 Writing Recipes with Directory Search](https://www.gnu.org/software/make/manual/html_node/Recipes_002fSearch.html)。

在 `VPATH` 变量中，目录名用 **冒号** 或 **空格** 分隔。(`VPATH` 中)目录的列出顺序是 _make_ 在搜索中遵循的顺序。（在MS-DOS和MS Windows上，分号在 `VPATH` 中用作目录名的分隔符，因为冒号可以在路径名本身的驱动器号后面使用。）

示例：
```
VPATH = src:../headers
```
指定一个包含 src 和../headers 两个目录的路径，_make_ 按该顺序进行搜索。

利用VPATH的这个值，以下规则:

```
foo.o : foo.c
```
被解释为它是这样写的（假设文件 foo.c 不存在于当前目录中，而是位于 src 目录中。）：
```
foo.o : src/foo.c
```

### 4.5.2 指令 `vpath`

小写的指令 `vpath` 与 变量`VPATH` 相似，但前者更具有选择性，允许您为特定类别的文件名（那些与特定模式匹配的）指定搜索路径。因此，您可以为一类文件名提供某些搜索目录，而为其他文件名提供其它目录（或不提供）。

`vpath` 指令有三种形式：
- `vpath pattern directories`
    指定匹配 pattern 的文件名的搜索路径目录。
    搜索路径 _directories_ 是要搜索的目录列表，由冒号（MS-DOS和MS-Windows上的则是 semi-colons(`;`)）或空白分隔
- `vpath pattern`
    **清除** 与 pattern 关联的搜索路径。
- `vpath`
    **清除** 之前使用 vpath 指令指定的所有搜索路径。

_vpath_ 中的 _pattern_ 是包含 `%` 字符的字符串。该字符串必须与正在搜索的 prerequisite 的文件名相匹配，`%` 字符匹配任何零个或多个字符的序列（如模式规则；请参阅 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules)）。例如，'%. h' 匹配以 .h 结尾的文件。（如果没有“`%`”，则 _pattern_ 必须与 prerequisite 完全匹配，这通常不太有用。）

*vpath* 指令 _pattern_ 中的“%”字符可以被前置反斜杠（“\”）转义。会转义“%”字符的反斜杠可以用更多的反斜杠转义。在将 _pattern_ 与文件名进行比较之前，将从 _pattern_ 中删除转义“%”字符或其他反斜杠的反斜杠。**不存在转义“%”字符危险的反斜杠不会受到干扰**。

当前目录中不存在先决条件时，如果 *vpath* 指令中的 _pattern_ 与先决条件文件的名称匹配，则会像搜索 `VPATH` 变量中的目录一样（以及之前）搜索该指令中的目录。

### 4.5.3 目录搜索是如何执行的

### 4.5.4 使用目录搜索编写 Recipes

这是通过诸如“$^”之类的自动变量来完成的（请参阅自动变量）。

当通过目录搜索在另一个目录中找到 prerequisite 时，这不能改变 rule 的 recipe ；它们将按所写的那样执行。因此，您必须小心编写 recipe，以便它使用 make 找到的 prerequisite 所在目录中的 prerequisite 。

这是通过诸如 '`$^`' 之类的自动变量完成的 (参见 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)）。例如，'`$^`' 的值是 rule 的所有先决条件的列表，包括在其中找到它们的目录的名称，而 '`$@`' 的值则是目标。因此

```Makefile
foo.o : foo.c
        cc -c $(CFLAGS) $^ -o $@ # 译者注：不推荐使用$^
```
（变量CFLAGS存在，因此您可以通过隐式规则为C编译指定标志；我们在此处使用它是为了保持一致性，因此它将统一影响所有C编译；请参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html)。）

通常，先决条件还包括头文件，您不想在 recipe 中提及这些头文件。自动变量 "`$<`" 只是第一个先决条件：

```Makefile
VPATH = src:../headers
foo.o : foo.c defs.h hack.h
        cc -c $(CFLAGS) $< -o $@ # 译者注：推荐使用$<
```

### 4.5.5 目录搜索与隐式规则
在考虑隐式规则（请参阅[10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)）时，也会搜索 `VPATH` 或 `vpath` 中指定的目录。

例如，当文件 foo.o 没有显式规则时，*make* 会考虑隐式规则，例如编译 foo.c 的内置规则（如果该文件存在）。如果当前目录中缺少这样的文件，则会搜索相应的目录。如果在任何目录中都存在 foo.c（或在makefile中提到），则应用c编译的隐式规则。

隐式规则的 recipe 通常会根据需要使用自动变量；因此，他们将使用目录搜索找到的文件名，而不需要额外的努力。

### 4.5.6 链接库的目录搜索
目录搜索以一种特殊的方式应用于与链接器一起使用的库。当您编写名称为“-lname”形式的先决条件时，此特殊功能就会发挥作用。（你可以看出这里发生了一些奇怪的事情，因为先决条件通常是文件名，而库的文件名通常看起来像 libname.a，而不是“-lname”。）

当先决条件的名称的形式为“-lname”时，*make* 对其进行特殊处理，先搜索文件 `libname.so`，如果找不到，则在当前目录中，或在匹配的 vpath 搜索路径和 VPATH 搜索路径指定的目录中，然后在 `/lib`、`/usr/lib`和 `prefix/lib` 目录（通常为 `/usr/local/lib` ，但MS-DOS/MS Windows版本的make的行为就像前缀被定义为 DJGPP 安装树的根一样）中搜索 `libname.a`。

例如，如果您的系统上有 `/usr/lib/libccurses.a` 库（而没有 `/usr/lib/lipccurses.so` 文件），那么

```Makefile
foo : foo.c -lcurses
        cc $^ -o $@
```

当 foo 比 foo.c 或 `/usr/lib/libccurses.a` 旧时，将执行命令 `cc foo.c /usr/lib/libcurses.a -o foo`。

尽管要搜索的默认文件集是 `libname.so` 和 `libname.a` ，但这是可以通过 `.LIBPATTERNS` 变量自定义的。此变量值中的每个单词都是一个模式字符串。当看到诸如“-lname”之类的先决条件时，*make* 将用 *name* 替换列表中每个模式中的百分号(%)，并使用每个库文件名执行上述目录搜索。

`.LIBPATTERNS` 的默认值 是 `lib%.so lib%.a`，它提供了上述默认行为。通过将此变量设置为空值，可以完全关闭链接库展开。

## 4.6 伪目标(Phony Targets)
一个伪目标并不是一个文件的真正名称；相反，它只是当您发出明确请求时要执行的 recipe 的名称。使用伪目标有两个原因：一是为了避免与同名文件发生冲突，二是为了提高性能。

如果您编写的规则的 recipe 不会创建目标文件，则每次出现要重新制作的目标时都会执行该 recipe。以下是一个示例：

```makefile
clean:
        rm *.o temp
```
因为 rm 命令不会创建名为 clean 的文件，所以可能永远不会存在这样的文件。因此，每次您输入 "_make clean_" 时，都会执行 rm 命令。

在本例中，如果在此目录中创建了名为 clean 的文件，则clean目标将无法正常工作。由于它没有先决条件，clean 总是被认为是最新的，它的 recipe 不会被执行。为了避免这个问题，您可以通过**将目标作为特殊目标 .PHONY（见[4.9 Special Built-in Target Names]的先决条件来明确声明它是假的**。如下：

```makefile
.PHONY: clean
clean:
        rm *.o temp
```

完成此操作后，无论是否有名为 clean 的文件，"_make clean_" 都将运行该 recipe 。

.PHONY 的先决条件总是被解释为字面上的目标名称，而不是模式(pattern)（即使它们包含“%”字符）。若要始终重新生成模式规则(pattern rule)，请考虑使用“force target”（请参阅 [4.7  Rules without Recipes or Prerequisites](https://www.gnu.org/software/make/manual/make.html#Force-Targets)）。

伪目标与 make 的递归调用结合使用也很有用（请参阅 [5.7  Recursive Use of make](https://www.gnu.org/software/make/manual/make.html#Recursion)）。在这种情况下，makefile通常会包含一个变量，该变量列出了要构建的多个子目录。处理这一问题的一种简单方法是定义一个 recipe 在子目录上循环的规则，如下所示：

```makefile
SUBDIRS = foo bar baz

subdirs:
        for dir in $(SUBDIRS); do \
          $(MAKE) -C $$dir; \
        done
```

然而，这种方法也存在问题。首先，此规则会忽略在 sub-*make* 中检测到的任何错误，因此即使失败，它也会继续生成其余目录。这可以通过添加 shell 命令来记录错误并退出来的方式克服，但即使使用 -k 选项调用make，它也会这样做，这很不幸。第二点（但也许更重要的一点），您不能充分利用 make 的并行构建目标的能力（请参阅 [5.4 Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel)），因为只有一个规则。每个 makefile 的目标都将并行构建，但一次只能构建一个子目录。

通过将子目录声明为伪目标（您必须这样做，因为子目录显然总是存在的；否则它将无法构建）您可以消除这些问题。

```makefile
SUBDIRS = foo bar baz

.PHONY: subdirs $(SUBDIRS)

subdirs: $(SUBDIRS)

$(SUBDIRS):
        $(MAKE) -C $@

foo: baz
```

这里我们还声明，在子目录 _baz_ 完成之前，不能构建子目录 _foo_；当尝试并行构建时，这种关系声明尤其重要。

隐式规则搜索（请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)）将跳过伪目标。这就是为什么将目标声明为伪目标的性能很好，即使您不担心实际存在的文件。

伪目标不应该是真实目标文件的先决条件；如果是，则每次 make 考虑该文件时都会运行其 recipe (译者注，这里"考虑"的翻译可能有问题，原文是：its recipe will be run every time make considers that file)。只有当伪目标不是真正目标的先决条件时，伪目标的 recipe 才会仅在伪目标作为被指定目标时（请参阅 [9.2 Arguments to Specify the Goals](https://www.gnu.org/software/make/manual/make.html#Goals)）被执行。

您不应该将包含的 makefile 声明为伪。伪目标并不代表真实的文件，而且由于伪目标总是被认为是过时的，所以 make 总是会重新生成它，然后重新执行它自己（请参阅 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/make.html#Remaking-Makefiles)）。为了避免这种情况，如果重新构建了标记为假的包含文件，make 将不会重新执行自己。

伪目标可能有先决条件。当一个目录包含多个程序时，在一个生成文件 _./Makefile_ 中描述所有程序是最方便的。由于默认情况下重新生成的目标将是 makefile 中的第一个目标，因此通常会将其作为名为 “all” 的伪目标，并将所有单独的程序作为先决条件。例如

```makefile
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
        cc -o prog1 prog1.o utils.o

prog2 : prog2.o
        cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
        cc -o prog3 prog3.o sort.o utils.o
```

现在你可以输入 “make” 来重新制作所有三个程序，或者将要重新制作的程序作为参数进行指定（如“make prog1 prog3”）。伪性(Phoniness)不是继承的：伪目标的先决条件本身并不是伪的，除非明确声明是伪的。

当一个伪目标是另一个伪目标的先决条件时，它就充当了另一个伪目标的子程序。例如，此处的“make-cleanall”将删除目标文件、差异文件和名称为 _program_ 的文件：

```makefile
.PHONY: cleanall cleanobj cleandiff

cleanall : cleanobj cleandiff
        rm program

cleanobj :
        rm *.o

cleandiff :
        rm *.diff
```

## 4.7 没有 Recipes 或 Prerequisites 的 Rule
如果一个规则没有先决条件或配方，并且该规则的目标是一个不存在的文件，那么每当运行该规则时，*make* 就会假设该目标已经更新。这意味着所有依赖于此目标的目标都将始终运行其配方（译者注，这里的意思应该是因为先决条件新于目标，所以配方才会运行）。

一个例子将说明这一点：
```makefile
clean: FORCE
    rm $(objects)
FORCE:
```

在这里，目标 *FORCE* 满足特殊条件，因此依赖它的目标 clean 被迫运行其配方。“*FORCE*” 这个名字并没有什么特别之处，但这是一个常用的名字。

正如您所看到的，以这种方式使用 *FORCE* 与使用 “,PHONY: clean” 具有相同的结果。

使用 “.PHONY” 更明确、更高效。但是，make的其他版本不支持“.PHONY”；因此“FORCE”出现在许多makefile中。

## 4.8 空目标文件以记录事件
空目标是伪目标的变体；它用于保存您偶尔发出的明确操作请求的配方。与伪目标不同，此目标文件可以真实存在；但文件的内容并不重要，并且通常是空的。

空目标文件的目的是记录规则的配方最后一次执行的时间及其最后一次修改时间。这样做是因为配方中的一个命令是用于更新目标文件的 `touch` 命令。

空的目标文件应该有一些先决条件（否则就没有意义了）。当你要求重制空目标时，如果任何先决条件比目标更新(换言之，如果自上次重新制作目标后，先决条件发生了更改)，就会执行该配方；以下是一个示例：

```
print: foo.c bar.c
        lpr -p $?
        touch print
```

使用此规则，如果自上次 “make-print” 以来任何一个源文件发生了更改，则 “make-praint” 将执行 `lpr` 命令。自动变量“$？”用于仅打印那些已更改的文件（请参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)）。

## 4.9 特殊内置目标名称
某些名称如果作为目标出现，则具有特殊含义。

.PHONY
    特殊目标 .PHONY 的先决条件被认为是伪目标。当需要考虑这样一个目标时，_make_ 将无条件地运行其配方，无论是否存在具有该名称的文件或其上次修改时间是多少。请参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)。

.SUFFIXES
    特殊目标 .SUFFIXES 的先决条件是用于检查后缀规则的后缀列表。请参阅 [10.7 Old-Fashioned Suffix Rules](https://www.gnu.org/software/make/manual/make.html#Suffix-Rules)。

.DEFAULT

.PRECIOUS

.INTERMEDIATE

.NOTINTERMEDIATE

.SECONDARY

.SECONDEXPANSION

.DELETE_ON_ERROR

.IGNORE

.LOW_RESOLUTION_TIME

.SILENT

.EXPORT_ALL_VARIABLES

.NOTPARALLEL

.ONESHELL

.POSIX
    如果 .POSIX 被提到作为目标，那么 makefile 将被解析并以符合 POSIX 的模式运行。这并不意味着只有符合POSIX 的 makefile 才会被接受，所有高级 GNU make 功能仍然可用。然而，这个目标会导致 make 在默认行为不同的区域按照 POSIX 的要求进行操作。
    特别是，如果提到了这个目标，那么配方将被调用，就像 shell 被传递了 -e 标志一样：配方中的第一个失败命令将导致配方立即失败。

任意定义的隐式规则后缀如果以目标的形式呈现，那么它也将被视为特殊目标，两个后缀的串联也将被算作特殊目标，例如 “.c.o”。这些目标是后缀规则(suffix rule)，这是一种过时的定义隐式规则的方式（但仍被广泛使用）。原则上，如果您将任何目标名称一分为二并将其添加到后缀列表中，则任何目标名称都可能是特殊的。在实践中，后缀通常以“.”开头，所以这些特殊的目标名称也以“.”开头。请参阅 [10.7 Old-Fashioned Suffix Rules](https://www.gnu.org/software/make/manual/make.html#Suffix-Rules)。

## 4.10 一条规则中包含多个目标
当一个显式规则有多个目标时，可以用两种可能的方式之一处理它们：作为独立目标或组目标。处理它们的方式由目标列表后出现的分隔符决定。

**具有独立目标的规则**

使用标准目标分隔符 `:` 的规则定义独立目标。这相当于为每个目标编写一次相同的规则，即前提条件和配方是重复的。通常，配方会使用诸如 `$@` 之类的自动变量来指定正在构建的目标。

具有独立目标的规则在两种情况下很有用：

- 你只需要先决条件，不需要配方。例如

```makefile
kbd.o command.o files.o: command.h
```

为上述三个目标文件中的每一个提供了一个附加的先决条件。它相当于写：

```makefile
kbd.o: command.h
command.o: command.h
files.o: command.h
```

- 类似的配方适用于所有目标。自动变量 `$@` 可用于替换要重新映射到命令中的特定目标（请参阅[10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html)）。例如：

```makefile
bigoutput littleoutput : text.g
    generate text.g -$(subst output,,$@) > $@
```

相当于

```makefile
bigoutput : text.g
    generate text.g -big > bigoutput
littleoutput : text.g
    generate text.g -little > littleoutput
```

在这里，我们假设程序 `program` 生成两种类型的输出，一种是给定 “-big” 的输出，另一种是假定给定 “-little” 的输出。有关子函数的解释，请参阅 [8.2 Functions for String Substitution and Analysis](https://www.gnu.org/software/make/manual/html_node/Text-Functions.html)。

假设您想根据目标更改先决条件，就像变量 `$@` 允许您更改配方一样。在普通规则中不能对多个目标执行此操作，但可以使用静态模式规则执行此操作。

**组目标的规则**

您有一个可以通过一次调用就生成多个文件的配方，而不是独立的目标，那么您可以使用组目标(grouped targets)来声明规则从而表达这种关系。组目标规则使用分隔符 `&:`（此处的“&”表示“all”）。

当 *make* 构建任何一个组目标时，它会认为组中的所有其他目标也会因调用配方而更新。此外，如果只有一些组目标过期或丢失，*make* 将意识到运行配方将更新所有目标。最后，如果任何组目标已过期，则所有组目标都将被视为过期。

例如，此规则定义了一个组目标：

```makefile
foo bar biz &: baz boz
    echo $^ > foo
    echo $^ > bar
    echo $^ > biz
```

在执行组目标的配方期间，自动变量 `$@` 被设置为触发规则的组中特定目标的名称。如果在组目标规则的配方中依赖此变量，则必须小心。

与独立目标不同，组目标规则**必须**包括配方。然而，作为组目标的成员中的目标也可能出现在没有配方的独立目标规则定义中。

每个目标可能只有一个关联的配方。如果一个组目标出现在一个独立的目标规则中，或者出现在另一个带有配方的组目标规则中时，您将收到警告，后一个配方将取代前一个配方。此外，目标将从以前的组中删除，并仅显示在新组中。

如果希望一个目标出现在多个组中，则在声明包含该目标的所有组时，必须使用双冒号组的目标分隔符 `&::`。组的双冒号目标分别独立考虑，如果多个目标中至少有一个需要更新，则每个分组的双冒号规则的配方最多执行一次。

## 4.11 多条规则的目标相同
一个文件可以是多条规则的目标。**所有规则中提到的所有先决条件都合并到目标的一个先决条件列表中。如果目标比任何规则中的任何先决条件都旧，则执行配方**。

对一个文件来说只能有一个配方被执行。如果多个规则为相同文件提供了配方，*make* 将使用最后一个规则并打印一条错误消息。（在特殊情况下，如果文件名以句点开头，则不会打印错误消息。这种奇怪的行为只是为了与 “make…” 的其他实现兼容，您应该避免使用它）。有时，让同一个目标调用在makefile的不同部分中定义的多个配方是有用的；您可以使用双冒号规则（请参阅 [4.13 Double-Colon Rules](https://www.gnu.org/software/make/manual/html_node/Double_002dColon.html)）。

一个只有先决条件的额外规则可以用于同时为多个文件提供一些额外的先决条件。例如，makefile通常有一个变量，如 *objects*，其中包含正在生成的系统中所有编译器输出文件的列表。如果config.h发生更改，则必须重新编译所有这些文件的一个简单方法是编写以下内容：

```makefile
objects = foo.o bar.o
foo.o : defs.h
bar.o : defs.h test.h
$(objects) : config.h
```

它可以插入或取出，而无需更改真正指定如何制作目标文件的规则，如果您希望间歇性地添加额外的先决条件，则可以方便地使用它。

另一个问题是，可以使用传递给 *make* 的命令行参数中设置的变量来指定附加的先决条件（请参阅 [9.5 Overriding Variables](https://www.gnu.org/software/make/manual/html_node/Overriding.html)）。例如

```makefile
extradeps=
$(objects) : $(extradeps)
```

意味着命令 “make extradeps=foo.h” 将把 foo.h 视为每个目标文件的先决条件，而普通的 “make” 则不会。

如果目标的显式规则都没有配方，则搜索适用的隐式规则以找到一个（请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/html_node/Implicit-Rules.html)）。

## 4.12 静态模式规则

静态模式规则(_Static pattern rules_) 是指定多个目标并根据目标名称为每个目标构造先决条件名称的规则。它们比具有多个目标的普通规则更通用，因为目标不必具有相同的先决条件。它们的先决条件必须相似，但不一定相同。

### 4.12.1 静态模式规则语法
以下是静态模式规则的语法：

```makefile
targets …: target-pattern: prereq-patterns …
    recipe
    …
```

*targets list* 指定规则应用的目标。目标可以包含通配符，就像普通规则的目标一样(参阅 [4.4 Using Wildcard Characters in File Names](https://www.gnu.org/software/make/manual/make.html#Wildcards))

*target-pattern* and *prereq-patterns* 说明了如何计算每个 目标 的 先决条件。每个目标都与 *target-pattern* 匹配，以提取目标名称的一部分，称为词干(*stem*)。该 *stem* 被替换到每个 `prereq-patterns` 中，以生成 先决条件名称（每个 *prereq-patterns* 生成一个）。

每个 pattern 通常只包含一次字符“%”。当 *target-pattern* 匹配目标时，“%”可以匹配 目标名称 的任何部分；这部分称为词干(stem)。pattern 的其余部分必须完全匹配。例如，目标 foo. o 匹配模式 "%.o" 时，stem 是 “foo”。目标 foo.c 和 foo.out 不匹配该 pattern。

每个 target 的 prerequisite names 是通过将每个 `prereq-patterns` 式中的 `%` 替换为 stem 来确定的。例如，如果一个 `prereq-patterns` 是 %. c，那么将 stem “foo”替换为 prerequisite names 'foo.c'。编写一个不包含“%”的`prereq-patterns`是合法的；那么这个 prerequisite 对所有目标都是一样的。

pattern rules 中的“%”字符可以用前置反斜杠 `\`  进行转义(quote)。用于转义 % 的前置反斜杠可以被更多的前置反斜杠转义。用于转义 % 和 `\` 的转义符号反斜杠会在与文件名进行比较或替换 stem 之前从模式中删除。但，不是用于转义 % 的 `\` 不受影响，例如，`the\%weird\\%pattern\\` 的结果是 `the%weird\%pattern\\`

这是一个示例，它从相应的 .c 文件编译 foo.o 和 bar.o 中的每一个：
```makefile
objects = foo.o bar.o

all: $(objects)

$(objects): %.o: %.c
        $(CC) -c $(CFLAGS) $< -o $@
```

指定的每个 *target* 都必须与 *target pattern* 匹配；当出现不匹配的 target 时会发出警告。如果您有一个文件列表，其中只有一些与 pattern 匹配，您可以使用 `filter` 函数删除不匹配的文件名，参阅 [8.2 Functions for String Substitution and Analysis](https://www.gnu.org/software/make/manual/make.html#Text-Functions))

```makefile
files = foo.elc bar.o lose.o

$(filter %.o,$(files)): %.o: %.c
        $(CC) -c $(CFLAGS) $< -o $@
$(filter %.elc,$(files)): %.elc: %.el
        emacs -f batch-byte-compile $<
```
在本例中，`$(filter%. o，$(files))` 的结果是 bar.o lose.o，第一个静态模式规则通过编译相应的C源文件来更新这些目标文件。`$(filter%.elc，$(file))`的结果是 foo.elc ，因此该文件是由 foo.el 制作的。

另一个示例展示了如何在静态模式规则中使用`$*`：

```makefile
bigoutput littleoutput : %output : text.g
        generate text.g -$* > $@
```
当 `generate` 命令运行时，`$*` 将展开到词干 ，要么是 'big'，要么是 'little'。

### 4.12.2 静态模式规则与隐式规则

静态模式规则与定义为模式规则的隐式规则有很多共同点（请参阅 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules)）。两者都有目标模式(a pattern for the target)和构造先决条件名称的模式(patterns for constructing the names of prerequisites)。区别在于 `make` 如何决定何时应用规则。

隐式规则可以应用于与其模式匹配的任何目标，但它仅在目标没有其它指定的 配方(recipe) 时、并且可以找到 先决条件 时才适用。如果出现多个隐式规则都可能适用，则只有一个适用；选择取决于规则的顺序。

相比之下，静态模式规则精确地适用于您在规则中指定的目标列表。它不能应用于任何其他目标，并且总是适用于指定的每个目标。如果两个冲突的规则适用，并且都有 配方(recipe) ，那就是错误。

由于以下原因，静态模式规则可能比隐式规则更好：
- 对于那些名称无法在语法上分类但可以在显式列表中给出的文件，您可能希望覆盖这些文件的普通隐式规则。
- 如果您不能确定所使用目录的精确内容，您可能无法确定哪些其他不相关的文件可能会导致使用错误的隐式规则。选择可能取决于隐式规则搜索的顺序。使用静态模式规则，则没有不确定性：每个规则都精确地应用于指定的目标。

## 4.13 双冒号规则
双冒号规则是在目标名称后使用 `::` 编写的显式规则。当同一目标出现在多个规则中时，它们的处理方式与普通规则不同。带有双冒号的模式规则具有完全不同的含义（请参阅 [
10.5.5 Match-Anything Pattern Rules](https://www.gnu.org/software/make/manual/html_node/Match_002dAnything-Rules.html)）。

当一个目标出现在多个规则中时，所有规则都必须是相同的类型：全普通或全双冒号。**如果它们是双冒号，则每个目标都独立于其他目标。如果目标早于该规则的任何先决条件，则执行每个双冒号规则的配方**。如果该规则没有先决条件，则始终执行其配方（即使目标已经存在）。这可能导致不执行任何、任何或所有双冒号规则。

**具有相同目标的双冒号规则实际上是完全独立的。每个双冒号规则都是单独处理的**，就像处理具有不同目标的规则一样。

目标的双冒号规则按照它们在生成文件中的显示顺序执行。然而，**双冒号规则真正有意义的情况是那些执行配方的顺序无关紧要的情况**。

双冒号规则有些晦涩难懂，而且通常不太有用；它们提供了一种机制，用于更新目标的方法因导致更新的先决条件文件而异，这种情况很少见。

每个双冒号规则都应该指定一个配方；如果没有，则在应用时将使用隐式规则。请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/html_node/Implicit-Rules.html)。

## 4.14 自动生成先决条件

在程序的 makefile 中，需要编写的许多规则通常只表示某个目标文件依赖于某个头文件。例如，如果 main.c 通过 `#include` 使用 defs.h ，那么您将编写：

```makefile
main.o: defs.h
```

你需要这个规则，让 *make* 知道无论何时 defs.h 发生变化，它都必须重新制作 main.o 。您可以看到，对于一个大型程序，您必须在 makefile 中编写数十个这样的规则。而且，每次添加或删除 `#include` 时，都必须非常小心地更新 makefile。

为了避免这种麻烦，大多数现代C编译器都可以通过查看源文件中的 `#include` 行来为您编写这些规则。通常，这是通过编译器的 `-M` 选项完成的。例如，命令：

```makefile
cc -M main.c
```

生成输出：

```makefile
main.o : main.c defs.h
```

因此，您不再需要自己编写所有这些规则。编译器会帮你做的。

请注意，这样的规则构成了在 makefile 中提及 main.o，因此通过隐式规则搜索，它永远不能被视为中间文件。这意味着 make 在使用该文件后永远不会删除该文件；请参阅 [10.4 Chains of Implicit Rules](https://www.gnu.org/software/make/manual/html_node/Chained-Rules.html)。

对于旧的 make 程序，传统做法是通过 `make-depend` 等命令使用此编译器功能根据需要生成先决条件。该命令将创建一个包含所有自动生成的先决条件的文件 *depend*，然后 makefile 可以使用 `include` 来读取它们（请参阅 [3.3 Including Other Makefiles](https://www.gnu.org/software/make/manual/html_node/Include.html)）。

在GNU make中，重新制作(译者注，原文为remake，翻译成重新执行可能会更恰当？) makefile 的功能使这种做法过时了——你永远不需要明确地告诉 make 重新制作先决条件，因为它总是重新制作任何过期的 makefile。请参阅如 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/html_node/Remaking-Makefiles.html)。

**我们建议自动生成先决条件的做法是，每个源文件对应一个 _makefile_**。对于每个源文件 name.c，都有一个 makefile name.d，其中列出了目标文件 name.o 所依赖的文件。这样，只需要重新扫描已更改的源文件即可生成新的先决条件。

以下是从名为 name.c 的C源文件生成名为 name.d b的先决条件文件（即makefile）的模式规则：

```makefile
%.d: %.c
    @set -e; rm -f $@; \
     $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
     sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' <$@.$$$$ > $@; \
     rm -f $@.$$$$
```

有关定义模式规则的信息，请参阅 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/html_node/Pattern-Rules.html)。如果 `$(CC)` 命令（或任何其他命令）失败（以非零状态退出），shell的 `-e` 标志将使其立即退出。

对于 GNU C 编译器，您可能希望使用 `-MM` 标志而不是 `-M` 。这省略了系统头文件的先决条件。有关详细信息，请参阅使用 GNU CC 中的 [3.13 Options Controlling the Preprocessor](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)。

`sed` 命令的目的是翻译（例如）：

```makefile
main.o : main.c defs.h
```

into

```makefile
main.o main.d : main.c defs.h
```

这使得每个 “.d” 文件都依赖于相应的“.o”文件所依赖的所有源文件和头文件。然后 make 会知道，每当任何源文件或头文件发生更改时，它都必须重新生成(regenerate)先决条件。

一旦定义了重新制作 ".d" 文件的规则，就可以使用 `include` 指令来读取所有文件。请参阅 [3.3 Including Other Makefiles](https://www.gnu.org/software/make/manual/html_node/Include.html)）。例如

```makefile
sources = foo.c bar.c

include $(sources:.c=.d)
```
（此示例使用替换变量引用将源文件 “foo.c bar.c” 的列表转换为先决条件生成文件 “foo.d bar.d” 的列表。有关替换引用的完整信息，请参阅 [6.3.1 Substitution References](https://www.gnu.org/software/make/manual/html_node/Substitution-Refs.html)。）由于 “.d” 文件与其他文件一样都是 makefile ，make 将根据需要重新制作它们，而无需您做进一步的工作。请参阅 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/html_node/Remaking-Makefiles.html)。

请注意，“.d” 文件包含目标定义；您应该确保将 `include` 指令放在 makefile 中的第一个默认目标之后，或者冒着让随机目标文件成为默认目标的风险。请参阅 [2.3 How make Processes a Makefile](https://www.gnu.org/software/make/manual/html_node/How-Make-Works.html)。

# 5 在规则中编写配方
规则的配方由一个或多个 shell 命令行组成，这些命令行将按照出现的顺序每次执行一个。通常，执行这些命令的结果是使规则的目标更新为最新。

用户可以使用许多不同的 shell 程序，但除非 *makefile* 另有规定，否则 *makefile* 中的配方始终由 /bin/sh 执行。请参阅 [5.3 Recipe Execution](https://www.gnu.org/software/make/manual/html_node/Execution.html)。

## 5.1 配方的语法
*Makefile* 有一个不同寻常的特性，即在一个文件中实际上有两种不同的语法。*makefile* 中大部分内容使用 *make* 语法（请参阅 [3 Writing Makefiles](https://www.gnu.org/software/make/manual/html_node/Makefiles.html)）。然而，配方是由 shell 解释的，因此它们是使用 shell 语法编写的。*make* 程序并不试图理解 shell 语法：在将配方交给 shell 之前，它只对配方的内容执行很少的特定翻译。

配方中的每一行都必须以一个制表符（或 `.RECIPEPREFIX` 变量值中的第一个字符；请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/html_node/Special-Variables.html)）开头，但第一个配方行可以用分号连接到“目标-先决条件”行。*makefile* 中以 tab 开头并出现在“规则上下文”中的任何一行（即，在一个规则启动之后，直到另一个规则或变量定义）都将被视为该规则的配方的一部分。在配方行中可能会出现空行和仅注释行，它们被忽略。

这些规则的一些后果包括：

- 以制表符开头的空行不是空白的：这是一个空配方（请参阅 [5.9 Using Empty Recipes](https://www.gnu.org/software/make/manual/make.html#Empty-Recipes)）。
- **配方中的注释不是注释；它将按原样传递给 shell**。shell 是否将其视为注释取决于您的 shell。
- “规则上下文”中(即以制表符作为行上第一个字符)的变量定义，将被视为配方的一部分，而不是 *make* 变量定义，并传递给 shell。
- “规则上下文”中(即以制表符作为行上第一个字符)的条件表达式（ifdef、ifeq等，请参阅 [7.2 Syntax of Conditionals](https://www.gnu.org/software/make/manual/make.html#Conditional-Syntax)）将被视为配方的一部分，并传递给 shell。

### 5.1.1 拆分配方行
*make* 解释配方的为数不多的方法之一是检查换行符前的反斜杠。与正常的 *makefile* 语法一样，通过在每个换行符之前放置一个反斜杠，可以将 *makefile* 中的单个逻辑配方行拆分为多个物理行。像这样的一系列行被视为单个配方行，将调用 shell 的一个实例来运行它。

但是，与 *makefile* 中其他地方的处理方式不同（请参阅 [3.1.1 Splitting Long Lines](https://www.gnu.org/software/make/manual/make.html#Splitting-Lines)），“反斜杠-换行符对”不会从配方中删除。反斜杠和换行符都会被保留并传递给 shell。如何解释“反斜杠-换行符”取决于您的shell。如果反斜杠/换行后下一行的第一个字符是配方前缀字符（默认情况下为制表符；请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)），则会删除该字符（并且仅删除该字符）。配方中从不添加空白。

例如，此 *makefile* 中目标 “all” 的配方：

```makefile
all :
    @echo no\
space
    @echo no\
    space
    @echo one \
    space
    @echo one\
     space
```

由四个独立的shell命令组成，其中输出为：

```
nospace
nospace
one space
one space
```

作为一个更复杂的例子，这个 makefile：

```makefile
all : ; @echo 'hello \
        world' ; echo "hello \
    world"
```

将使用以下命令调用一个shell：

``` shell
echo 'hello \
world' ; echo "hello \
    world"
```

根据 shell 引用规则，它将产生以下输出：

``` shell
hello \
world
hello     world
```

请注意，“反斜杠-换行符”对是如何在用双引号（" "）引起来的字符串内删除的，而不是从用单引号（' '）引出来的字符串中删除的。这是默认 shell（/bin/sh）处理“反斜杠-换行符对”的方式。如果在 makefile 中指定了不同的 shell，则可能会对它们进行不同的处理。

有时，您希望在单引号内拆分一条长行，但不希望“反斜杠-换行符”出现在引用的内容中。当将脚本传递给 Perl 等语言时，通常会出现这种情况，脚本中多余的反斜杠可能会改变其含义，甚至是语法错误。一种简单的处理方法是将带引号的字符串，甚至整个命令放入 make 变量中，然后在配方中使用该变量。在这种情况下，将使用 makefile 的换行规则，并删除“反斜杠-换行符”。如果我们使用此方法重写上面的示例：

```makefile
HELLO = 'hello \
world'

all : ; @echo $(HELLO)
```
我们将得到这样的输出：
```shell
hello world
```

如果您愿意，您也可以使用特定于目标的变量（请参阅 [6.11 Target-specific Variable Values](https://www.gnu.org/software/make/manual/make.html#Target_002dspecific)）来获得变量和使用它的配方之间更紧密的对应关系。

### 5.1.2 在配方中使用变量
*make* 处理配方的另一种方法是展开其中的任何变量引用（请参阅 [6.1 Basics of Variable References](https://www.gnu.org/software/make/manual/make.html#Reference)）。这发生在 *make* 完成所有 makefile 的读取并且目标被确定为过期之后；因此，不被重建的目标的配方永远不会展开。

配方中的变量和函数引用与 makefile 中其他地方的引用具有相同的语法和语义。他们也有相同的转义规则：如果你想在配方中出现一个美元符号，你必须将其加倍（"$$"）。对于像默认 shell 这样使用美元符号引入变量的shell，重要的是要记住要引用的变量是make变量（使用一个美元符号）还是shell变量（使用两个美元符号）。例如

```makefile
LIST = one two three
all:
        for i in $(LIST); do \
            echo $$i; \
        done
```

导致以下命令被传递到 shell：
```shell
for i in one two three; do \
    echo $i; \
done
```

其产生预期结果：
```
one
two
three
```

## 5.2 配方中的回送显示(echo)
通常情况下，在执行配方之前，*make* 会打印配方的每一行。我们称之为回送显示(echoing)，因为它给人的感觉是你自己在打字。

**当一行以“@”开头时，该行的回显将被抑制**。在将该行传递给 shell 之前，将丢弃“@”。通常，您会将其用于唯一效果是打印某些内容的命令，例如用于指示 makefile 进度的echo命令：

```makefile
@echo About to make distribution files
```

当 *make* 被赋予 `-n` 或 `--just-print` 标志时，它只会回显大多数配方，而不会执行它们。请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/make.html#Options-Summary)。在这种情况下，甚至会打印以“@”开头的配方行。这个标志有助于找出哪些配方是必要的，而不需要 *make* 实际操作。

传递给 *make* 的 `-s` 或 `--silent` 标志可以阻值所有的回显，就好像所有的配方都以 `@` 开头一样。在 makefile 中为特殊目标 “`.SILENT`” 设置的不带先决条件的规则具有相同的效果。（请参阅 [4.9 Special Built-in Target Names](https://www.gnu.org/software/make/manual/make.html#Special-Targets)）

（译者注，考虑两个 `*makefile*`

```makefile
text = print_information

all:
	echo $(text)
```
执行结果会在终端中显示两行，即一行是 `makefile` 中的配方的具体内容，领一行是配方的执行结果
```sh
echo print_information
print_information
```

而增加 `@` 字符后的 `echo`（常用于调试，或查看某个变量的值具体是什么，就不用 [6.3.2 Computed Variable Names](https://www.gnu.org/software/make/manual/make.html#Computed-Names) 中那些复杂内容了 ）
```makefile
text = print_information

all:
	@echo $(text)
```
的执行结果只会在终端显示一行
```sh
print_information
```

）

## 5.3 配方的执行
当需要执行配方来更新目标时，它们是通过为配方的每一行调用一个新的 子shell(sub-shelll) 来执行的，除非特殊目标 `.ONESHELL` 生效（请参阅 [5.3.1 Using One Shell](https://www.gnu.org/software/make/manual/html_node/One-Shell.html)）（在实践中，make可能会采取不影响结果的快捷方式。）

请注意：这意味着设置 shell 变量和调用 shell 命令（如cd）（为每个进程设置本地上下文）不会影响配方中的后续行（官方脚注：在MS-DOS上，当前工作目录的值是全局的，因此更改它将影响这些系统上的后续配方行。）。如果您想使用 cd 来影响下一个语句，请将两个语句放在一个配方行中。那么 make 将调用一个 shell 来运行整行，shell 将按顺序执行语句。例如：

```makefile
foo : bar/lose
    cd $(<D) && gobble $(<F) > ../$@
```

在这里，我们使用 shell 与运算符`&&`，这样，如果 cd 命令失败，脚本将失败，而不会尝试在错误的目录中调用命令 gobble ，如果调用了可能会导致问题（在示例情况下，它肯定会导致../foo被截断）。

### 5.3.1 使用一个 shell
有时，您更希望将配方中的所有行都传递给单个调用的 shell。通常在两种情况下是有用的：首先，它可以通过避免额外的进程来提高由许多命令行组成的配方的 makefile 的性能。其次，您可能希望在您的配方命令中包含换行符（例如，您使用的解释器与SHELL完全不同）。如果特殊目标 `.ONESHELL` 出现在makefile中的任何位置，则每个目标的所有配方行都将提供给单个调用的 shell。配方行之间的换行符将被保留。例如

```makefile
.ONESHELL:
foo : bar/lose
    cd $(<D)
    gobble $(<F) > ../$@
```

现在将按预期工作，即使命令位于不同的配方行上。

如果提供了 `.ONESHELL`，则只会检查配方的第一行中的特殊前缀字符（“`@`”、“`-`”和“`+`”）。调用 shell 时，后续行将包括配方行中的特殊字符。如果你想让你的配方以特殊字符其中一个开头，你需要安排它们不能作为第一行的第一个字符，也许可以添加注释或类似的内容。例如，这将是 Perl 中的语法错误，因为第一个“`@`”是通过make删除的：

```makefile
.ONESHELL:
SHELL = /usr/bin/perl
.SHELLFLAGS = -e
show :
    @f = qw(a b c);
    print "@f\n";
```

然而，这两种替代方案中的任何一种都可以正常工作：

```makefile
.ONESHELL:
SHELL = /usr/bin/perl
.SHELLFLAGS = -e
show :
        # Make sure "@" is not the first character on the first line
        @f = qw(a b c);
        print "@f\n";
```
或
```makefile
.ONESHELL:
SHELL = /usr/bin/perl
.SHELLFLAGS = -e
show :
        my @f = qw(a b c);
        print "@f\n";
```

作为一个特殊功能，如果 SHELL 被确定为 POSIX 风格的 SHELL，则在处理配方之前，“内部”配方行中的特殊前缀字符将被删除。此功能旨在允许现有makefile添加特殊目标 `.ONESHELL` 且无需大量修改的情况下仍然可以正常运行。由于 POSIX shell 脚本中一行开头的特殊前缀字符是不合法的，所以这并不是功能上的损失。例如，这可以按预期工作：
```makefile
.ONESHELL:
foo : bar/lose
    @cd $(@D)
    @gobble $(@F) > ../$@
```

然而，即使有了这个特殊功能，带有 `.ONESHELL` 的 makefile 也会以明显的方式表现出不同的行为。例如，通常情况下，如果配方中的任何一行失败，就会导致规则失败，不再处理更多的配方行。在 `.ONESHELL` 下，*make* 不会注意到除配方最后一行以外的任何一个配方行的错误。您可以修改 `.SHELLFLAGS` 用来将 -e 选项添加到 shell，这将导致命令行中任何位置的任何故障都会导致 shell 失败，但这本身可能会导致您的配方表现不同。最终，你可能需要强化你的配方行，让它们与 `.ONESHELL` 一起工作。

### 5.3.2 选择 shell
用作 shell 的程序取自变量 `SHELL`。如果在 makefile 中未设置此变量，则使用程序 /bin/sh 作为 shell。传递给 shell 的参数取自变量 `.SHELLFLAGS`。`.SHELLFLAGS` 的默认值在正常情况下为 `-c`，在POSIX兼容模式下为 `-ec`。

与大多数变量不同，变量 `SHELL` 绝不从环境中设置。这是因为环境变量 `SHELL` 用于指定您个人选择的交互式 shell 程序。个人选择会影响 makefile 的功能将是非常糟糕的。请参阅 [6.10 Variables from the Environment](https://www.gnu.org/software/make/manual/html_node/Environment.html)。

此外，当您在 makefile 中确实设置了 `SHELL` 时，该值不会从环境中导出到 *make* 调用的配方行。相反只会导出从用户环境（如果有的话）继承的值。您可以通过显式导出 `SHELL`（请参阅 [
5.7.2 Communicating Variables to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Variables_002fRecursion.html)）来覆盖此行为，强制它在环境中传递到配方行。

(
译者注，因为我在翻译这段的时候被绕晕了，就添加一点儿我自己的理解：
- 首先 *shell* 有很多种，比如 /bin/sh 或 /bin/bash 还有 csh tcsh ash 等等，总之就是很多，用户可能会根据习惯，将某种 shell 通过环境变量的方式选择为默认，在 终端terminal 中执行脚本语言的是通过环境变量设置的 shell。
- 当在 终端terminal 中输入 "`make`" 会执行某个 *makefile* 中的内容，根据 [5.3 Recipe Execution](https://www.gnu.org/software/make/manual/html_node/Execution.html)中提到的，执行配方时，会将配方中的内容传给一个新的shell
- *makefile* 想达到的一种效果是，不依赖于在其控制范围之外设置的环境变量，因为这会导致不同的用户从同一个 makefile 中获得不同的结果，这在 [6.10 Variables from the Environment](https://www.gnu.org/software/make/manual/html_node/Environment.html) 中有提到，所以它不管环境变量 `SHELL` 是如何使用的，而默认使用 /bin/sh 来执行配方。
- 但是写 *makefile* 的配方时，此 makefile 的作者不一定用的是 sh 风格的编程语言，即 /bin/sh 不一定能执行它，那么就需要用别的 shell，一般会在 makefile 里写一个变量 `SHELL` 来指定，这样就在 makefile 的控制下了
- 虽然两个 `SHELL` 名称一样，但 makefile 中的 `SHELL` 并不会受到环境变量 `SHELL`的影响。
)

但是，在 MS-DOS 和 MS-Windows 上，使用环境中 `SHELL` 的值，因为在这些系统上，大多数用户不会设置此变量，因此它很可能是专门设置为由 *make* 使用的。在 MS-DOS 上，如果 `SHELL` 的设置不适合 *make*，可以将变量 `MAKESHELL` 设置为 `make `应该使用的 shell；如果设置，它将用作shell，而不是 `SHELL` 的值。

**在DOS和Windows中选择外壳**
暂略

## 5.4 并行执行
GNU *make* 知道如何同时执行多个配方。通常，*make* 一次只执行一个配方，等待它完成后再执行下一个。然而，“`-j`”或“`--jobs`”选项告诉 *make* 同时执行许多配方。您可以从 *makefile* 中禁止某些或所有目标的并行性（请参阅 [5.4.1 Disabling Parallel Execution](https://www.gnu.org/software/make/manual/html_node/Parallel-Disable.html)）。

在 MS-DOS 上，“`-j`”选项无效，因为该系统不支持多处理。

如果“`-j`”选项后面跟着一个整数，这是同时执行的配方数；这被称为作业槽(job slots)的数量。如果“`-j`”选项后面没有类似整数的内容，则作业槽的数量没有限制。作业槽的默认数量是一个，这意味着串行执行（一次执行一件事）。

处理递归 *make* 调用会引发并行执行的问题。有关此方面的详细信息，请参阅 [5.7.3 Communicating Options to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Options_002fRecursion.html)。

如果某个配方失败（被信号终止或以非零状态退出），并且该配方的错误未被忽略（请参阅 [5.5 Errors in Recipes](https://www.gnu.org/software/make/manual/html_node/Errors.html)），则用于重制相同目标的剩余配方行将不会运行。如果配方失败，并且没有给出“`-k`”或“`--keep-going`”选项（请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/html_node/Options-Summary.html)），则中止执行。如果 *make* 由于任何原因（包括信号）终止，但子进程正在运行，*make* 将等待它们完成，然后才真正退出。

当系统负载较重时，您可能希望运行的作业比负载较轻时更少。您可以使用“`-l`”选项告诉make根据平均负载限制同时运行的作业数。“`-l`”或“`--max load`”选项后面跟一个浮点数。例如：

```makefile
-l 2.5
```

如果平均负载高于2.5，则不会让make启动多个作业。不带后置数字的 “`-l`” 选项将删除负载限制（如果之前使用了“`-l`”选项）。

更准确地说，当make启动一个作业，并且它已经有至少一个作业在运行时，它会检查当前的平均负载；如果它不低于“`-l`”给定的限制，则进行等待，直到平均负载低于该限制，或者直到所有其他作业完成。

默认情况下，没有负载限制。

### 5.4.1 禁用并行执行
如果一个 *makefile* 完全准确地定义了其所有目标之间的依赖关系，那么无论是否启用并行执行，*make* 都将正确地构建目标。这是编写 *makefile* 的理想方式。

然而，有时 *makefile* 中的部分或全部目标无法并行执行，并且添加所需的先决条件来通知 *make* 是不可行的。在这种情况下，*makefile* 可以使用各种方法来禁用并行执行。

如果在任何位置指定了不带先决条件的特殊目标“`.NOTPARALLEL`”，那么无论并行设置如何，*make* 的整个实例都将串行运行。例如

```makefile
all: one two three
one two three: ; @sleep 1; echo $@

.NOTPARALLEL:
```

无论如何调用 *make*，目标 *one*、*two* 和 *three* 都将串行运行。

如果特殊目标“`.NOTPARALLEL`”具有先决条件，那么这些先决条件中的每一个都将被视为目标，并且这些目标的所有先决条件都将串行运行。请注意，只有在构建此目标时，先决条件才会串行运行：如果其他目标列出了相同的先决条件，并且不在“`.NOTPARALLEL`”中，则这些先决条件可以并行运行。例如：

```makefile
all: base notparallel

base: one two three
notparallel: one two three

one two three: ; @sleep 1; echo $@

.NOTPARALLEL: notparallel
```

这里“`make -j base`”将并行运行目标 *one*、*two* 和 *three*，而“`make -j notparallel`”将串行运行它们。如果您运行“`make -j all`”，那么它们将并行运行，因为 *base* 将它们列为先决条件，并且没有序列化（serialized）。

目标`.NOTPARALLEL`不应具有命令。

最后，您可以使用特殊目标 `.WAIT`以细粒度的方式(in a fine-grained way)控制特定先决条件的序列化(serialization)。当此目标出现在先决条件列表中并且启用了并行执行时，*make* 将不会生成 `.WAIT` 右侧的任何先决条件，直到位于`.WAIT` 左侧的所有先决条件都已完成。例如：

```makefile
all: one two .WAIT three
one two three: ; @sleep 1; echo $@
```

如果启用了并行执行，*make* 将尝试并行构建目标 *one*、*two*，但在两者都完成之前不会尝试构建目标 *three*。

与提供给的“`.NOTPARALLEL`”目标一样，"`.WAIT`"仅在构建其先决条件列表中显示的目标时生效。如果其他目标中存在相同的先决条件，且不存在`.WAIT`，则它们可能仍然并行运行。也正因为如此，无论是`.NOTPARALLEL`还是`.WAIT`，在控制并行执行方面与定义先决条件关系一样可靠。然而，它们很容易使用，在不太复杂的情况下可能就足够了。

`.WAIT`先决条件将不存在于规则的任何自动变量中。

为了可移植性，您可以在您的 *makefile* 中创建一个 `.WAIT` 实际目标，但这不是使用此功能所必需的。如果`.WAIT`目标已创建，它不应具有先决条件或命令。

`.WAIT`功能也在其他版本的 *make* 中实现，它在POSIX版本的 *make* 标准中指定。

### 5.4.2 并行执行期间的输出
当并行运行多个配方时，每个配方的输出一生成就会出现，结果是来自不同配方的消息可能会穿插在一起，有时甚至会出现在同一行。这会使读取输出变得非常困难。

要避免这种情况，可以使用“`--output-sync`”（“`-O`”）选项。此选项指示 *make* 保存其调用的命令的输出，并在命令完成后全部打印。此外，如果有多个递归 *make* 调用并行运行，它们将进行通信，以便一次只生成(generate)其中一个输出。

如果启用了工作目录打印（请参阅 [5.7.4 The ‘--print-directory’ Option](https://www.gnu.org/software/make/manual/make.html#g_t_002dw-Option)），则会在每个输出分组周围打印输入/离开消息。如果不希望看到这些消息，请将“`--no-print-directory`”选项添加到 `MAKEFLAGS`。

同步输出时有四个粒度(granularity)级别，通过为选项提供参数来指定（例如，“`-Oline`”或“`--output-sync=recurse`”）。

- none<br>这是默认设置：所有输出在生成(generate)时直接发送，不执行同步。
- line<br>配方中每一行的输出都会进行分组，并在该行完成后立即打印。如果一个配方由多行组成，它们可能会穿插其他配方中的行。
- target<br>对每个目标的整个配方的输出进行分组，并在目标完成后打印。如果在没有参数的情况下提供`--output-sync`或`-O`选项，则这是默认值。
- recurse<br>每次 *make* 递归调用的输出都会在递归调用完成后进行分组和打印。

无论选择何种模式，总构建时间都是相同的。唯一的区别在于输出的显示方式。

“`target`”和“`recurse`”模式都会收集目标的整个配方的输出，并在配方完成时不间断地显示。它们之间的区别在于如何处理包含 *make* 递归调用的配方（请参阅 [5.7 Recursive Use of make](https://www.gnu.org/software/make/manual/make.html#Recursion)）。对于所有没有递归行的配方，“`target`”和“`recurse`”模式的行为相同。

如果选择了“`recurse`”模式，则处理包含递归 *make* 调用的配方的方式，将与处理其他目标的方式相同：配方的输出，包括递归 *make* 的输出，都被保存并将在整个配方完成后打印。这确保了由给定递归*make*实例构建的所有目标的输出都被分组在一起，这可能会使输出更容易理解。然而，这也会导致在构建过程中长时间看不到输出，然后涌现大量输出。如果你不是在构建过程中观察构建，而是在构建完成后查看构建日志，这可能是最好的选择。

如果您正在观看输出，那么在构建过程中长时间的安静间隔可能会令人沮丧。“`target`”输出同步模式可以检测何时使用标准方法递归调用*make*，并且不会同步这些行的输出。递归 *make* 将为其目标执行同步，每个目标的输出将在完成后立即显示。请注意，配方的递归行的输出是不同步的（例如，如果递归行在运行*make*之前打印了一条消息，则该消息将不会同步）。

“`line`”模式对于观看制作输出的前端非常有用，以跟踪配方何时开始和完成。

如果 *make* 调用的某些程序确定将输出写入终端而不是文件，则它们的行为可能会有所不同（通常被描述为“交互式(interactive)”与“非交互式(non-interactive)”模式）。例如，如果许多可以显示彩色输出的程序，在确定它们不是在向终端写入，它们就不会这样做。如果您的 *makefile* 调用这样的程序，那么使用输出同步选项将使程序相信它是在“非交互式”模式下运行的，即使输出最终会到达终端。

### 5.4.3 并行执行期间的输入
两个进程不能同时从同一设备获取输入。为了确保只有一个配方尝试同时从终端获取输入，*make* 将使除正在运行的配方外的所有配方的标准输入流无效。如果另一个配方试图从标准输入中读取，通常会产生致命错误（“*Broken pipe*”信号）。

不可预测的是，哪个配方将具有有效的标准输入流（来自终端，或重定向 *make* 的标准输入的任何地方）。第一个配方运行总是会先得到它，在完成后开始的第一个配方会得到下一个，以此类推。

**如果我们找到更好的替代方案，我们将改变 *make* 在这方面的工作方式。同时，如果您使用并行执行功能，则不应依赖任何使用标准输入的配方；但如果你没有使用这个功能，那么标准输入在所有配方中都能正常工作**。

## 5.5 配方中的错误
每次 *shell* 调用返回后，*make* 都会查看其退出状态。如果 *shell* 成功完成（退出状态为零），则在新的 *shell* 中执行配方中的下一行；最后一行执行结束后，规则就执行结束了。

如果出现错误（退出状态为非零），*make* 放弃当前规则，也许放弃所有规则。

有时，某一配方行的错误并不表示有问题。例如，您可以使用 `mkdir` 命令来确保目录存在。如果目录已经存在， `mkdir` 将报告一个错误，但您可能希望 *make* 继续。

要忽略配方行中的错误，请在该行文本的开头（在初始 *tab* 之后）写一个“`-`”。在将该行传递给 *shell* 执行之前，将丢弃“`-`”。

例如:

```makefile
clean:
    -rm -f *.o
```

这导致即使 `rm` 无法删除文件，*make* 也会继续。

当您使用“`-i`”或“`--ignore errors`”标志运行make时，所有规则的所有配方中都会忽略错误。如果没有先决条件，则特殊目标“`.IGNORE`”的 *makefile* 中的规则具有相同的效果。这不太灵活，但有时很有用。

当由于“`-`”或“`-i`”标志而忽略错误时，*make* 会将错误返回视为成功，只是它会打印一条消息，告诉您 *shell* 退出的状态代码，并表示已忽略错误。

当发生一个 *make* 没有被告知要忽略的错误时，这意味着当前目标无法正确地重新生成，直接或间接依赖它的任何其他目标也无法正确地生成。由于这些目标的前提(precondition)尚未实现，因此不会执行这些目标的进一步的配方。

通常情况下，*make* 会在这种情况下立即放弃，返回非零状态。但是，如果指定了“`-k`”或“`--keep-going`”标志，*make* 将继续考虑挂起目标的其他先决条件，如有必要，在放弃并返回非零状态之前对其进行重制。例如，在编译一个目标文件时出错后，“`make -k`”将继续编译其他目标文件，即使它已经知道无法链接它们。请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/make.html#Options-Summary)。

通常的行为会假设您的目的是使指定的目标更新到最新；一旦 *make* 得知这是不可能的，它还不如立即报告失败。“`-k`”选项表示，真正的目的是测试程序中尽可能多的更改，也许是为了找到几个独立的问题，以便在下次尝试编译之前将其全部更正。这就是为什么默认情况下 Emacs(译者注，[GNU Emacs](https://www.gnu.org/software/emacs/) 是一款文本编辑器) 的`compile`命令会默认传递“`-k`”标志。

通常，当配方行出现故障时，如果它已经更改了目标文件，则该文件已损坏，无法使用——或者至少没有完全更新。然而，文件的时间戳表明它现在是最新的，所以下次运行时，它不会尝试更新该文件。这种情况与 shell 被信号终止时的情况完全相同；请参阅 [5.6 Interrupting or Killing make](https://www.gnu.org/software/make/manual/make.html#Interrupts)。因此，如果在开始更改文件后执行配方失败，通常正确的做法是删除目标文件。如果“`.DELETE_ON_ERROR`”作为目标出现，*make* 将执行此操作。这几乎总是你想做的，但这不是历史实践；因此，为了实现兼容性，您必须明确地请求它。

## 5.6 中断或终止make
如果 *make* 在执行 *shell* 时收到致命信号，它可能会删除配方应该更新的目标文件。如果目标文件的上次修改时间自 *make* 第一次检查后发生了更改，则会执行此操作。

删除目标的目的是确保在下次运行 *make* 时从头开始重新制作它。为什么会这样？假设您在编译器运行并且它已经开始写入目标文件 *foo.o* 时键入 `Ctrl-c`。`Ctrl-c`会终止编译器，导致一个不完整的文件，其上次修改时间比源文件 *foo.c* 新。但是 *make* 也会收到 `Ctrl-c` 信号并删除此不完整的文件。如果 *make* 没有这样做，那么下一次调用 *make* 时会认为 *foo.o* 不需要更新——当链接器试图链接一个目标文件时，会从链接器中产生一条奇怪的错误消息，其中一半丢失了。

您可以通过将特殊目标`.PRECIOUS`依赖于此目标文件，来防止以这种方式删除目标文件。



## 5.7 make的递归使用
递归使用 *make* 意味着在 makefile 中使用 `make` 作为命令。当组成更大系统的各个子系统需要分离的 *makefile* 时，此技术非常有用。例如，假设您有一个子目录 *subdir*，它有自己的 *makefile*，并且您希望包含目录的 *makefile* 在该子目录上运行 *make*。你可以这样写：

``` makefile
subsystem:
    cd subdir && $(MAKE)
```

或者等效地这样写

``` makefile
subsystem:
    $(MAKE) -C subdir
```

只需复制此示例，就可以编写递归 *make* 命令，但关于它们的工作方式和原因，以及 sub-*make* 与 顶层make 的关系，还有很多事情需要了解。您可能还发现将调用递归make命令的目标声明为 `.PHONY` 也很有用（有关何时有用的更多讨论，请参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html)）。

为了方便起见，当GNU make启动时（在它处理了所有 `-c` 选项之后），它将变量 `CURDIR` 设置为当前工作目录的路径名。之后`make`再也不会触及此值：特别注意，如果您包含其他目录中的文件，则 `CURDIR` 的值不会更改。该值的优先级与在 *makefile* 中设置该值时的优先级相同（默认情况下，环境变量 `CURDIR` 不会覆盖该值）。请注意，设置此变量对 *make* 的操作没有影响（例如，它不会导致 *make* 更改其工作目录）。

### 5.7.1 变量`MAKE`的工作原理
递归 *make* 命令应始终使用变量 `MAKE`，而不是显式命令`make`，如下所示：

```makefile
subsystem:
    cd subdir && $(MAKE)
```

此变量的值是用于调用 *make* 的文件名。如果此文件名为 `/bin/make` ，则执行的配方为 `cd subdir&&/bin/make`。如果使用特殊版本的 *make* 来运行 顶层makefile，那么递归调用将使用相同的特殊版本来执行。

作为一项特殊功能，在规则的配方中使用变量 `MAKE` 会更改`-t`（`--touch`）、`-n`（`--just-print`）或`-q`（`-question`）选项的效果。使用 `MAKE` 变量的效果与在配方行开头使用 `+` 字符的效果相同。请参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/html_node/Instead-of-Execution.html)。只有当 `MAKE` 变量直接出现在配方中时，才会启用此特殊功能：如果通过展开另一个变量引用 `MAKE` 变量，则此功能不适用。在后一种情况下，您必须使用 `+` 标记才能获得这些特殊效果。

请考虑上面示例中的命令“`make-t`”。（“`-t`”选项在不实际运行任何配方的情况下将目标标记为最新；请参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/html_node/Instead-of-Execution.html)。）根据“`-t`”的常见定义，示例中的“`make -t`”命令将创建一个名为 *subsystem* 的文件，而不执行其他操作。你真正想让它做的是运行 `cd subdir && make -t`；但这需要执行配方，“`-t`”表示不执行配方。

这个特殊功能使它可以随心所欲：每当规则的配方行包含变量 `MAKE` 时，标志“`-t`”、“`-n`”和“`-q`”都不适用于该行。包含 `MAKE` 的配方行正常执行，尽管存在导致大多数配方不运行的标志。通常的 `MAKEFLAGS` 机制将标志传递给 sub-*make*（请参阅 [5.7.3 Communicating Options to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Options_002fRecursion.html)），因此您新建文件（译者注，原文中是 touch files， 但联想了一下 linux 中 `touch` 命令是用来新建也给文件的）或打印配方的请求会传播到子系统。

### 5.7.2 向 sub-*make* 传递变量

顶层 *make* 的变量值可以用显式请求通过环境传递给 sub-*make*。这些变量在 sub-*make* 中定义为默认值，但它们不会覆盖 sub-*make* 使用的 *makefile* 中定义的变量，除非使用 “`-e`” 开关（请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/html_node/Options-Summary.html)）。

要向下传递或导出变量，*make* 将变量及其值添加到运行配方每一行的环境中。相应地，sub-*make* 使用环境来初始化其变量值表。请参阅 [6.10 Variables from the Environment](https://www.gnu.org/software/make/manual/html_node/Environment.html)。

除非通过明确的请求，否则仅当变量最初在环境中定义，或者在命令行上设置并且其名称仅由字母、数字和下划线组成时，才使导出成为变量。

*make* 变量 `SHELL` 的值不会被导出。相反，调用环境中的 `SHELL` 变量的值会传递给 sub-*make*。您可以使用 `export` 指令强制 *make* 为 `SHELL` 导出其值，如下所述。请参阅 [5.3.2 Choosing the Shell](https://www.gnu.org/software/make/manual/html_node/Choosing-the-Shell.html)。

特殊变量 `MAKEFLAGS` 始终被导出（除非您取消导出它）。将 `MAKEFILES` 设置为任意值即可导出它。

*make* 通过将命令行上定义的变量值放入 `MAKEFLAGS` 变量中，自动向下传递这些值。请参阅 [5.7.3 Communicating Options to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Options_002fRecursion.html)。

如果变量是由 *make* 默认创建的，则通常不会向下传递（请参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html)）。sub-*make*将自己定义这些。

如果要将特定变量导出到 sub-*make*，请使用`export`指令，如下所示：

```makefile
export variable …
```

如果要阻止变量导出，请使用 `unport` 指令，如下所示：

```makefile
unexport variable …
```

在这两种形式中，要 `export` 和 `unexport` 的参数都是展开的，要展开为变量名（列表）的变量或函数也可以要导出(或不导出)。

为了方便起见，您可以定义一个变量并同时导出它，方法是：

```makefile
export variable = value
```

具有与以下相同的结果：

```makefile
variable = value
export variable
```

```makefile
export variable := value

# equivalent to

variable := value
export variable
```

```makefile
export variable += value

# equivalent to

variable += value
export variable
```
请参阅 [6.6 Appending More Text to Variables](https://www.gnu.org/software/make/manual/html_node/Appending.html)

您可能会注意到，`export` 和 `unexport` 指令在 *make* 中的工作方式与它们在 *shell sh* 中的工作方式相同。

如果希望默认情况下导出所有变量，则可以单独使用 `export`：

```sh
export
```

这告诉 *make* 应该导出 `export` 和 `unexport` 指令中未明确提及的变量。`unexport` 指令中给定的任何变量仍然不会导出。

导出指令本身引发的行为是旧版本 GNU *make* 中的默认行为。如果您的 *makefile* 依赖于此行为，并且希望与旧版本的 make 兼容，则可以添加特殊目标 `.EXPORT_ALL_VARIABLES` 到 makefile，而不是使用 `export` 指令。这将被旧的 *make* 忽略，而`export` 指令将导致语法错误。

默认情况下，使用`export`本身或`.export_ALL_VARIABLES`导出变量时，仅导出名称仅由字母数字和下划线组成的变量。若要导出其他变量，必须在`export`指令中特别提及它们。

将变量的值添加到环境中需要对其进行展开。如果展开变量有副作用（如`info`或`eval`或类似函数），则每次调用命令时都会看到这些副作用。您可以通过确保这些变量的名称在默认情况下不可导出来避免这种情况。然而，更好的解决方案是根本不使用这种“默认导出”功能，而是按名称显式`export`相关变量。

默认情况下，您可以使用`unexport`本身来告诉 *make* 不要导出变量。由于这是默认行为，因此只有在早些时候使用过“`export`”本身（可能在包含的 makefile 中），才需要执行此操作。您不能使用`export`和`unexport`本身来导出某些配方的变量，而不能导出其他配方的变量。最后一个单独出现的`export`或`unexport`指令决定整个 *make* 运行的行为。

作为一项特殊功能，变量 `MAKEVEL` 在从一个级别传递到另一个级别时会发生更改。此变量的值是一个字符串，它是以十进制数表示的级别深度。顶层 *make* 的值为"0"；"1"代表一个 sub-*make*，"2"代表一个 *sub-sub-make*，依此类推。当 *make* 为配方设置环境时，增量就会发生。

`MAKLELEVEL`的主要用途是在条件指令中测试它（请参阅 [7 Conditional Parts of Makefiles](https://www.gnu.org/software/make/manual/make.html#Conditionals)）；通过这种方式，您可以编写一个 *makefile*，如果以递归方式运行，则以一种方式运行，如果由您直接运行，则采用另一种方式。

可以使用变量`MAKEFILES`使所有 sub-*make* 命令使用其他生成文件。`MAKEFILES`的值是一个以空格分隔的文件名列表。如果在外部级别的 *makefile* 中定义了该变量，则该变量将通过环境传递；然后它作为一个额外的 *makefile* 列表，供sub-*make*在通常或指定的makefile之前读取。请参阅 [3.4 The Variable MAKEFILES](https://www.gnu.org/software/make/manual/make.html#MAKEFILES-Variable)。

### 5.7.3 将选项传达给sub-*make*
诸如“`-s`”和“`-k`”之类的标志通过变量`MAKEFLAGS`自动传递给 sub-*make*。此变量由*make*自动设置，以包含 *make* 收到的标志字母。因此，如果执行“`make-ks`”，则`MAKEFLAGS`将获得值“`ks`”。

因此，每个 sub-*make* 都会在其环境中获得一个`MAKEFLAGS`值。作为响应，它从该值中获取标志，并将它们作为参数进行处理。请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/make.html#Options-Summary)。这意味着，与其他环境变量不同，环境中指定的`MAKEFLAGS`优先于 *makefile* 中指定的`MAKEFLAGS`。

`MAKEFLAGS`的值可能是一组空字符，表示不带参数的单字母选项，后面跟一个空格以及带参数或具有长选项名称的任何选项。如果一个选项同时具有单字母选项和长选项，则始终首选单字母选项。如果命令行上没有单字母选项，则`MAKEFLAGS`的值以空格开头。

同样，命令行上定义的变量也通过`MAKEFLAGS`传递给 sub-*make*。`MAKEFLAGS`值中包含“=”的单词，将其视为变量定义，就像它们出现在命令行上一样。请参阅 [9.5 Overriding Variables](https://www.gnu.org/software/make/manual/make.html#Overriding)。

未放入`MAKEFLAGS`的选项"`-C`"、"`-f`"、"`-o`"和"`-W`"；这些选项不会被传递下去。

"`-j`" 选项是一种特殊情况（请参阅 [5.4 Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel)）。如果您将其设置为某个数值“N”，并且您的操作系统支持它（大多数UNIX系统都支持；其他系统通常不支持），则父make和所有子make将进行通信，以确保它们之间只有“N”个作业同时运行。请注意，任何标记为递归的作业（请参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/make.html#Instead-of-Execution)）都不计入总作业数（否则，我们可能会运行“N”个 sub-*make*，并且没有剩余的槽位用于任何实际作业！）

如果您的操作系统不支持上述通信，则不会向`MAKEFLAGS`中添加 "`-j`"，因此 sub-*make* 将以非并行模式运行。如果 "`-j`" 选项被传递给 sub-*make*，你会得到比你要求的更多的并行工作。如果你给出的 "`-j`"没有数字参数，意思是并行运行尽可能多的作业，这一点会被传递下去，因为多个无穷大也不过就是一个无穷大（译者注：原文是 since multiple infinities are no more than one）。

如果不想传递其他标志，则必须更改`MAKEFLAGS`的值，例如：

```makefile
subsystem:
    cd subdir && $(MAKE)MAKEFLAGS=
```

命令行变量定义实际出现在变量 `MAKEOVERRIDES` 中，并且 `MAKEFLAGS` 包含对此变量的引用。如果您确实想正常传递标志，但不想传递命令行变量定义，可以将`MAKEOVERRIDES`重置为空，如下所示：

```makefile
MAKEOVERRIDES =
```

这样做通常没有用处。但是，有些系统对环境的尺寸有一个小的固定限制，将太多信息放入`MAKEFLAGS`的值可能会超过这个限制。如果您看到错误消息 “**Arg-list too long**”，这可能是问题所在。（为了严格遵守POSIX.2，如果特殊目标“.POSIX”出现在生成文件中，则更改`MAKEOVERRIDES`不会影响`MAKEFLAGS`。您可能并不关心这一点。）

为了历史兼容性，还存在类似的变量`MFLAGS`。它与`MAKEFLAGS`具有相同的值，只是它不包含命令行变量定义，并且它总是以连字符(hyphen)开头，除非它为空（`MAKEFLAGS`只有在它以没有单字母版本的选项（如"`--warn undefined variables`"）开头时才以连字符开始）。以往`MFLAGS`在递归 *make* 命令中显式使用，如下所示：

```makefile
subsystem:
    cd subdir && $(MAKE) $(MFLAGS)
```

但现在 `MAKEFLAGS` 使这种使用变得多余。如果您希望您的 *makefile* 与旧的 *make* 程序兼容，请使用此技术；它也可以与更现代的版本配合使用。

如果您希望在每次运行 *make* 时设置某些选项，如“`-k`”（请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/html_node/Options-Summary.html)）），则`MAKEFLAGS`变量也很有用。您只需在您的环境中为`MAKEFLAGS`设置一个值。您也可以在 *makefile* 中设置 `MAKEFLAGS`，以指定其他标志，这些标志也应该对该 *makefile* 有效。（请注意，不能以这种方式使用`MFLAGS`。该变量的设置仅用于兼容性；*make* 不会以任何方式解释您为其设置的值。）

当 *make* 解释 `MAKEFLAGS` 的值（无论是来自环境或 *makefile* ）时，如果该值尚未以连字符开头，它首先会在前加一个连字符。然后，它将值切割成用空格分隔的单词，并将这些单词解析为命令行上给定的选项（除了“`-C`”、“`-f`”、“`-h`”、“`-o`”、“`-W`”及其长名称版本被忽略；无效选项不会出错）。

如果您确实将`MAKEFLAGS`放在环境中，则应确保不要包含任何会严重影响 *make* 操作并破坏 *makefile* 和 *make* 本身目的的选项。例如，“-t”、“-n”和“-q”选项，如果放在其中一个变量中，可能会产生灾难性的后果，而且肯定会产生至少令人惊讶和可能令人讨厌的影响。

如果除了GNU *make*之外，您还想运行 *make* 的其他实现，因此不想将GNU *make*特定的标志添加到`MAKEFLAGS`变量中，则可以将它们添加到`GNUMAKEFLAGS`变量中。此变量在`MAKEFLAGS`之前进行解析，解析方式与`MAKEFLAGS`相同。当 *make* 构造`MAKEFLAGS`传递给 递归*make* 时，它将包括所有标志，甚至是从`GNUMAKEFLAGS`获取的标志。因此，在解析`GNUMAKEFLAGS`之后，GNU make将此变量设置为空字符串，以避免在递归过程中重复标志。

最好只将`GNUMAKEFLAGS`与不会实质性改变 *makefile* 行为的标志一起使用。如果您的 *makefile* 无论如何都需要 GNU Make，那么只需使用`MAKEFLAGS`。诸如“`--no-print-directory`”或“`--output-sync`”之类的标志可能适用于 `GNUMAKEFLAGS`。

### 5.7.4 "--print directory"选项
如果您使用多个级别的递归 *make* 调用，“`-w`”或“`--print directory`”选项可以在 *make* 开始处理和 *make* 完成处理时显示每个目录，从而使输出更容易理解。例如，如果“`make -w`”在目录 */u/gnu/make* 中运行，*make*将在做任何事情之前打印以下形式的一行：

```sh
make: Entering directory `/u/gnu/make'.
```
并且会在过程完成后打印以下形式的一行：
```sh
make: Leaving directory `/u/gnu/make'.
```

通常，您不需要指定此选项，因为 *make* 可以为您指定：当您使用“`-C`”选项时，“`-w`”会自动打开，并且在 sub-*make*S 中也是如此。如果同时使用“`-s`”（表示为静默），或者使用“`--no print directory`”显式禁用它，make将不会自动启用“`-w`”。

## 5.8 定义预制配方(Canned Recipes)
当相同的命令序列在制作各种目标时很有用时，可以使用`define`指令将其定义为预制序列，并从这些目标的配方中引用预制序列。预制序列实际上是一个变量，因此名称不能与其他变量名称冲突。

以下是定义预制配方的示例：
```makafile
define run-yacc =
yacc $(firstword $^)
mv y.tab.c $@
endef
```

这里，`run-yacc`是所定义的变量的名称；`endef`标志着定义的结束；中间的行是命令(command)。`define`指令不会预制序列中展开变量引用和函数调用；“`$`”字符、圆括号、变量名等都将成为您定义的变量值的一部分。有关`define`指令的完整解释，请参阅 [6.8 Defining Multi-Line Variables](https://www.gnu.org/software/make/manual/make.html#Multi_002dLine)。

本例中的第一个命令在使预制序列的规则的第一个先决条件下运行Yacc。Yacc的输出文件始终命名为 *y.tab.c*。第二个命令将输出改动至规则的目标文件名。

要使用预制序列，请将变量替换为规则的配方。您可以像替换任何其他变量一样替换它（请参阅 [6.1 Basics of Variable References](https://www.gnu.org/software/make/manual/make.html#Reference)）。因为`define`定义的变量是递归展开的变量，所以您在`define`中编写的所有变量引用，此时都被展开了。例如：

```makefile
foo.c : foo.y
    $(run-yacc)
```
当变量"`$^`"出现在`run yacc`的值中时，"`foo.y`"将替换它，"`foo.c`"将替换"`$@`"。

这是一个实际的例子，但实际上不需要这个特定的例子，因为 *make* 有一个隐式规则来根据所涉及的文件名来计算这些命令（请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)）。

在配方执行中，预制序列的每一行，都被视为该行以前置一个 *tab* 键的形式在规则中单独出现。特别是，*make* 为每一行调用一个单独的子shell。您可以在预制序列的每一行使用影响命令行（“@”、“-”和“+”）的特殊前缀字符。请参阅 [5 Writing Recipes in Rules](https://www.gnu.org/software/make/manual/make.html#Recipes)。例如，使用此预制序列：

```makefile
define frobnicate =
@echo "frobnicating target $@"
frob-step-1 $< -o $@-step-1
frob-step-2 $@-step-1 -o $@
endef
```
*make* 不会回显第一行，即回显命令。但它将回显后续的两条配方行。

另一方面，配方行上引用预制序列的前缀字符适用于序列中的每一行。所以规则是：

```makefile
frob.out: frob.in
    @$(frobnicate)
```

不回显任何配方行。（有关“`@`”的完整解释，请参阅 [5.2 Recipe Echoing](https://www.gnu.org/software/make/manual/make.html#Echoing)。）


## 5.9 使用空配方

有时定义什么都不做的配方是有用的。这只给出一个只包含空格的 recipe 即可（译者注：在 10.1 里说的是要写一个分号，这是为了不应用隐式规则）。例如：

``` Makefile
target: ;
```
这为目标定义了一个空配方。您也可以使用以配方前缀字符开头的一行来定义空配方，但这会令人困惑，因为这一行看起来是空的。（译者注：例如
```makefile
# 注意第二行不是空的，而是有一个 tab 键的，默认没有修改过默认的 .RECIPEPREFIX 值
target: 

```
关于这两种形式，参阅 [4.2 Rule Syntax](https://www.gnu.org/software/make/manual/make.html#Rule-Syntax)
）

你可能想知道为什么你想定义一个什么都不做的配方。这很有用的一个原因是防止目标获取隐式配方（来自隐式规则或特殊目标 `.DEFAULT`；请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules) 和 [10.6 Defining Last-Resort Default Rules](https://www.gnu.org/software/make/manual/make.html#Last-Resort)）。

空配方也可以用来避免某些目标（这些目标会作为另一个配方的副作用被创建）的错误：如果目标不存在，空配方确保 *make* 不会抱怨它不知道如何构建目标，并且 *make* 会假设目标已经过时。

您可能倾向于为“不是实际文件的、其存在只是为了重新制作其先决条件”的目标定义空配方。但是，这不是最好的方法，因为如果目标文件确实存在，则可能无法正确地重新生成先决条件。有关更好的方法，请参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)。


# 6 如何使用变量

变量是定义在 *makefile* 中用于表示一个文本字符串（被称作变量的值）的名称。这些值由显式请求被替换为目标、先决条件、配方和 *makefile* 的其他部分。（在有些版本的 *make* 中变量也被称作宏(macros)）

除配方、使用“`=`”的变量定义的右侧以及使用`define`指令的变量定义体除外，*makefile* 中的所有部分中的变量和函数都会在被读取时展开。变量展开到的值是展开时其最新定义的值。换句话说，变量是动态作用域的。

变量可以表示文件名列表、要传递给编译器的选项、要运行的程序、要查找源文件的目录、要写入输出的目录，或者您可以想象的任何其他内容。

变量名可以是不包含“`:`”、“`#`”、“`=`”或空白字符的任意字符序列。但是，应仔细考虑包含字母、数字和下划线以外的字符的变量名，因为在某些 shell 中，它们无法通过环境传递给 sub-*make*（请参阅 [5.7.2 Communicating Variables to a Sub-make](https://www.gnu.org/software/make/manual/make.html#Variables_002fRecursion)）。以“`.`”和大写字母开头的变量名可能在未来版本的 *make* 中被赋予特殊含义。

变量名区分大小写。名称“foo”、“FOO”和“Foo”是不同的变量。

传统上在变量名中会使用大写字母，但我们推荐在 *makefile* 中为内部目的变量使用小写字母，并为控制隐式规则的参数或用户应使用命令选项覆盖的参数保留大写字母（请参阅 [9.5 Overriding Variables](https://www.gnu.org/software/make/manual/make.html#Overriding)）。

一些变量的名称是单个标点符号或只有几个字符。这些是自动变量，它们有特殊的特殊用途。请参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)。
## 6.1 变量引用基础

要替换变量的值，请在一个美元符号后紧跟括号或者大括号围起来的变量名称，例如`$(foo)` 或者 `${foo}`都是对变量 _foo_ 的有效引用。由于 `$` 的特殊意义，要在文件名或者配方中达到单个美元符号的效果时，应使用 ‘`$$`’。

变量引用可以在任何上下文中使用：目标、先决条件、配方、大多数指令和新变量值。以下是一个常见情况的示例，其中变量包含程序中所有目标文件的名称：

```makefile
objects = program.o foo.o utils.o
program : $(objects)
    cc -o program $(objects)

$(objects) : defs.h
```

变量引用通过严格的文本替换来工作。因此，规则

```makefile
foo = c
prog.o : prog.$(foo)
    $(foo)$(foo) -$(foo) prog.$(foo)
```
可用于编译 C 程序 *prog.c*。由于变量值之前的空格在变量赋值中被忽略，因此 *foo* 的值正是“c”。（实际上不要这样写*makefile*！）

一个美元符号后面紧跟的单字符（除美元符号、左括号或左大括号外），会被视为变量名。因此，您可以使用“`$x`”引用变量`x`。然而，这种做法可能会导致混淆（例如，“`$foo`”指的是变量`f`后面跟着字符串`oo`），因此我们建议在所有变量周围使用括号或大括号，即使是单字符变量，除非省略括号会显著提高可读性。经常用于提高可读性的一个地方是自动变量（请参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)）。


## 6.2 变量的两种风格

GNU *make* 中的变量可以通过不同的方式获得值，我们称之为“变量的风格”。风格的区别在于它们如何处理在 *makefile* 中分配的值，以及在以后使用和展开变量时如何管理这些值。

### 6.2.1 递归展开变量赋值

变量的第一种风格是递归展开变量(*recursively expanded variable*)。此种风格的变量的定义使用 `=`（参阅 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting)) 或者 `define` 指令 (参阅 [6.8 Defining Multi-Line Variables](https://www.gnu.org/software/make/manual/make.html#Multi_002dLine))。您指定的值是逐字被添加的；**如果它包含对其他变量的引用，那这些引用会在此变量被替换时（在展开其他字符串的过程中）被展开**。当这种情况发生时，称为递归展开(recursive expansion)。

```makefile
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all:;echo $(foo)
```
结果会显示 "`Huh?`"，其过程是 "`$(foo)`" 展开成 "`$(bar)`" 又展开成 "`$(ugh)`" 最后展开成 "`Huh?`"。

这种风格的变量是大多数其他版本的 *make* 支持的唯一一类。它有优点也有缺点。一个优点（大多数人会说）是：

```makefile
CFLAGS = $(include_dirs) -O
include_dirs = -Ifoo -Ibar
```

将执行预期操作：当“CFLAGS”在配方中展开时，它将展开到“`-Ifoo -Ibar -O`”。

这种变量风格的缺点是如果对一个变量多次使用 `=` 进行赋值，*make* 只会取其最后一次赋值；而如果想要在本身的后面追加内容，则会引起无限循环。

```makefile
CFLAGS = $(CFLAGS) -O
```

它会造成无限循环（实际上 *make* 会检测到无限循环并报错）。


另一个缺点是，每当变量展开时，定义中引用的任何函数（请参阅 [8 Functions for Transforming Text](https://www.gnu.org/software/make/manual/make.html#Functions)）都会被执行。这会使 *make* 运行速度变慢；更糟糕的是，它会导致 ` wildcard` 和 `shell` 函数产生不可预测的结果，因为您无法轻松控制它们何时被调用，甚至无法控制调用次数。

### 6.2.2 *简单展开变量* 赋值

为了避免递归展开变量的问题和不便，还有另一种风格：*简单展开变量*(*simply expanded variables*)。*简单展开变量*由使用 `:=` 或 `::=` 的行定义（请参阅 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting)）。这两种形式在 GNU *make* 中是等效的；然而，POSIX标准仅描述了 "`::=`" 形式（POSIX Issue 8 的 POSIX 标准中添加了对 "`::=`" 的支持）。

*简单展开变量*的值**只会被扫描一次，并在变量定义时展开对其他变量和函数的任何引用。一旦展开完成，变量的值就再也不会展开了**：当此变量被使用时，变量值会按照之前展开的那样被逐字复制。如果变量值当中包含了变量引用，那么展开的结果中，包含的是定义该简单展开变量时的其它变量引用展开的值。。因此

```makefile
x := foo
y := $(x) bar
x := later
```

相当于 

```makefile
y := foo bar
x := later
```

（译者注：在再次使用 `:=` 或 `::=` 赋值时，会覆盖掉之前的定义。

再举一个例子来分别 `=` 和 `:=`

```
name = zzk
curname = $(name)
name = zuozhongkai

printf:
    @echo curname: $(curname)
```
结果是 `curname: zuozhongkai`，使用 `=` 变量的真实值取决于它所引用的变量的最后一次有效值

```
name = zzk
curname := $(name)
name = zuozhongkai

printf:
    @echo curname: $(curname)
```
结果是 `curname: zzk`，使用 `:=` 变量的真实值会被立即展开，即使它所引用的变量的值变更也不会随之改变了
）

这里有一个稍微复杂一些的例子，说明了“`:=`”与 `shell` 函数（请参阅 [8.14 The shell Function](https://www.gnu.org/software/make/manual/make.html#Shell-Function)）的结合使用。此示例还显示了变量 `MAKEVEL` 的使用，当它从一个级别传递到另一个级别时，该变量会发生更改。（有关 `MAKEVEL` 的信息，请参阅 [5.7.2 Communicating Variables to a Sub-make](https://www.gnu.org/software/make/manual/make.html#Variables_002fRecursion)）

```makefile
ifeq (0,${MAKELEVEL})
whoami    := $(shell whoami)
host-type := $(shell arch)
MAKE := ${MAKE} host-type=${host-type} whoami=${whoami}
endif
```

使用 “`:=`” 的一个优点是，典型的“下降到目录中”配方看起来会像这样：

```makefile
${subdirs}:
    ${MAKE} -C $@ all
```

*简单展开变量*通常会使复杂的 *makefile* 程序更具可预测性，因为它们的工作方式与大多数编程语言中的变量类似。它们允许您使用变量自身的值（或由某个展开函数以某种方式处理的此变量值）重新定义变量，并更有效地使用展开函数（请参阅 [8 Functions for Transforming Text](https://www.gnu.org/software/make/manual/make.html#Functions)）。

您还可以使用它们将受控的前导空格引入变量值中。在替换变量引用和函数调用之前，您输入的前导空白字符将被丢弃；这意味着您可以通过使用变量引用保护前导空格，从而在变量值中包含前导空格，如下所示：

```makefile
nullstring :=
space := $(nullstring) # end of the line
```

这里，变量 *space* 的值恰好是一个空格。此处包含注释“# end of the line”只是为了清楚起见。由于尾部空格字符不会从变量值中去除，因此仅在行末尾的一个空格也会有同样的效果（但可读性不佳）。如果在变量值的末尾添加空白字符，那么最好在行的末尾添加这样的注释，以明确您的意图。相反，如果您不希望在变量值的末尾出现任何空白字符，则必须记住不要在某些空白之后的行末尾放置随机注释，例如：

```makefile
dir := /foo/bar    # directory to put the frobs in
```

这里，变量 `dir` 的值是'/foo/bar    '（后面有四个空格），这可能不是您想要的。（想象一下类似于“`$(dir)/file`”这样的定义！）

### 6.2.3 立即展开变量赋值

对于立即展开，与简单赋值不同的另一种被允许的赋值形式，其生成的变量是递归的：每次使用时都会再次展开。为了避免意外的结果，值被立即展开后，它将自动被转义：展开后的值中的所有 `$` 实例都将转换为 `$$`。这种类型的赋值使用“`:::=`”运算符。示例：

```makefile
var = first
OUT :::= $(var)
var = second
```
结果是，`OUT` 包含的文本是 `first` 。然而

```makefile
var = one$$two
OUT :::= $(var)
var = three$$four
```

结果是，`OUT` 包含的文本是 `one$$two` 。当变量被赋值时，该值被展开，因此结果是 `var` 的第一个值的展开“`one$two`”；然后在赋值完成之前重新转义该值，得到最终结果“`one$$two`”。

此后，变量`OUT`被视为递归变量，因此在使用时会重新展开。

这在功能上似乎与 `:=` 或 `::=` 运算符等效，但有一些区别：

首先，赋值后的变量是一个正常的递归变量；当您用 `+=` 向变量添加值时，右侧的值不会立即展开。如果您希望 `+=` 运算符立即展开右侧的值，则应使用`:=` 或 `::=`赋值。

其次，这些变量的效率略低于*简单展开变量*，因为它们在使用时确实需要重新展开，而不仅仅是复制。然而，由于所有变量引用都是转义的，这种展开只需取消转义值，就不会展开任何变量或执行任何函数。

下面是另一个例子：

```makefile
var = one$$two
OUT :::= $(var)
OUT += $(var)
var = three$$four
```

`OUT` 的结果是 `one$$two $(var)` ，当 `OUT` 被使用时，会被展开成 `one$two three$four` 。

这种类型的赋值相当于传统的 BSD *make* 中的 "`:=`" 运算符；正如您所看到的，它的工作原理与 GNU *make* "`:=`"运算符略有不同。`:::=` 运算符在 Issue 8 中添加到 POSIX 规范中，以提供可移植性。

（译者注：在再次使用 `:::=` 赋值时，会覆盖掉之前的定义。）

### 6.2.4 条件变量赋值

变量还有另一个赋值运算符“`?=`”。这被称为条件变量赋值运算符(*conditional variable assignment operator*)，因为它只有在变量尚未定义时才有效。此声明：

```makefile
FOO ?= bar
```
等价于(请参阅 [8.11 The origin Function](https://www.gnu.org/software/make/manual/make.html#Origin-Function))

```makefile
ifeq ($(origin FOO), undefined)
    FOO = bar
endif
```

注意，设置为空值的变量也算被定义了，`?=` 不会对该变量进行设置。

## 6.3 引用变量的高级功能
本节介绍了一些高级功能，您可以使用这些功能以更灵活的方式引用变量。

### 6.3.1 替换引用
*替换引用*(*substitution reference*) 使用您指定的更改去替换变量的值。它的形式为 `$(var:a=b)` 或 `${var:a=b}`，其含义是取变量 *var* 的值，将该值中的单词末尾的每个 *a* 替换为 *b*，并替换生成的字符串。

“在单词末尾”指的是要么 _a_ 之后紧接着空格，要么出现在 _var_ 值的最后，才能被替换。值中出现的其他 _a_ 不变。例如

``` makefile
foo := a.o b.o l.a c.o
bar := $(foo:.o=.c)
```

最终结果是将 `bar` 设置为 `a.c b.c l.a c.c`。（请参阅 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting)）

*替换引用*是 `patsubst` 展开函数的简写（请参阅 [8.2 Functions for String Substitution and Analysis](https://www.gnu.org/software/make/manual/make.html#Text-Functions)）：“`$(var:a=b)`”等效于“`$(patsubst %a,%b,var)`”。为了与 *make* 的其他实现兼容，我们提供了*替换引用*以及`patsubst`。

另一种类型的 *替换引用* 允许您使用 `patsubst` 函数的全部功能。它的形式与上面描述的“`$(var:a=b)`”相同，只是现在 *a* 必须包含一个 “`%`” 字符。这种情况相当于“`$(patsubst a,b,$(var))`”。有关 `patsubst` 函数的描述，请参阅 [8.2 Functions for String Substitution and Analysis](https://www.gnu.org/software/make/manual/make.html#Text-Functions)。例如

```makefile
foo := a.o b.o l.a c.o
bar := $(foo:%.o=%.c)
```

将 “`bar`” 设置为 “`a.c b.c l.a c.c`”. 

### 6.3.2 计算变量名称 

计算变量名是一个高级概念，在更复杂的 *makefile* 程序中非常有用。在简单的情况下，你不需要考虑它们，但它们可能非常有用。

（译者注：原文地址 [6.3.2 Computed Variable Names](https://www.gnu.org/software/make/manual/make.html#Computed-Names)。暂时不想翻译这段，不仅复杂而且有点儿多，建议直接写一个规则，然后在配方里用 `echo` 打印某个变量的值就得了，别自己计算了。）

## 6.4 变量获取值的方式

变量可以通过几种不同的方式获取值：

- 运行 *make* 时，可以指定替代值。请参见 [9.5 Overriding Variables](https://www.gnu.org/software/make/manual/make.html#Overriding)。
- 可以通过赋值（请参见 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting)）或逐字定义（请参见 [6.8 Defining Multi-Line Variables](https://www.gnu.org/software/make/manual/make.html#Multi_002dLine)）在 *makefile* 中指定值。
- 您可以使用`let`函数（请参阅 [8.5 The let Function](https://www.gnu.org/software/make/manual/make.html#Let-Function)）或 `foreach` 函数（请参见 [8.6 The foreach Function](https://www.gnu.org/software/make/manual/make.html#Foreach-Function)）指定一个短期值(short-lived value)。
- 环境中的变量变成了 *make* 变量。请参见 [6.10 Variables from the Environment](https://www.gnu.org/software/make/manual/make.html#Environment)。
- 自动变量可根据规则不同而赋予新值。每一个自动变量都有单一的常规用途。请参见 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)。
- 有几个变量具有恒定的初始值。请参见 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)。

## 6.5 设置变量

若要从 *makefile* 中设置变量，以变量名开头、后跟赋值运算符“`=`”、“`:=`”、“`::=`”或“`:::=`”之一的一行。运算符后面的任何内容和行上的任何初始空白都将成为值。例如

```makefile
objects = main.o foo.o bar.o utils.o
```
定义一个名为 `objects` 的变量以包含值“`main.o foo.o bar.o utils.o`”。**变量名周围和紧跟在 `=` 后面的空白字符被忽略**。（译者注，这里建议看一下我在 8.1 小节末尾的注释）

用 "`=`" 定义的变量是*递归展开变量*。用"`:=`"或 "`::=`" 定义的变量是*简单展开变量*；这些定义可以包含变量引用，这些引用将在定义之前展开。用 "`:::=`" 定义的变量是*立即展开变量*。不同的赋值操作符在 [6.2 The Two Flavors of Variables](https://www.gnu.org/software/make/manual/make.html#Flavors) 中有所描述。

变量名可能包含函数和变量引用，当读取该行以查找要使用的实际变量名时，这些引用会展开。

除了计算机上的内存量之外，变量值的长度没有限制。为了易读性，您可以将变量值拆分为多个物理行（请参阅 [3.1.1 Splitting Long Lines](https://www.gnu.org/software/make/manual/make.html#Splitting-Lines)）。

大多数您从未设置过的变量名称，都被认为具有空字符串作为值。一些变量具有不为空的内置初始值，但您可以通过常规的方式设置它们（请参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)）。一些特殊变量会自动为每个规则设置一个新值；这些被称为自动变量（请参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)）。

如果您希望仅在该变量尚未被设置时才将其设置为某值，则可以使用速记运算符 "`?=`" 而不是 "`=`"。变量 "`FOO`" 的这两个设置是相同的（请参阅 [8.11 The origin Function](https://www.gnu.org/software/make/manual/make.html#Origin-Function)）：

```makefile
FOO ?= bar
```
和
```makefile
ifeq ($(origin FOO), undefined)
FOO = bar
endif
```

shell 赋值操作符 "`!=`" 可用于执行 shell 脚本并为其输出设置变量。此操作符首先计算右侧，然后将结果传递给 shell 执行。如果执行结果以换行符结尾，则删除该换行符；所有其他换行符都用空格替换。然后将生成的字符串放入给定名称的*递归展开变量*中。例如：

```makefile
hash != printf '\043'
file_list != find . -name '*.c'
```

如果执行的结果可能会产生一个 `$`，并且您不打算将接下来的内容解释为 *make* 变量或函数引用，那么您必须在执行中“将每个 `$` 替换为 `$$` ”作为执行的一部分。或者，您可以将一个 *简单展开变量* 设置为调用 `shell` 函数运行程序的结果。请参阅 [8.14 The shell Function](https://www.gnu.org/software/make/manual/make.html#Shell-Function)。例如：

```makefile
hash := $(shell printf '\043')
var := $(shell find . -name "*.c")
```

与 `shell` 函数一样，刚刚调用的 shell 脚本的退出状态存储在 `.SHELLSTATUS ` 变量中。

## 6.6 将更多文本附加到变量

通常，向已定义的变量的值添加更多文本很有用。您可以使用包含 "`+=`" 的行来执行此操作，如下：

```makefile
objects += another.o
```

这将获取变量 *objects* 的值，并将文本 “*another.o*” 添加到其中（如果变量中之前已经有值，则会在新添加的内容前面加一个空格）。因此：

```makefile
objects = main.o foo.o bar.o utils.o
objects += another.o
```

会将 *objets* 设置为 "*main.o foo.o bar.o utils.o another.o*"

使用 "`+=`" 类似于：

```makefile
objects = main.o foo.o bar.o utils.o
objects := $(objects) another.o
```

但是当您使用更复杂的值时，不同的方式变得很重要。

当之前未定义相关变量时，"`+=`" 的行为就像正常的  "`=`"：它定义了一个 *递归展开变量*。然而，当存在以前的定义时，"`+=`" 的确切作用取决于您最初定义的变量的风格。有关变量的两种风格的解释，请参阅 [6.2 The Two Flavors of Variables](https://www.gnu.org/software/make/manual/make.html#Flavors)。

当你用 "`+=`" 向变量的值添加内容时, *make* 表现的基本上就像你在变量的初始定义中包含了额外的文本一样。如果你首先用 "`:=`" 或者 "`::=`" 定义它, 使它成为一个*简单展开变量*, "`+=`" 添加到这个简单展开的定义中，并在将新文本附加到旧值之前展开它，就像"`:=`" 一样（请参阅 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting)，以获得 "`:=`" 或者 "`::=`" 的完整解释). 事实上，

```makefile
variable := value
variable += more
```

完全等同于：

```makefile
variable := value
variable := $(variable) more
```

大致相当于：

```makefile
temp = value
variable = $(temp) more
```

当然，它从来没有定义一个名为 *temp* 的变量。当变量的旧值包含变量引用时，这一点就很重要了。举个常见的例子：

```makefile
CFLAGS = $(includes) -O
…
CFLAGS += -pg # enable profiling
```

第一行定义了 `CFLAGS` 变量，并引用了另一个变量 `include`。（`CFLAGS` 用于 C 编译规则使用；请参阅 [10.2 Catalogue of Built-In Rules](https://www.gnu.org/software/make/manual/make.html#Catalogue-of-Rules)。）使用 "`=`" 定义使 `CFLAGS` 成为*递归展开变量*，这意味着当处理 `CFLAGS` 的定义时，"`$(includes) -O`" 不会被展开。因此，目前还不需要定义 `include` 来使其值生效，它只需要在引用 `CFLAGS` 之前定义。如果我们试图不使用 "`+=`" 向 `CFLAGS` 的值添加内容, 我们可能会这样做：

```makefile
CFLAGS := $(CFLAGS) -pg # enable profiling
```

这很接近，但不是我们想要的。使用 "`:=`" 将 `CFLAGS` 重新定义为 *简单展开变量*；这意味着 *make* 在设置变量之前展开文本 "`$(CFLAGS) -pg`"。如果尚未定义 `includes`，我们得到 "` -O -pg`"，且之后对 `includes` 的定义将不起作用。相反，通过使用 "`+=`"，我们将 `CFLAGS` 设置为非展开的值 "`$(includes) -O -pg`"。因此，我们保留了对 `includes` 的引用，因此如果该 `includes` 变量在稍后定义，像 "`$(CFLAGS)`" 这样的引用仍然使用它的值。

## 6.7 `override` 指令

如果变量已使用命令参数设置（请参阅 [9.5 Overriding Variables](https://www.gnu.org/software/make/manual/make.html#Overriding)），则 *makefile* 中的普通赋值将被忽略。尽管一个变量已经是使用命令参数设置的，您还是想在 *makefile* 中设置它，那么您可以使用 `override` 指令

```makefile
override variable = value
```
或
```makefile
override variable := value
```

要将更多文本附加到在命令行上定义的变量（请参阅 [6.6 Appending More Text to Variables](https://www.gnu.org/software/make/manual/make.html#Appending)），请使用：
```makefile
override variable += more text
```

标有 `override` 标志的变量赋值比其它赋值具有更高的优先级（除非也是 `override` 赋值）。未标记 `override` 的后续赋值或向此变量的附加将被忽略。

`override` 指令不是为了激化 *makefile* 和命令参数之间的战争中而发明的，而是为了让您可以更改和添加用户使用命令参数指定的值。

例如，假设您在运行C编译器时总是想要 "`-g`" 选项，但您希望像往常一样允许用户使用命令参数指定其他开关。则可以使用 `override` 指令：

```makefile
override CFLAGS += -g
```

您还可以将 `override` 指令与 `define` 指令一起使用。这是按照您的预期完成的：

``` makefile
override define foo =
bar
endef
```

请参阅 [6.8 Defining Multi-Line Variables](https://www.gnu.org/software/make/manual/make.html#Multi_002dLine)

## 6.8 定义多行变量
另一种设置变量值的方法是使用 `define` 指令。该指令有一种不寻常的语法，允许在值中包含换行符，这便于定义命令的预制序列（请参阅 [5.8 Defining Canned Recipes](https://www.gnu.org/software/make/manual/make.html#Canned-Recipes)），以及与 `eval` 一起使用的 *makefile* 语法部分（请参阅 [8.10 The eval Function](https://www.gnu.org/software/make/manual/make.html#Eval-Function)）。

在同一行中，`define` 指令后面跟着被定义变量的名称和一个（可选的）赋值操作符，然后就没有更多内容了。给定变量的值出现在后续行中。值的末尾由仅包含单词 `endf` 的行标记。

除了这种语法上的差异，`define` 指令就像任何其他变量定义一样工作。变量名可能包含函数和变量引用，当读取指令以查找要使用的实际变量名时，它们会被展开。

`endf`之前的最后一个换行符不包含在值中；如果您希望您的值包含尾随换行符，则必须包含一个空行。例如，为了定义包含换行符的变量，您必须使用两个空行，而不是一个：

```makefile
define newline


endef
```

如果您愿意，您可以省略变量赋值操作符。如果省略，*make* 会假定它是 "`=`" 并创建一个*递归展开*变量（请参阅 [6.2 The Two Flavors of Variables](https://www.gnu.org/software/make/manual/make.html#Flavors)）。使用 "`+=`" 运算符时，该值会像任何其他附加操作一样附加到前一个值：用一个空格分隔旧值和新值。

您可以嵌套 `define` 指令：*make* 将跟踪嵌套指令，如果它们没有全部正确地用 `endf` 关闭，则会报告错误。请注意，以配方前缀字符开头的行被视为配方的一部分，因此出现在该行的任何 `define` 或 `endf` 字符串都不会被视为 *make* 指令。

```makefile
define two-lines
echo foo
echo $(bar)
endef
```

当在配方中使用时，前面的示例在功能上等价于：

```makefile
two-lines = echo foo; echo $(bar)
```

因为用分号分隔的两个命令的行为很像两个单独的 shell 命令。但是，请注意，使用两个单独行意味着 *make* 将调用 shell 两次，为每一行运行一个独立的子shell。请参阅 [5.3 Recipe Execution](https://www.gnu.org/software/make/manual/make.html#Execution)。

如果您希望使用 `define` 创建的变量定义优先于命令行变量定义，您可以将 `overrid` 指令与 `define` 一起使用：

```makefile
override define two-lines =
foo
$(bar)
endef
```

请参阅 [6.7 The override Directive](https://www.gnu.org/software/make/manual/make.html#Override-Directive)

## 6.9 取消变量定义

如果您想清除一个变量，将其值设置为空通常就足够了。无论变量是否被设置，展开这样的变量将产生相同的结果（空字符串）。但是，如果您使用的是 `flavor` 函数（[8.12 The flavor Function](https://www.gnu.org/software/make/manual/make.html#Flavor-Function)）和 `origin` 函数（[8.11 The origin Function](https://www.gnu.org/software/make/manual/make.html#Origin-Function)）中，一个值被设置为空或是没有被设置，是有区别的。在这种情况下，您可能希望使用 `undefine` 指令使变量看起来好像从未设置过一样。例如：

```makefile
foo := foo
bar = bar

undefine foo
undefine bar

$(info $(origin foo))
$(info $(flavor bar))
```

此示例中，两个变量的打印都是“未定义(undefined)”。

如果要取消定义命令行变量定义，可以将 `overrid` 指令与 `undefined` 一起使用，类似于对变量定义的操作方式：

```makefile
override undefine CFLAGS
```

## 6.10 来自环境的变量
*make* 中的变量可以来自运行 *make* 的环境。*make* 启动时看到的每个环境变量都会转换为具有相同名称和值的 *make* 变量。但是，*makefile* 中的显式赋值或使用命令参数会覆盖环境。（如果指定了 “`-e`” 标志，则环境中的值将覆盖 *makefile* 中的分配。请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/html_node/Options-Summary.html)。但实际中不建议这样做。）

因此，通过在环境中设置变量 `CFLAGS` ，可以使大多数 *makefile* 中的所有C编译使用您喜欢的编译器开关。这对于具有标准或传统含义的变量是安全的，因为您知道没有 *makefile* 会将它们用于其他事情。（请注意，这并不完全可靠；一些 *makefile* 显式设置 `CFLAGS`，因此不受环境中的值的影响。）

当 *make* 运行一个配方时，*makefile* 中定义的一些变量会被放入 *make* 调用的每个命令的环境中。默认情况下，只有来自 *make* 环境或在其命令行上设置的变量才会被放置到命令的环境中。您可以使用 `export` 指令来传递其他变量。有关完整的详细信息，请参阅 [5.7.2 Communicating Variables to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Variables_002fRecursion.html)。

**不建议使用环境中的其他变量**。*makefile* 的功能依赖于在其控制范围之外设置的环境变量是不明智的，因为这会导致不同的用户从同一个 *makefile* 中获得不同的结果。这违背了大多数 *makefile* 的全部目的。

变量 `SHELL` 尤其可能出现这种问题，它通常存在于环境中，用于指定用户对交互式 *shell* 的选择。这种选择会影响 “*make*”，这是非常不可取的；因此，“*make*” 以一种特殊的方式处理 “`SHELL`” 环境变量；请参阅 [5.3.2 Choosing the Shell ](https://www.gnu.org/software/make/manual/html_node/Choosing-the-Shell.html)。


## 6.11 特定于目标的变量值

*make* 中的变量值通常是全局的；也就是说，无论在哪里评估它们，它们都是相同的（当然，除非它们被重置）。例外情况是用 `let` 函数（参见 [The let Function](https://www.gnu.org/software/make/manual/make.html#Let-Function)）或foreach函数（参见 [The foreach Function](https://www.gnu.org/software/make/manual/make.html#Foreach-Function)）定义的变量，以及自动变量（参见 [Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)）。

另一个例外是 *特定于目标的变量值*(*target-specific variable values*)。此功能允许您根据 *make* 当前正在构建的目标为同一变量定义不同的值。与自动变量一样，这些值仅在目标配方的上下文中可用（以及在其他特定于目标的赋值中）。

设置一个特定于目标的变量值，如下所示：

```makefile
target … : variable-assignment
```

可以使用特殊关键字 `export`, `unexport`, `override`, 或 `private` 中的一个或者所有作为特定于目标的变量赋值的前缀。这些仅将其正常行为应用于变量的此实例。

多个目标值分别为目标列表的每个成员创建一个特定于目标的变量值。

变量赋值可以是任何有效的赋值形式；递归 ("`=`"), 简单 ("`:=`" 或 "`::=`"), 立即 ("`:::=`"), 附加 ("`+=`"), 或条件 ("`?=`")。变量赋值中出现的所有变量都在目标的上下文中进行评估：因此，任何先前定义的特定于目标的变量值都将生效。请注意，这个变量实际上不同于任何“全局”值：两个变量不必具有相同的风格（递归与简单）。

特定于目标的变量与任何其他 *makefile* 变量具有相同的优先级。命令行上提供的变量（如果 "`-e`" 选项有效，则在环境中提供）将优先。指定 `override` 指令将允许首选特定于目标的变量值。

特定于目标的变量还有一个特殊特性：当您定义特定于目标的变量时，该变量值也对该目标的所有先决条件、以及先决条件的所有先决条件等有效（除非这些先决条件用它们自己的特定于目标的变量值覆盖该变量）。例如，这样的语句：

```makefile
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
```

将在 *prog* 的配方中将 `CFLAGS` 设置为 "`-g`"，但在创建 *prog.o*、*foo.o* 和 *bar.o* 的配方、以及任何创建其先决条件的配方中也将 `CFLAGS` 设置为 "`-g`"。

请注意，给定的先决条件最多只能在每次调用 *make* 时构建一次。如果同一个文件是多个目标的先决条件，并且每个目标对同一目标特定变量都有不同的值，那么**构建的第一个目标将导致该先决条件被构建，先决条件将从第一个目标继承目标特定值。它将忽略任何其他目标的目标特定值。**

## 6.12 特定于模式的变量值
除了特定于目标的变量值（请参阅 [6.11 Target-specific Variable Values](https://www.gnu.org/software/make/manual/make.html#Target_002dspecific)）之外，GNU *make* 还支持特定于模式(pattern-specific)的变量值。在这种形式中，变量是为与指定模式匹配的所有目标定义的。

设置一个特定于模式的变量值，如下所示：

``` makefile
pattern … : variable-assignment
```

其中 `pattern` 是 `%-pattern`。与特定于目标的变量值一样，多个模式值分别为每个模式创建特定于模式的变量值。变量赋值可以是任何有效的赋值形式。除非指定了 `override`，否则任何命令行变量设置都将优先。

例如：

```makefile
%.o : CFLAGS = -O
```
将会针对与模式 `%.o` 匹配的目标进行对`CFLAGS`赋值 "`-O`"。

如果目标匹配多个模式，则匹配了更长词干的特定模式匹配会被首先解释。这会导致更具体的变量优先于更通用的变量。示例：

```makefile
%.o: %.c
        $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@

lib/%.o: CFLAGS := -fPIC -g
%.o: CFLAGS := -g

all: foo.o lib/bar.o
```

在这个例子中，`CFLAGS` 变量的第一个定义将用于更新 `lib/bar.o`，即使第二个定义也适用于这个目标。相同词干长度匹配的特定模式变量则按照它们在 *makefile* 中定义的顺序被考虑。

“特定于模式的变量” 会在 “为该目标所显式定义的特定于目标的变量” 之后、在 “为父目标定义的特定于目标的变量” 之前被搜索。

## 6.13 抑制继承

如前几节所述，*make* 变量会被先决条件继承。此功能允许您根据导致重建的目标修改先决条件的行为。例如，您可以在 *debug* 目标上设置特定于目标的变量，然后执行 "`make debug`" 将导致该变量被 *debug* 的所有先决条件继承，而仅运行 "`make all`"（举例）将没有该赋值。

但是，有时您可能不希望一个变量被继承。对于这些情况，*make* 提供了 `private` 修饰符。尽管这个修饰符可以与任何变量赋值一起使用，但它对特定于目标或模式的变量最有意义。任何标记为 `private` 的变量都将对其本地目标可见，但不会被该目标的先决条件继承。标记为 `private` 的全局变量将在全局范围内可见，但不会被任何目标继承，因此在任何配方中都不可见。

例如，考虑以下 makefile
```makefile
EXTRA_CFLAGS =

prog: private EXTRA_CFLAGS = -L/usr/local/lib
prog: a.o b.o
```

由于 `private` 修饰符，*a.o* 和 *b.o* 不会从目标 *prog* 继承 `EXTRA_CFLAGS` 变量赋值。

## 6.14 其它特殊变量

GNU *make* 支持一些具有特殊属性的变量。

- `MAKEFILE_LIST`

包含由 *make* 解析的每个 *makefile* 的名称，按照顺序进行解析。该名称在 *make* 开始解析 *makefile* 之前附加。因此，如果 *makefile* 做的第一件事是检查这个变量中的最后一个单词，它将是当前 *makefile* 的名称。但是，一旦当前 *makefile* 使用了 `include`，最后一个单词将是刚刚包含的 *makefile*。
如果名为 *Makefile* 的 *makefile* 具有以下内容：

```makefile
name1 := $(lastword $(MAKEFILE_LIST))

include inc.mk

name2 := $(lastword $(MAKEFILE_LIST))

all:
    @echo name1 = $(name1)
    @echo name2 = $(name2)
```

然后你会期望看到这个输出：

```sh
name1 = Makefile
name2 = inc.mk
```

- `.DEFAULT_GOAL`

如果在命令行上未指定目标，则设置要使用的默认目标（请参阅 [9.2 Arguments to Specify the Goals](https://www.gnu.org/software/make/manual/make.html#Goals)）。`.DEFAULT_GOAL` 变量允许您发现当前默认目标、通过清除其值重新启动默认目的(goal)选择算法或显式设置默认目标。以下示例说明了这些情况：

```makefile
# Query the default goal.
ifeq ($(.DEFAULT_GOAL),)
  $(warning no default goal is set)
endif

.PHONY: foo
foo: ; @echo $@

$(warning default goal is $(.DEFAULT_GOAL))

# Reset the default goal.
.DEFAULT_GOAL :=

.PHONY: bar
bar: ; @echo $@

$(warning default goal is $(.DEFAULT_GOAL))

# Set our own.
.DEFAULT_GOAL := foo
```

这个 *makefile* 打印：

```sh
no default goal is set
default goal is foo
default goal is bar
foo
```

请注意，为 `.DEFAULT_GOAL` 分配多个目标名称是无效的，将导致错误。

- `MAKE_RESTARTS`

仅当 *make* 实例重新启动时才设置此变量（请参阅 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/make.html#Remaking-Makefiles)）：它将包含此实例重新启动的次数。请注意，这与递归（由 `MAKELEVEL` 变量计算）不同。您不应该设置、修改或导出此变量。

- `MAKE_TERMOUT`<br>`MAKE_TERMERR`

当 *make* 启动时，它将检查 *stdout* 和 *stderr* 是否会在终端上显示它们的输出。如果是这样，它将分别设置 `MAKE_TERMOUT` 和 `MAKE_TERMERR` 到终端设备的名称（如果无法确定，则为true）。如果设置，这些变量将被标记为导出。这些变量将不会被 *make* 更改，如果已经设置，也不会被修改。

可以使用这些值（特别是与输出同步（参见 [5.4.2 Output During Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel-Output)）结合使用）来确定 *make* 本身是否正在向终端写入；例如，可以测试它们以决定是否强制配方命令生成彩色输出。

如果您调用 sub-*make* 并重定向其 *stdou*t 或 *stderr*，如果您的 *makefile* 依赖它们，您也有责任重置或取消导出这些变量。

- `.RECIPEPREFIX`

此变量值的第一个字符用作 *make* 假设正在引入配方行的字符。如果变量为空（默认情况下），则该字符是标准制表符。例如，这是一个有效的 *makefile*：

```makefile
.RECIPEPREFIX = >
all:
> @echo Hello, world
```

`.RECIPEPREFIX` 的值可以被多次更改；一旦设置，它对所有解析的规则保持有效，直到它被再次修改。

- `.VARIABLES`

展开后是到目前为止定义的所有全局变量的名称列表。这包括具有空值的变量以及内置变量（请参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)），但不包括仅在特定目标上下文中定义的任何变量。请注意，您分配给此变量的任何值都将被忽略；它将始终返回其特殊值。

- `.FEATURES`

展开到此版本 *make* 支持的特殊功能列表。可能的值包括但不限于：
    "archives"

    "check-symlink"

    "else-if"

    "extra-prereqs"

    "grouped-target"

    "guile"

    "jobserver"

    "jobserver-fifo"

    "load"

    "notintermediate"

    "oneshell"

    "order-only"

    "output-sync"

    "second-expansion"

    "shell-export"

    "shortest-stem"

    "target-specific"

    "undefine"

- `.INCLUDE_DIRS`

展开后是 *make* 搜索包含 *makefile* 的目录列表（请参阅 [3.3 Including Other Makefiles](https://www.gnu.org/software/make/manual/make.html#Include)）。请注意，修改此变量的值不会更改要搜索的目录列表。

- `.EXTRA_PREREQS`

这个变量中的每个单词都是一个新的、被添加到预定的目标中的先决条件。这些先决条件与普通先决条件的不同之处在于它们不会出现在任何自动变量中（请参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)）。这允许了定义先决条件而不影响配方的不影响配方的情况。

考虑一个用于链接程序的规则：

```makefile
myprog: myprog.o file1.o file2.o
    $(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^ $(LDLIBS)
```

现在假设您想增强此 *makefile* 以确保对编译器的更新会导致程序重新链接。您可以将编译器添加为先决条件，但您必须确保它不会作为参数传递给链接命令。您需要这样的东西：

```makefile
myprog: myprog.o file1.o file2.o $(CC)
    $(CC) $(CFLAGS) $(LDFLAGS) -o $@ \
        $(filter-out $(CC),$^) $(LDLIBS)
```

然后考虑到有多个额外的先决条件：它们都必须被过滤掉。使用 `.EXTRA_PREREQS` 和特定于目标的变量提供了一个更简单的解决方案：

```makefile
myprog: myprog.o file1.o file2.o
    $(CC) $(CFLAGS) $(LDFLAGS) -o $@ $^ $(LDLIBS)
myprog: .EXTRA_PREREQS = $(CC)
```

如果您想向无法轻松修改的 *makefile* 添加先决条件，此功能也很有用：您可以创建一个新文件，例如 *extra.mk*：

```makefile
myprog: .EXTRA_PREREQS = $(CC)
```

然后调用 `make -f extra.mk -f Makefile`。

全局设置 `.EXTRA_PREREQS` 将导致这些先决条件被添加到所有目标（这些目标本身没有用特定于目标的值覆盖它）。注意 *make* 足够聪明，不会添加 `.EXTRA_PREREQS` 中列出的先决条件作为自身的先决条件。


# 7 *Makefile* 的条件句部分
根据变量的值，条件(conditional)指令会导致 *makefile* 的一部分被遵守或忽略。条件可以将一个变量的值与另一个变量进行比较，或者将变量的值与常量字符串做比较。条件控制 *make* 在 *makefile* 中实际“看到”的内容，因此在执行时不能用于控制配方。

## 7.1 条件句举例
下面的条件表达式示例告诉 *make*，如果`CC`变量为“`gcc`”则使用一组库，否则使用另一组库。它的工作原理是控制两个配方行中的哪一个将用于规则。结果是，“`CC=gcc`”作为一个参数，不仅可以更改使用的编译器，还可以更改链接的库。

```makefile
libs_for_gcc = -lgnu
normal_libs =

foo: $(objects)
ifeq ($(CC),gcc)
    $(CC) -o foo $(objects) $(libs_for_gcc)
else
    $(CC) -o foo $(objects) $(normal_libs)
endif
```

这个条件句使用三个指令：`ifeq`、`else` 和 `endif`。

`ifeq`指令开启了条件句，并指定条件。它包含两个参数，用逗号分隔，并用括号括起来。对两个参数执行变量替换，然后对它们进行比较。如果两个参数匹配，则会遵守 `ifeq` 后面的 *makefile* 行；否则它们将被忽略。

如果前一个条件失败，`else` 指令将导致遵守以下行。在上面的例子中，这意味着每当不使用第一备选方案时，就使用第二备选方案链接命令。在条件句中是否包含 `else` 是可选的。

`endif`指令结束条件句。每个条件句都必须以`endif`结尾。*makefile* 中的非条件句文本可以列写在其后。

如本例所示，条件语句在文本级别工作：根据条件，条件语句的行被视为 *makefile* 的一部分，或者被忽略。这就是为什么 *makefile* 中较大的语法单元（如规则）可能会跨越条件的开头或结尾。

当变量`CC`的值为“`gcc`”时，上面的示例具有以下效果：
```makefile
foo: $(objects)
    $(CC) -o foo $(objects) $(libs_for_gcc)
```

当变量`CC`具有任何其他值时，效果如下：
```makefile
foo: $(objects)
    $(CC) -o foo $(objects) $(normal_libs)
```
可以通过另一种方式，如通过条件句变量赋值，然后非条件地使用变量，获得等效结果：

```makefile
libs_for_gcc = -lgnu
normal_libs =

ifeq ($(CC),gcc)
  libs=$(libs_for_gcc)
else
  libs=$(normal_libs)
endif

foo: $(objects)
    $(CC) -o foo $(objects) $(libs)
```

## 7.2 条件句语法

不带`else`的简单条件的语法如下：
```makefile
conditional-directive
text-if-true
endif
```

`text-if-true` 可以是任意一行文本，如果条件为 true，则视为 *makefile* 的一部分。如果条件为 false，则不使用任何文本。

复杂条件的语法如下：

```makefile
conditional-directive
text-if-true
else
text-if-false
endif
```
或
```makefile
conditional-directive-one
text-if-one-is-true
else conditional-directive-two
text-if-two-is-true
else
text-if-one-and-two-are-false
endif
```

根据需要，可以有尽可能多的“`else`条件指令”子句。一旦给定条件为 true，则使用 `text-if-true`，而不使用其他子句；如果没有条件为 true，则使用 `text-if-false`。`text-if-true` 和 `text-if-false`可以是任意数量的文本行（译者注，如果您习惯使用 C 语言，那么需要注意与 C 语言不同的是，在 *makefile* 中的条件句不需要括号将其包围）。

无论条件指令是简单的还是复杂的，在`else`之后或不是之后，条件指令的语法都是相同的。有四种不同的指令用于测试不同的条件。以下是它们的表格：

``` makefile
ifeq (arg1, arg2)
ifeq 'arg1' 'arg2'
ifeq "arg1" "arg2"
ifeq "arg1" 'arg2'
ifeq 'arg1' "arg2"
```

展开 `arg1` 和 `arg2` 中的所有变量引用并进行比较。如果它们完全相同，则 `text-if-true` 有效；否则，`text-if-false`（如果有的话）有效。

通常，您希望测试变量是否具有非空值。当值由变量和函数的复杂展开产生时，您认为为空的展开实际上可能包含空白字符，因此不会被视为空。但是，您可以使用`strip`函数（请参阅 [8.2 Functions for String Substitution and Analysis](https://www.gnu.org/software/make/manual/make.html#Text-Functions)）来避免将空白字符（whitespace）解释为非空值。例如：

```makefile
ifeq ($(strip $(foo)),)
text-if-empty
endif
```
即使`$(foo)`的展开包含空白字符，也将评估`text-if-empty`（译者注，原文是 "will evaluate text-if-empty"，直译是“评估”，但我觉得原文想表达的是会执行`text-if-empty`）。

```makefile
ifneq (arg1, arg2)
ifneq 'arg1' 'arg2'
ifneq "arg1" "arg2"
ifneq "arg1" 'arg2'
ifneq 'arg1' "arg2"
```
展开 `arg1` 和 `arg2` 中的所有变量引用并比较它们。如果不相同，则 `text-if-true` 有效；否则，`text-if-false`（如果有的话）有效。

```makefile
ifdef variable-name
```

`ifdef` 将变量名称作为其参数，而不是对变量的引用。如果该变量的值具有非空值，则`text-if-true`有效；否则，`text-if-false`（如果有的话）有效。从未定义过的变量的值为空。文本`variable-name`是展开的，因此它可以是一个可以展开到变量名称的变量或函数。例如

```makefile
bar = true
foo = bar
ifdef $(foo)
frobozz = yes
endif
```

变量引用 `$(foo)` 被展开，展开的结果是`bar`，它被认为是变量的名称。变量`bar`不会被展开，但会检查其值以确定其是否为非空。

请注意，`ifdef` 只测试变量是否有值。它不会展开变量以查看该值是否为非空(译者注，注意：`variable-name` 仅会被展开一次，即使展开后是变量或是函数，也不会再被展开)。因此，使用`ifdef`的测试对于除 `foo =` 以外的所有定义都返回 true。要测试空值，请使用 `ifeq ($(foo),)`，例如

```makefile
bar =
foo = $(bar)
ifdef foo
frobozz = yes
else
frobozz = no
endif
```

将 `frobozz` 设置为 `yes`，而

```makefile
foo =
ifdef foo
frobozz = yes
else
frobozz = no
endif
```

将 `frobozz` 设置为 `no`

```makefile
ifndef variable-name
```

如果变量 `variable-name` 的值为空，则`text-if-true`有效；否则，`text-if-false`（如果有的话）有效。`variable-name` 的展开和测试规则与`ifdef`指令相同。

条件指令行的开头允许并忽略额外的空格，但不允许使用制表符。如果该行以制表符开头，它将被视为规则配方(recipe for a rule)的一部分。除此之外，除了在指令名称或参数中之外，可以在任何位置插入额外的空格或制表符，且不会产生任何效果。以“#”开头的注释可以会出现在行的末尾。

在条件中起作用的另外两个指令是`else`和`endif`。这些指令中的每一个都被写成一个单词，没有参数。允许并忽略行开头的额外空格，以及末尾的空格或制表符。以“#”开头的注释可以会出现在行的末尾。

条件语句会影响 make 的 *makefile* 的哪些行被使用。如果条件为true，*make* 将读取 `text-if-true` 作为 *makefile* 的一部分；如果条件为 false，make 将完全忽略这些行。因此，*make* 根据 *makefile* 的语法单元，可以将例如规则的这部分安全地在条件语法的开头或结尾进行拆分（译者注，我理解的意思是“可以将条件语句穿插在规则之中”）。

*make* 会在读取 *Makefile* 文件时评估条件语句，而自动变量是在 *reipce* 被执行后才被定义，所以 **条件语句中不允许使用自动变量**（请参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)）。

为了防止出现无法容忍的混乱，不允许在一个 *makefile* 中启动条件语句并在另一个 *makefile* 中结束它。但是，如果不尝试在包含的文件中终止条件语句，则可以在条件中编写 `include` 指令。

## 7.3 用于测试 Flag 的条件句
您可以编写一个条件语句，通过将变量 `MAKEFLAGS` 与 `findstring` 函数（请参阅 [8.2 Functions for String Substitution and Analysis](https://www.gnu.org/software/make/manual/make.html#Text-Functions)）一起使用，来测试 *make* 命令标志（如“-t”）。当 `touch` 不足以使文件显示为最新时，这很有用。

回调(recall) `MAKEFLAGS` 会将所有单字母选项（如“-t”）放入第一个单词中，如果没有给出单字母选项，则该单词将为空。要处理此问题，在开头添加一个值以确保有一个词是很有帮助的：例如“`-$(MAKEFLAGS)`”。

`findstring` 函数确定一个字符串是否显示为另一个字符串的子字符串。如果要测试“`-t`”标志，请将“t”用作第一个字符串，`MAKEFLAGS`的第一个单词用作另一个字符串。

例如，以下是如何安排使用“`ranlib -t`”来完成对存档文件的最新标记：

```makefile
archive.a: …
ifneq (,$(findstring t,$(firstword -$(MAKEFLAGS))))
    +touch archive.a
    +ranlib -t archive.a
else
    ranlib archive.a
endif
```

前缀“`+`”将这些配方行标记为“递归”，这样即使使用了“`-t`”标志，它们也会被执行。请参阅 [5.7 Recursive Use of make](https://www.gnu.org/software/make/manual/make.html#Recursion)。


# 8 用于转换文本的函数

函数允许您在 *makefile* 中进行文本处理，以计算要操作的文件或要在配方中使用的命令。您在函数调用中使用一个函数，在该调用中您提供函数名称和一些文本（参数）供函数操作。函数处理的结果会在调用处被替换到 *makefile* 中，就像变量被替换一样。

## 8.1 调用函数的语法

函数调用类似于变量引用，任何可以出现变量引用的地方也可以出现函数调用，并且使用与变量引用相同的规则对其进行展开。函数调用看起来像这样

```makefile
$(function arguments)
```
或者像这样
```makefile
${function arguments}
```

在这里 `function` 是一个函数名称；可以是 make 中的简短列表之一。您还可以使用内置函数 `call` 创建自己的函数。

参数(arguments)是函数的参数。它们**与函数名之间用一个或多个空格或制表符分隔**，如果有**多个参数，则用逗号分隔**。这些空格和逗号不是参数值的一部分。用于包围函数调用的分隔符，无论是括号还是大括号，只能以匹配对的形式出现在参数中；另一种类型的分隔符可能单独出现。如果参数本身包含其他函数调用或变量引用，最好对所有引用使用相同类型的分隔符；写“ `‘$(subst a,b,$(x))` ”，而不是“ `$(subst a,b,${x})` ”。这是因为它更清晰，并且因为只有一种类型的分隔符匹配以找到引用的结尾。

除非下面另有说明，否则每个参数都会在调用函数之前展开。替换是按照参数出现的顺序完成的。

**特殊字符**
当使用特殊的字符作为函数参数时，您可能需要隐藏它们。GNU *make* 不支持使用反斜杠或其他转义序列转义字符；（*疑问，不支持吗？估计指的是不支持使用反斜杠来转移这些特殊字符吧*）；但是，由于参数在展开之前会被拆分，因此您可以通过将它们放入变量来隐藏它们。
需要隐藏的特殊字符包含：
- 逗号
- 第一个参数中的初始空白
- 不匹配的小括号或大括号
- 一个您不想让它匹配的开圆括号或开大括号

例如，您可以定义变量 `comma` 和 `space`，其值是单独的逗号和空格字符，然后在需要这些字符的地方替换这些变量，如下所示：

```makefile
comma:= ,
empty:=
space:= $(empty) $(empty)
foo:= a b c
bar:= $(subst $(space),$(comma),$(foo))
# bar is now ‘a,b,c’.
```
这里 `subst` 函数将 `foo` 的值中的每个空格用逗号替换。
（译者注，在 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting) 中描述到 **变量名周围和紧跟在 `=` 后面的空白字符被忽略**

```makefile
space_two =    
empty:=
space:= $(empty) $(empty)

text_space_two = print$(space_two)information
text_space = print$(space)information

debug:
	@echo $(text_space_two)
	@echo $(text_space)
```

的执行结果是
```
printinformation
print information
```

即， 即使在`space_two`变量的`=` 符号后输入了两个（或者三个、四个...）空白字符，它的变量值也是空的。）

## 8.2 字符串替换和分析的函数

以下是一些对字符串进行操作的函数：

- `$(subst from,to,text)`
	将 `text` 中出现的每一个 `from` 替换成 `to`
	
- `$(patsubst pattern,replacement,text)`
    查找 *text* 中与 *pattern* 匹配的以空格分隔的单词，并用 *replacement* 替换它们。这里的 *pattern* 可能包含一个充当通配符的“%”，以匹配不限字符及数量的单个单词。如果 *replacement* 也包含一个“%”，则“%”将替换为与 *pattern* 中的“%”匹配的文本。与 *pattern* 不匹配的单词将在输出时保持不变。只有 *pattern* 和 *replacement* 中的第一个“%”被这样处理；任何后续的“%”都保持不变。

    *patsubst* 函数调用中的'%'字符可以用前置反斜杠(`\`)进行转义。而本身可能回引起对‘%’字符进行转义的反斜杠(`\`) 可以用更多的反斜杠进行转义。转义'%'字符或其他反斜杠的反斜杠在比较文件名或替换词干之前从 *pattern* 中删除。没有转义'%'字符的反斜杠不受干扰。例如，`the\%weird\\%pattern\\` 的结果是 `the%weird\`+任意单词+`pattern\\` ，最后两个反斜杠被保留的原因是它们不会影响"%"字符。
    
    单词之间的空格被折叠成单个空格字符；前导和尾随空格被丢弃。
    
    [6.3.1 Substitution References](https://www.gnu.org/software/make/manual/make.html#Substitution-Refs) 可以以更简单的方式实现 *patsubst* 函数的功能（*注，很有用*）
    ```makefile
    $(var:pattern=replacement)
    # 等效于
    $(patsubst pattern,replacement,$(var))

    # suffix在这里特指后缀，一定要是后缀
    $(var:suffix=replacement)
    # 等效于
    $(patsubst %suffix,%replacement,$(var))
    ```

    例如
    ```makefile
    objects = foo.o bar.o baz.o
    $(objects:.o=.c) # 等效于 $(patsubst %.o,%.c,$(objects))
    ```

- `$(strip string)`
	从*string*中删除前导和尾随空格，并将内部序列中的一个或多个空格字符替换为单个空格。`$(strip a b c )` 的结果是 `a b c`
	当使用`ifeq`或`ifneq`将某物与空字符''进行比较时，您通常需要一个只有空格的字符串来匹配空字符串(详见 [7 Conditional Parts of Makefiles](https://www.gnu.org/software/make/manual/make.html#Conditionals))(建议看看 bootloader 和 linux 的 makefile 写法)。例如：
    ```makefile
    .PHONY: all
    ifneq   "$(needs_made)" ""
    # ifneq 这行更好的方法是使用 
    # ifneq "$(strip $(needs_made))" ""
    all: $(needs_made)
    else
    all:;@echo 'Nothing to make!'
    endif
    ```

- `$(findstring find,in)`
	搜索 *in* 中 *find* 是否出现。如果出现了，则该值为 *find*；否则，该值为空。您可以在条件中使用此函数来测试给定字符串中是否存在特定子字符串。配合条件语句使用（[7.3 Conditionals that Test Flags](https://www.gnu.org/software/make/manual/make.html#Testing-Flags)）
	
- `$(filter pattern…,text)`
	返回 *text* 中与任何 *pattern* 单词匹配的所有由空格分隔的单词，删除任何不匹配的单词。
	filter 函数可用于分离变量中不同类型的字符串（例如文件名）。例如
    ``` makefile
    sources := foo.c bar.c baz.s ugh.h
    foo: $(sources)
            cc $(filter %.c %.s,$(sources)) -o foo
    ```
    foo依赖于foo. c、bar.c、baz.s和ugh.h，但只应在编译器的命令中指定foo.c、bar.c和baz.s。

 - `$(filter-out pattern…,text)`
	 返回 *text* 中与 *pattern* 均不匹配的所有由空格分隔的单词，删除与一个或多个 *pattern* 匹配的单词。这与函数 `filter` 完全相反。
- `$(sort list)`
	按词汇顺序对 *list* 中的单词进行排序，删除重复的单词。输出是由单个空格分隔的单词列表。并且排序*会删除重复*的单词。
- `$(word n,text)`
	返回 *text* 中的第n个单词。n的合法值从1开始。如果n大于文本中的单词数，则该值为空。
- `$(wordlist s,e,text)`
	返回 *text* 中以 *s* 开头、以*e*（含）结尾的单词列表。*s*的合法值从1开始；e可以从0开始。如果*s*大于文本中的单词数，则该值为空。如果*e*大于文本中的单词数，则返回文本末尾的单词。如果*s*大于*e*，则不返回任何内容。例如
    ```makefile
    $(wordlist 2, 3, foo bar baz)
    ```
    的结果是 `bar baz`
- `$(words text)`
	返回值是一个数，代表 *text* 中的单词数。因此`$(word $(words text),text)` 可以返回 *text* 中最后一个单词
- `$(firstword names…)`
	参数 *names* 被视为一系列由空格分隔的名称。返回值是 *names* 中的第一个名称。其余名称将被忽略。`$(firstword text)` 等效于 `$(word 1,text)`
- `$(lastword names…)`
	参数 *names* 被视为一系列由空格分隔的名称。返回值是 *names* 中的最后一个名称。其余名称将被忽略。`$(lastword names…)` 等效于 `$(word $(words text),text)`

    下面是`subst`和`patsubst`使用的真实示例。假设makefile使用`VPATH`变量来指定应该搜索先决条件文件的目录列表（见 [4.5.1 `VPATH` Search Path for All Prerequisites](https://www.gnu.org/software/make/manual/make.html#General-Search)）。此示例显示如何告诉C编译器在同一目录列表中搜索头文件。
    `VPATH`的值是由冒号分隔的目录列表，例如 `src:../headers`。首先，`subst`函数用于将冒号更改为空格：
    ```makefile
    $(subst :, ,$(VPATH))
    ```
    结果是 `src ../headers` 。然后使用 `patsubst` 将每个目录名转换为“-I”标志。这些可以添加到变量CFLAGS的值中，该值会自动传递给C编译器，如下所示：
    ```makefile
    override CFLAGS += $(patsubst %,-I%,$(subst :, ,$(VPATH)))
    ```
    效果是将文本 `-Isrc -I../headers` 附加到先前给定的CFLAGS值。使用覆盖指令，即使使用命令参数指定了`CFLAGS`的先前值，也会分配新值。

## 8.3 用于文件名的函数

一些内置的展开功能专门涉及拆分文件名或文件名列表。

以下每个函数都对文件名执行特定的转换。函数的参数被视为一系列由空格分隔的文件名。（前导和尾随空格被忽略。）系列中的每个文件名都以相同的方式转换，结果之间用单个空格连接。

- `$(dir names...)`

    提取 `names` 中每个文件名的目录部分。文件名的目录部分是其中直到最后一个斜杠（包括最后一个斜杠）的所有内容。如果文件名不含斜杠，则目录部分是字符串 "`./`". 例如，
    ```makefile
    $(dir src/foo.c hacks)
    ```
    执行的结果是 "`src/ ./`"

- `$(notdir names...)`

    提取 `names` 中每个文件名的目录部分以外的所有内容。如果文件名不包含斜杠，则保持不变。否则，最后一个斜杠前的所有内容都将从中删除。

    以斜杠结尾的文件名变成了空字符串。这很不幸，因为这意味着结果中的以空格分割的文件名并不总是与参数中的数量；但是我们没有任何其它有效的替代方案。

    例如

    ```makefile
    $(notdir src/foo.c hacks)
    ```

    结果是 "`foo.c hacks`"

- `$(suffix names…)`
    提取 `names` 中每个文件名的后缀。如果文件名包含句点，则后缀是以最后一个句点开始的所有内容。否则，后缀是空字符串。这通常意味着即使 `names` 不为空时结果也可能空，如果 `names` 包含多个文件名，结果可能包含更少的文件名。
    例如
    ```makefile
    $(suffix src/foo.c src-1.0/bar.c hacks)
    ```
    执行的结果是 "`.c .c`"
- `$(basename names…)`
    提取 `names` 中每个文件名的后缀以外的所有内容。如果文件名包含句点，则 *basename* 是从开始直到最后一个句点（不包括）的所有内容。**目录部分中的句点将被忽略**。如果没有句点，则 *basename* 是整个文件名。例如，
    ```makefile
    $(basename src/foo.c src-1.0/bar hacks)
    ```
    结果是 "`src/foo src-1.0/bar hacks`"

- `$(addprefix prefix,names…)`
    参数 `names` 被视为一系列由空格分隔的名称；`prefix` 用作一个单位。`prefix` 的值被添加到每个单独名称的前面，生成的多个较大名称之间用单个空格连接。例如，

    ```makefile
    $(addprefix src/,foo bar)
	```

	结果是 'src/foo src/bar'

- `$(join list1,list2)`

    将两个参数一个单词一个单词地相连：两个参数中的各自的第一个单词连接形成结果中的第一个单词，两个第二个单词形成结果的第二个单词，依此类推。所以结果的第 n 个单词来自每个参数的第 n 个单词。如果一个参数比另一个参数有更多的单词，则将额外的单词原封不动地复制到结果中。

    例如，"`$(join a b,.c .o)`" 生成 "`a.c b.o`"。

    列表中单词之间的空格不会保留；它被单个空格替换。

    此函数可以合并 `dir` 和 `notdir` 函数的结果，以生成提供给这两个函数的原始文件列表。

- `$(wildcard pattern)`
    参数 *pattern* 是文件名模式，通常包含通配符(如在shell文件名模式中)。`wildcard` 函数的结果是一个由空格分隔的、匹配该 *pattern* 的、现有文件的名称列表。请参阅 [4.4 Using Wildcard Characters in File Names](https://www.gnu.org/software/make/manual/make.html#Wildcards)。

- `$(realpath names…)`
    对于 `names` 中的每个文件名，返回规范的绝对名称。规范名称不包含任何 `.` 或 `..` 元素，也不包含任何重复的路径分隔符（/）或符号链接(symlink)。如果失败，则返回空字符串。有关可能的失败原因列表，请参阅 realpath(3) 文档（译者注，参阅 [realpath(3) — Linux manual page](https://www.man7.org/linux/man-pages/man3/realpath.3.html) 作为参考）。

- `$(abspath names…)`
    对于 `names` 中的每个文件名，返回一个不包含任何 `.` 或 `..` 元素、也不包含任何重复的路径分隔符（/）的绝对名称。请注意，与 `realpath` 函数相比，`abspath` 不解析符号链接，也不要求文件名引用现有文件或目录。使用 `wildcard` 函数测试是否存在。

## 8.4 条件函数

有四个函数提供条件展开。这些函数的一个关键是并不是所有的参数都在最初被展开。只有那些需要展开的参数才会被展开。

- `$(if condition,then-part[,else-part])`

- `$(or condition1[,condition2[,condition3…]])`

- `$(and condition1[,condition2[,condition3…]])`

- `$(intcmp lhs,rhs[,lt-part[,eq-part[,gt-part]]])`

## 8.5 `let` 函数

`let` 函数提供了一种限制变量范围的方法。`let` 表达式中命名变量的赋值仅在 `let` 表达式提供的文本中有效，这种赋值不会影响任何外部范围的此名称的变量。

此外，`let` 函数通过将所有未分配的值分配给最后一个命名变量来启用列表解包。

`let` 函数的语法是：

```makefile
$(let var [var ...],[list],text)
```

前两个参数，`var` 和 `list`，在完成任何其他操作之前被展开；请注意，最后一个参数 `text` 不会同时被展开。接下来，`list`的展开值的每个单词会依次被绑定到·`var` 中的每个变量名，最后的变量名称被绑定到 展开`list` 的其余部分。换句话说，`list` 的第一个单词绑定到第一个变量 `var`，第二个单词绑定到第二个变量`var`，依此类推。

如果 `var` 中的变量名称数量多于 `list` 中的单词数量，则将剩余的 `var` 变量名称设置为空字符串。如果 `var` 少于 `list` 中的单词，则将最后一个 `var` 设置为 `list` 中的所有剩余单词。

在 `let` 的执行过程中，`var` 中的变量被分配为*简单展开*变量。请参阅 [6.2 The Two Flavors of Variables](https://www.gnu.org/software/make/manual/make.html#Flavors)

绑定所有变量后，展开 `text` 以提供 `let` 函数的结果。

例如，这个宏颠倒了列表中作为第一个参数给出的单词的顺序：

```makefile
reverse = $(let first rest,$1,\
            $(if $(rest),$(call reverse,$(rest)) )$(first))

all: ; @echo $(call reverse,d c b a)
```

会打印 "`a b c d`"。第一次调用的时候，`let` 将 `$1` 展开为 `d c b a`。然后它将 `first` 分配给 `d`，并将 `rest` 分配给 `c b a`。然后它将展开 *if语句*，其中 `$(rest)` 不为空，因此我们递归调用 `rest` 当前值是 `c b a` 的 `reverse` 函数。`let` 的递归调用将 `first` 分配给 `c`，`rest` 分配给 `b a`。递归一直持续到 `let` 被调用时只有一个值 `a`。这里 `first` 是 `a`，`rest` 为空，所以我们不递归，而是简单地将 `$(first)`展开为 `a` 并返回，这个过程中增加了 `b` 和其它。

（
译者注，这里稍微解释一下
```makefile
$(let var [var ...],[list],text)
```
中的 `var [var ...]` 表示的是 `var` 至少有一个变量名称，如果有多个，则以空格分隔。

和上个 `makefile` 中对应，`first rest` 对应 `var [var]`。
`$1` 展开成的 `d c b a` 对应 `[list]`
后续就跟着原文中的解释理解就可以了
）

`reverse` 调用完成后，不再设置 `first` 和 `rest`。如果这些名称的变量事先存在，它们不受宏 `reverse` 展开的影响。


## 8.6 `foreach` 函数

`foreach` 函数类似于 `let` 函数，但与其他函数非常不同。它导致一段文本被重复使用，但每次都对其执行不同的替换。`foreach` 函数类似于 shell *sh*中的 `for` 命令和 C-shell *csh* 中的 `foreach` 命令。

`foreach` 函数的语法是
```makefile
$(foreach var,list,text)
```

前两个参数，`var` 和 `list`，在做其他任何事情之前展开；注意最后一个参数 `text` 没有同时展开。然后将 `list` 中以空格分隔的字符串依次赋值给 `var` 的展开值命名的变量，并在每次赋值后执行 `text` 展开的表达式；重复直到 `list` 的最后一个字符串。根据上述描述可推测 `text` 包含对 `var` 变量的引用，所以每次它的展开都会不同。

结果是 `text` 被展开的次数等于 `list` 中以空格分隔的单词的个数。`text` 的多个展开被连接起来，它们之间有空格，作为 `foreach` 的结果。

这个简单的示例将变量 "*files*" 设置为列表 "*dirs*" 中目录中所有文件的列表：

```makefile
dirs := a b c d
files := $(foreach dir,$(dirs),$(wildcard $(dir)/*))
```

这里 `text` 是 "`$(wildcard $(dir)/*)`"。第一次重复找到 `dir` 的值 "`a`" ，因此它产生的结果是 "`$(wildcard a/*)`"；第二次重复产生的结果是 "`$(wildcard b/*)`"；第三次是 "`$(wildcard c/*)`"。

此示例与以下示例具有相同的结果（设置 "`dirs`" 除外）：

```makefile
files := $(wildcard a/* b/* c/* d/*)
```

当 `text` 很复杂时，您可以通过给它一个具有名称的变量来提高易读性：

```makefile
find_files = $(wildcard $(dir)/*)
dirs := a b c d
files := $(foreach dir,$(dirs),$(find_files))
```

这里我们以这种方式使用变量 `find_files`。我们使用简单的 `=` 来定义一个*递归展开*变量，这样它的值就包含了一个实际的函数调用，这个函数调用将在 `foreach` 的控制下重新展开；*简单展开*的变量则不会重新展开，因为 `wildcard` 函数只会在定义 `find_files` 时被调用一次。

与 `let` 函数一样，`foreach` 函数对变量 `var` 没有永久影响；`var` 在 `foreach` 函数调用后的值和风格与之前相同。从 `list` 中获取的其他值仅在 `foreach` 执行期间暂时有效。变量 `var` 是在 `foreach` 执行期间*简单展开*的变量。如果 `var` 在 `foreach` 函数调用之前未定义，则在调用后也会是未定义的。请参阅 [6.2 The Two Flavors of Variables](https://www.gnu.org/software/make/manual/make.html#Flavors)。

在使用可能会是某个变量名的复杂变量表达式时，您必须小心，因为许多奇怪的东西都是有效的变量名，但可能不是你想要的。例如，

```makefile
files := $(foreach Esta-escrito-en-espanol!,b c ch,$(find_files))
```

如果 `find_files` 的值引用名称为 "`Esta-escrito-en-espanol!`" 的变量(这是一个很长的名字，不是吗？)，则可能会很有用，但这更有可能是一个错误。

## 8.7 `file` 函数

`file` 函数允许 *makefile* 对文件进行写入或读取。支持两种写入模式：覆盖（其中文本被写入文件的开头，并且任何现有内容都将丢失）和追加（其中文本被写入文件的末尾，保留现有内容）。在这两种情况下，如果文件不存在，则创建文件。如果文件无法打开进行写入，或者写入操作失败，这是一个致命的错误。`file` 函数在写入文件时展开为空字符串。

从文件中读取时，`file` 函数会展开为文件的逐字内容，但最终的换行符（如果有的话）将被剥离。尝试从不存在的文件中读取会展开为空字符串。

`file` 函数的语法是

```makefile
$(file op filename[,text])
```

当 `file` 函数被评估时，它的所有参数在第一时间被展开，然后 `filename` 指示的文件将以 `op` 描述的模式打开。

操作 `op` 可以是 `>` 表示文件将被新内容覆盖，`>>` 表示将新内容附加到文件的当前内容后，或 `<` 表示文件的内容将被读入。`filename` 指定要写入或读取的文件。`op` 和 `filename` 之间可以有空格。

读取文件时，提供 `text` 值会发生错误。

写入文件时，`text` 将被写入文件。如果 `text` 尚未以换行符结尾，则将写入最终换行符（即使 `text` 是空字符串）。如果根本没有给出 `text` 参数，则不会写入任何内容。

例如，如果您的构建系统具有有限的命令行大小，并且您的配方运行的命令也可以接受文件中的参数，则 `file` 函数非常有用。许多命令使用这样的约定——以 `@` 为前缀的参数指定文件包含更多参数。然后您可以这样编写配方：

```makefile
program: $(OBJECTS)
    $(file >$@.in,$^)
    $(CMD) $(CMDFLAGS) @$@.in
    @rm $@.in
```

如果命令要求每个参数在输入文件的单独行上，您可以像这样编写配方：

```makefile
program: $(OBJECTS)
    $(file >$@.in) $(foreach O,$^,$(file >>$@.in,$O))
    $(CMD) $(CMDFLAGS) @$@.in
    @rm $@.in
```

## 8.8 `call` 函数

`call` 函数的独特之处在于它可用于创建新的参数化函数。您可以编写一个复杂的表达式作为变量的值，然后使用 `call` 将其展开为不同的值。

`call` 函数的语法是

```makefile
$(call variable,param,param,…)
```

当 *make* 展开此函数时，它将每个 `param` 分配给临时变量 `$(1)`、`$(2)` 等。变量 `$(0)` 将包含 `variable`。参数没有最大数量，也没有最小值，但是使用没有参数的 `call` 没有意义。

然后 `variable` 在这些临时赋值的上下文中展开为 *make* 变量。因此，在调用 `call` 中，`variable` 值中对 `$(1)` 的任何引用都将解析为的第一个 `param`。

请注意此处的 `variable` 是变量的名称，而不是对该变量的引用。因此，您在编写它时通常不会使用 "`$`" 或括号。（但是，如果您希望此处的变量名称不是常量，则可以使用变量引用。）

如果 `variable` 是内置函数的名称，则始终调用内置函数（即使该名称的 *make* 变量也存在）。

`call` 函数在将 `param` 分配给临时变量之前展开 `param` 参数。这意味着包含对具有特殊展开规则（如 `foreach` 或 `if`）的内置函数的引用的 `variable` 值可能无法按预期工作。

一些例子可能会使这一点更清楚。

这个宏只是反转它的参数：

```makefile
reverse = $(2) $(1)

foo = $(call reverse,a,b)
```

执行后 `foo` 会包含 `b a`。

这个稍微有趣一点：它定义了一个宏来搜索 `PATH` 中程序的第一个实例：

```makefile
pathsearch = $(firstword $(wildcard $(addsuffix /$(1),$(subst :, ,$(PATH)))))

LS := $(call pathsearch,ls)
```

执行后变量 `LS` 包含 `/bin/ls` 或者相似的内容。

`call` 函数可以嵌套。每个递归调用都有自己的 `$(1)` 等局部值，这些值掩盖了更高级别 `call` 函数的值。例如，这是一个 `map` 函数的实现：

```makefile
map = $(foreach a,$(2),$(call $(1),$(a)))
```

现在您可以将在一个步骤中将通常只接受一个参数（例如 `origin`）的函数 `map`(映射) 到多个值：

```makefile
o = $(call map,origin,o map MAKE)
```

并以包含 "`file file default`" 之类的内容的 `o` 结束。

最后一个注意事项：向 `call` 函数的参数中添加空白字符时要小心。与其他函数一样，第二个和后续参数中包含的任何空白字符都将保留；这可能会导致奇怪的影响。在提供给 `call` 函数的参数时，删除所有无关的空格通常是最安全的。

## 8.9 `value` 函数

`value` 函数为您提供了一种使用变量值而不进一步展开变量值的方法。请注意，这不会撤消已经发生的展开；例如，如果您创建一个*简单展开*的变量，它的值会在定义期间展开；在这种情况下，值函数将返回与直接使用变量相同的结果。

`value` 函数的语法是

```makefile
$(value variable)
```

请注意此处的 `variable` 是变量的名称，而不是对该变量的引用。因此，您在编写它时通常不会使用 "`$`" 或括号。（但是，如果您希望此处的变量名称不是常量，则可以使用变量引用。）

此函数的结果是一个包含没有发生过任何展开的 `variable` 变量值的字符串。例如，在这个 *makefile* 中：

```makefile
FOO = $PATH

all:
    @echo $(FOO)
    @echo $(value FOO)
```

第一个输出行将是 `ATH`，因为 "`$P`" 将作为 *make* 变量展开，而第二个输出行将是你的环境变量 `$PATH` 的当前值，因为 `value` 函数避免了展开。

`value` 函数最常与 `eval` 函数结合使用。请参阅 [8.10 The eval Function](https://www.gnu.org/software/make/manual/make.html#Eval-Function)。

## 8.10 `eval` 函数

`eval` 函数非常特殊：它允许您定义新的不恒定的 *makefile* 结构；这是评估其他变量和函数的结果。`eval` 函数的参数被展开，然后扩展的结果被解析为 *makefile* 语法。展开的结果可以定义新的 *make* 变量、目标、隐式或显式规则等。

`eval` 函数的结果总是空字符串；因此，它几乎可以放置在 *makefile* 中的任何位置，而不会导致语法错误。

重要的是要意识到 `eval` 参数被扩展了两次；首先是由 `eval` 函数，然后当它们被解析为 *makefile* 语法时，第一次展开的结果被再次展开。这意味着在使用 `eval` 时，您可能需要为 "`$`" 字符提供额外的转义级别。`value` 函数（请参阅 [8.9 The value Function](https://www.gnu.org/software/make/manual/make.html#Value-Function)）有时在这些情况下很有用，可以规避不需要的扩展。

下面是一个如何使用 `eval` 的例子；这个例子结合了许多概念和其它函数。尽管与仅仅是写出规则相比，在这个例子中使用 `eval` 似乎过于复杂，但要考虑两件事：首先，模板定义（在 `PROGRAM_template` 中）可能需要比这里的复杂更多；其次，您可以将这个例子中复杂的、通用的部分放入另一个 *makefile* 中，然后将其包含在所有单独的 *makefile* 中。现在您的单独的 *makefile* 非常简单。

```makefile
PROGRAMS    = server client

server_OBJS = server.o server_priv.o server_access.o
server_LIBS = priv protocol

client_OBJS = client.o client_api.o client_mem.o
client_LIBS = protocol

# Everything after this is generic

.PHONY: all
all: $(PROGRAMS)

define PROGRAM_template =
 $(1): $$($(1)_OBJS) $$($(1)_LIBS:%=-l%)
 ALL_OBJS   += $$($(1)_OBJS)
endef

$(foreach prog,$(PROGRAMS),$(eval $(call PROGRAM_template,$(prog))))

$(PROGRAMS):
    $(LINK.o) $^ $(LDLIBS) -o $@

clean:
    rm -f $(ALL_OBJS) $(PROGRAMS)
```

（译者注
- `define ... endef` 参阅 [5.8 Defining Canned Recipes](https://www.gnu.org/software/make/manual/make.html#Canned-Recipes)。

- `$(foreach ...)` 参阅 [8.6 The foreach Function](https://www.gnu.org/software/make/manual/make.html#Foreach-Function)

- `$(call ...)` 和 `$(1)` 参阅 [8.8 The call Function](https://www.gnu.org/software/make/manual/make.html#Call-Function)

- `LDLIBS` 参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)

- `$^` 和 `$@` 参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)

）

## 8.11 `origin` 函数

`origin` 函数与大多数其他函数不同，因为它不对变量的值进行操作；它告诉你一些关于变量的事情。具体来说，它告诉你它来自哪里。

`origin` 函数的语法

```makefile
$(origin variable)
```

请注意此处的 `variable` 是变量的名称，而不是对该变量的引用。因此，您在编写它时通常不会使用 "`$`" 或括号。（但是，如果您希望此处的变量名称不是常量，则可以使用变量引用。）

此函数的结果是一个字符串，告诉您变量变量是如何定义的：

- ‘undefined’

- ‘default’

- ‘environment’

- ‘environment override’

- ‘file’
    `variable` 由某个 *makefile* 定义

- ‘override’

- ‘automatic’

此信息是非常有用（除了满足您的好奇心），其可以用于确定您是否要相信变量的值。例如，假设您有一个包含另一个 *makefile* `bar` 的 *makefile* `foo`。如果您运行命令 `make -f bar`，您希望在 `bar` 中定义一个变量 `bletch`，即使环境包含 `bletch` 的定义。但是，如果 `foo` 在包含 `bar` 之前定义了 `bletch`，您不希望覆盖该定义。这可以通过在 `foo` 中使用 `override` 指令来完成，使该定义优先于 `bar` 中的后面定义；不幸的是，`override` 指令也会覆盖任何命令行定义。因此，`bar` 可以包括：

```makefile
ifdef bletch
ifeq "$(origin bletch)" "environment"
bletch = barf, gag, etc.
endif
endif
```

如果 `bletch` 是从环境中定义的，这将重新定义它。

如果您想覆盖来自环境的 `bletch` 的先前定义，即使在'`-e`'下，您也可以编写：

```makefile
ifneq "$(findstring environment,$(origin bletch))" ""
bletch = barf, gag, etc.
endif
```

这里，如果 "`$(origin bletch)`" 返回 "environment" 或 "environment override"，则会进行重新定义。参阅 [8.2 Functions for String Substitution and Analysis](https://www.gnu.org/software/make/manual/make.html#Text-Functions)

## 8.12 `flavor` 函数

`flavor` 函数，像 `origin` 函数一样，不对变量的值进行操作，而是告诉你一些关于变量的事情。具体来说，它告诉你变量的风格（参见 [6.2 The Two Flavors of Variables](https://www.gnu.org/software/make/manual/make.html#Flavors)）。

`flavor` 函数的语法

```makefile
$(flavor variable)
```

请注意此处的 `variable` 是变量的名称，而不是对该变量的引用。因此，您在编写它时通常不会使用 "`$`" 或括号。（但是，如果您希望此处的变量名称不是常量，则可以使用变量引用。）

此函数的结果是一个字符串，用于标识 `variable` 变量的风格：
- 'undefined'

- 'recursive'

- 'simple'

## 8.13 用于控制 make 的函数

这些函数控制 *make* 运行的方式。通常，它们用于向 *makefile* 的用户提供信息，或者在检测到某种环境错误时导致 *make* 停止。

- `$(error text…)`
    生成致命错误并且消息为 `text`。请注意，每当评估此函数时都会生成错误。因此，如果您将其放在配方中或递归变量赋值的右侧，则要到稍后才会对其进行评估。`text` 将在生成错误之前展开。
    例如
    ```makefile
    ifdef ERROR1
    $(error error is $(ERROR1))
    endif
    ```

    如果定义了 *make* 变量 `ERROR1`，将在读取 *makefile* 期间生成致命错误。或者，

    ```makefile
    ERR = $(error found an error!)

    .PHONY: err
    err: ; $(ERR)
    ```

    如果调用 `err` 目标，将在 *make* 运行时生成致命错误。

- `$(warning text…)`
    此函数的工作方式类似于上面的 `error` 函数，不同之处在于 *make* 不会退出。相反， `text` 被展开并显示生成的消息，但 *makefile* 的处理仍在继续。

    此函数扩展的结果是空字符串。

- `$(info text…)`
    此函数进用于将其（展开的）参数打印到标准输出。没有添加 *makefile* 名称或行号。此函数展开的结果是空字符串。

## 8.14 `shell` 函数

`shell` 函数与 `wildcard` 函数（请参阅 [4.4.3 The Function wildcard](https://www.gnu.org/software/make/manual/make.html#Wildcard-Function)）以外的任何其他函数不同，因为它与 *make* 之外的世界通信。

`shell` 函数为 *make* 提供的功能，与反引号(\`)为大多数 shell 中提供的功能相同：它进行命令扩展。这意味着它将 shell 命令作为参数并展开成命令的输出。*make* 对结果所做的唯一处理是将每个换行符（或回车-换行符对）转换为单个空格。如果有尾随（回车和）换行符，它将被简单地删除。(译者注，写这篇翻译用的是 markdown 语法，在 markdown 中反引号会指示代码段的出现，所以在这一段的 “反引号(\`)” 处我用反斜杠转义了一下。如果看的是 markdown 的展示效果的话，不会看到反斜杠，如果看的是 markdown 编辑模式的话，才会看到这个反斜杠)。

通过调用 `shell` 函数运行的命令在函数调用被展开时运行（请参阅 [3.7 How make Reads a Makefile](https://www.gnu.org/software/make/manual/make.html#Reading-Makefiles)）。因为这个函数涉及生成一个新的 shell，所以您应该仔细考虑在*递归展开*变量和*简单展开*变量中使用 `shell` 函数对性能的影响（请参阅变量的两种风格）。

`shell` 函数的另一种选择是 ‘`!=`’ 赋值操作符；它提供了类似的行为，但有细微的区别（请参阅 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting)）。 ‘`!=`’ 赋值操作符被包含在较新的 POSIX 标准中。

使用 `shell` 函数或 ‘`!=`’ 赋值操作符后，其退出状态将放在 `.SHELLSTATUS` 变量中。

以下是一些使用 `shell` 函数的示例：

```makefile
contents := $(shell cat foo)
```

将 *contents* 设置为文件 *foo* 的内容，每行之间用空格（而不是换行符）分隔。

```makefile
files := $(shell echo *.c)
```

将 *files* 设置为 “`*.c`” 的展开。除非 *make* 使用非常奇怪的 *shell*，否则这与 "`$(wildcard *.c)`" 具有相同的结果（只要至少存在一个 “`*.c`” 文件）。

所有标记为 `export` 的变量也将传递给 `shell` 函数启动的 *shell*。可以创建一个变量展开循环：例如这个makefile：

```makefile
export HI = $(shell echo hi)
all: ; @echo $$HI
```

当 *make* 想要运行配方时，它必须将变量 `HI` 添加到环境中；为此，它必须被扩展。这个变量的值需要调用 `shell` 函数，要调用它，我们必须创建它的环境。由于 `HI` 被导出，我们需要展开它来创建它的环境。依此类推。在这个模糊的情况下，*make* 将使用提供给 *make* 的环境中变量的值，或者如果没有，则使用空字符串，而不是循环或发出错误。这通常是你想要的；例如：

```makefile
export PATH = $(shell echo /usr/local/bin:$$PATH)
```

然而，在 `export HI = ` 处使用*简单展开*变量(`:=`)会更简单、更有效 。

## 8.15 `guile` 函数

如果 GNU *make* 的构建支持 GNU Guile 作为内嵌扩展语言，那么 `guile` 函数将可用。`guile` 函数接受一个参数，该参数首先由 *make* 以正常方式扩展，然后传递给 GNU Guile 评估器。评估器的结果被转换为字符串并用作 *makefile* 中 `guile` 函数的展开。有关在 Guile 中编写展开的详细信息，请参阅 [12.1 GNU Guile Integration](https://www.gnu.org/software/make/manual/make.html#Guile-Integration)。

您可以通过检查 `.FEATURES` 变量中的 `guile` 字符来确定是否支持 GNU Guile。

# 9 如何运行 make

一个用于说明如何重新编译程序的 *makefile* 可以以多种方式被使用。最简单的使用方式是重新编译每个过期的文件。通常，*makefile* 就是这样编写的，即如果您在没有参数的情况下运行 *make*，它就会这样做。

但是您可能只想更新其中的一些文件；您可能希望使用不同的编译器或不同的编译器选项；您可能只想找出哪些文件已过期，而不更改它们。

通过在运行 *make* 时给出参数，您可以做任何这些事情和许多其他事情。

make的退出状态始终是三个值之一：

- 0
    如果退出状态是0，则意味着 *make* 运行成功。
- 2
    如果退出状态为2，则意味着 *make* 遇到错误，它将打印描述特定错误的消息。
- 1
    如果您使用 '`-q`' 标志并 make 确定某个目标尚未更新，则退出状态为1。参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/make.html#Instead-of-Execution)

## 9.1 用于指定 *Makefile* 的参数

指定 *makefile* 名称的方法是使用 '`-f`' 或 '`--file`' 选项（'`--makefile`' 也可以）。例如，'`-f altmake`'表示使用文件 *altmake* 作为 *makefile*。

如果您多次使用 '`-f`' 标志并在每个 '`-f`' 后面加上一个参数，则所有指定的文件都将共同用作 *makefile*。

如果您不使用 '`-f`' 或 '`--file`' 标志，则默认是按顺序尝试 *GNUmakefile*、*makefile*和*Makefile*，三个存在或可以制作的第一个（请参阅 [3 Writing Makefiles](https://www.gnu.org/software/make/manual/make.html#Makefiles)）。

## 9.2 用于指定终点目标的参数

终点目标(goals)是 *make* 最终应该努力更新的目标(target)。如果其他目标作为终点目标的先决条件或终点目标的先决条件的先决条件等出现，它们也会更新。

默认情况下，终点目标是 makefile 中的第一个目标（不包括以句点开头的目标）。因此，makefile 通常是这样编写的：以便第一个目标用于编译它们描述的整个程序。**如果 makefile 中的第一个规则有多个目标，则只有规则中的第一个目标成为默认目标，而不是整个列表**。您可以在 makefile 中使用 `.DEFAULT_GOAL` 变量来管理默认终点目标的选择（请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)）。

您还可以使用命令行参数给 *make* 指定一个或多个不同的终点目标。使用终点目标的名称作为参数。如果您指定了多个终点目标，*make* 按照您命名它们的顺序依次处理每个终点目标。

makefile 中的任何目标都可以指定为终点目标（除非它以 '`-`' 开头或包含 '`=`'，在这种情况下，它将分别被解析为切换开关(switch)或变量定义）。如果 *make* 可以找到说明如何制作它们的隐式规则，则即使不在 makefile 中的目标也可以指定。

*Make* 会将特殊变量 `MAKECMDGOALS` 设置为您在命令行上指定的终点目标列表。如果命令行上没有给出终点目标，则此变量为空。请注意，此变量应仅在特殊情况下使用。

适当使用的一个示例是避免在 *clean* 规则期间包含 *.d* 文件（请参阅 [4.14 Generating Prerequisites Automatically](https://www.gnu.org/software/make/manual/make.html#Automatic-Prerequisites)），因此 make 不会只是为了立即再次删除它们而创建它们：

```makefile
sources = foo.c bar.c

ifeq (,$(filter clean,$(MAKECMDGOALS))
include $(sources:.c=.d)
endif
```

指定终点目标的一种用途是假如您只想编译程序的一部分，或者只想编译几个程序中的一个。将您要重新制作的每个文件指定为终点目标。例如，考虑一个包含多个程序的目录，其吧makefile 如下所示：

```makefile
.PHONY: all
all: size nm ld ar as
```

如果您正在处理程序 `size`，您可能想键入 “`make size`”，以便仅重新编译该程序的文件。

指定终点目标的另一个用途是制作通常不制作的文件。例如，可能有一个调试输出的文件，或者一个专门为测试而编译的程序版本，它在 makefile 中有一个不是默认目标的先决条件的规则。

指定终点目标的另一个用途是运行与假目标（请参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)）或空目标（请参阅 [4.8 Empty Target Files to Record Events](https://www.gnu.org/software/make/manual/make.html#Empty-Targets)）关联的配方。许多 makefile 包含一个名为 “*clean*” 的假目标，它会删除除源文件之外的所有内容。当然，只有当您使用 “`make clean`” 明确请求时，才会这样做。以下是典型的假目标和空目标名称列表。请参阅 [16.6 Standard Targets for Users](https://www.gnu.org/software/make/manual/make.html#Standard-Targets)，以获取 GNU 软件包使用的所有标准目标名称的详细列表。

- all
    创建 makefile 知道的所有顶级目标。

- clean
    删除通常由运行 make 创建的所有文件。

- mostlyclean
    类似 “*clean*”，但可能会避免删除一些人们通常不想重新编译的文件。例如，GCC的 “*mostlyclean*” 目标不会删除 *libgcc.a*，因为很少需要重新编译它，并且重新编译需要大量时间。

- distclean<br>realclean<br>clobber
    这些目标中的任何一个都可能被定义，用来删除比 “*clean*” 更多的文件。例如，这将删除您通常为编译做准备而创建的配置文件或链接，即使 makefile 本身无法创建这些文件。

- install
    将可执行文件复制到用户通常搜索命令的目录中；将可执行文件使用的任何辅助文件复制到将查找它们的目录中。

- print
    打印已更改的源文件的列表。

- tar
    创建源文件的tar文件。

- shar
    创建源文件的 shell 存档（shar文件）。

- dist
    创建源文件的分发文件。这可能是tar文件、shar文件或上述文件之一的压缩版本，甚至不止一个。

- TAGS
    更新此程序的标签表。

- check<br>test
    对这个 makefile 构建的程序执行自我测试。

## 9.3 请不要执行配方

*makefile* 告诉 *make* 如何判断目标是否是最新的，以及如何更新每个目标。但你并不是每次都想更新目标。某些选项指定 *make* 的其他活动。

- ‘-n’<br>‘--just-print’<br>‘--dry-run’<br>‘--recon’
    “无操作”。导致 make 打印使目标更新所需的配方，但实际上并没有执行它们。请注意，即使使用此标志，某些配方仍在执行（请参阅 [5.7.1 How the MAKE Variable Works](https://www.gnu.org/software/make/manual/make.html#MAKE-Variable)）。此外，更新包含的 makefile 所需的任何配方仍在执行（请参阅 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/make.html#Remaking-Makefiles)）。

- ‘-t’<br>‘--touch’
    “创建”。将目标标记为最新，而不实际更改它们。换句话说，*make* 假装更新目标，但并没有真正更改其内容；相反，只更新它们的修改时间。

- ‘-q’<br>‘--question’
    “质疑”。静默检查目标是否是最新的，但不执行配方；退出代码显示是否需要任何更新。

- ‘-W file’<br>‘--what-if=file’<br>‘--assume-new=file’<br>‘--new-file=file’
    “如果”。每个 “`-W`” 标志后面都有一个文件名。给定文件的修改时间由 make 标记为当前时间，尽管实际修改时间保持不变。您可以将 “`-W`” 标志与 “`-n`” 标志结合使用，以查看如果您要修改特定文件会发生什么。

使用 '`-n`' 标志，*make* 打印它通常执行的配方，但实际上并没有执行它们。

使用 “`-t`” 标志，*make* 忽略规则中的配方，并对需要重新制作的每个目标使用（实际上）命令 `touch`。`touch` 命令也被打印出来，除非使用 “`-s`” 或 `.SILENT`。为了速度，*make* 实际上并不调用程序 `touch`。它直接完成工作。 

使用 “`-q`” 标志，*make* 不打印任何内容，也不执行配方，但当且仅当要考虑的目标已经是最新的时，它返回的退出状态代码为零。如果退出状态为 **1**，则需要进行一些更新。如果 *make* 遇到错误，则退出状态为 **2**，因此您可以区分错误和不是最新的目标。

在同一个 make 调用中使用这三个标志中的一个以上是错误的。

“`-n`”、“`-t`” 和 “`-q`” 选项不影响以 `+` 字符开头或包含字符串`$(MAKE)` 或 `${MAKE}` 的配方行。请注意，无论这些选项如何，仅运行包含 `+` 字符或字符串 `$(MAKE)` 或 `${MAKE}` 的行。同一规则中的其他行不运行，除非这些行也以 '`+`' 开头或包含 `$(MAKE)` 或 `${MAKE}` 的配方行。参阅 [5.7.1 How the MAKE Variable Works](https://www.gnu.org/software/make/manual/make.html#MAKE-Variable)

'`-t`' 标志防止伪目标 (请参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)) 被更新，除非存在以 '`+`' 开头或包含 `$(MAKE)` 或 `${MAKE}` 的配方行。

'`-W`'标志提供了两个功能：
- 如果您还使用 "`-n`" 或 "`-q`" 标志，您可以看到如果您要修改某些文件，*make* 会做什么。
- 如果没有 "`-n`" 或 "`-q`" 标志，当 *make* 实际执行配方时，'`-W`'标志可以指示 *make* 就好像某些文件已被修改一样，而无需实际运行这些文件的配方。

请注意，选项 '`-p`' 和 '`-v`' 允许您获取有关 make 或正在使用的 makefile 的其他信息。[9.8 Summary of Options](https://www.gnu.org/software/make/manual/make.html#Options-Summary)

## 9.4 避免某些文件的重新编译

有时您可能已经更改了一个源文件，但您不想重新编译依赖于它的所有文件。例如，假设您向许多其他文件依赖的头文件添加了一个宏或声明。保守地说，*make* 假定头文件中的任何更改都需要重新编译所有依赖文件，但是你知道它们不需要重新编译，你不浪费时间等待它们编译。

如果您在更改头文件之前预料到问题，您可以使用 “`-t`” 标志。此标志告诉 *make* 不要运行规则中的配方，而是通过更改其最后修改日期来将目标标记为最新。您将遵循以下过程：

1. 使用命令 '`make`' 重新编译真正需要重新编译的源文件，确保目标文件在开始之前是最新的。
2. 在头文件中进行更改。
3. 使用命令 '`make -t`' 将所有目标文件标记为最新。下次运行 *make* 时，头文件中的更改不会导致任何重新编译。

如果您已经在某些文件确实需要重新编译的时候更改了头文件，那么做上述步骤已经太晚了。相反，您可以使用 “`-o file`” 标志，它将指定的文件标记为 “old”（请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/make.html#Options-Summary)）。这意味着文件本身不会被重做，并且不会因为此原因重做任何其他内容。请遵循以下过程：

1. 使用 "`make -o headerfile`" 重新编译由于与特定头文件无关的原因需要编译的源文件。如果涉及多个头文件，请为每个头文件使用单独的 “`-o`” 选项。
2. 使用 “`make -t`” 创建所有目标文件。

## 9.5 覆盖变量

包含 “`=`” 的**参数**指定变量的值: “`v=x`” 将变量 `v` 的值设置为 `x`。如果以这种方式指定值，则 makefile 中相同变量的所有普通赋值都将被忽略; 我们说它们**已被命令行参数覆盖**。

使用此工具的最常见方法是将额外的标志传递给编译器。例如，在正确编写的 makefile 中，变量 `CFLAGS` 包含在运行C编译器的每个配方中，因此文件 *foo.c* 将被编译如下:

```makefile
cc -c $(CFLAGS) foo.c
```

因此，您为 `CFLAGS` 设置的任何值都会影响发生的每次编译。makefile 可能指定 `CFLAGS` 的常用值，如下所示：

```makefile
CFLAGS=-g
```

每次运行make时，如果需要，可以覆盖此值。例如，如果你键入 `make CFLAGS='-g -O'`，每个C编译将用 `cc -c -g -O` 完成。(这也说明了在覆盖变量时，如何在 shell 中使用引号将空格和其他特殊字符括在变量的值中。)

变量 `CFLAGS` 只是存在的许多、您可以通过这种方式更改它们的标准变量之一。有关完整列表，请参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)。

您还可以对 makefile 进行编程，以查看您自己的其他变量，从而使用户能够通过更改变量来控制 makefile 工作方式的其他方面。

使用命令行参数覆盖变量时，可以定义*递归展开*变量或*简单展开*变量。上面显示的示例创建了一个*递归展开*变量; 要创建一个*简单展开*变量，请编写 “`:=`” 或 “`::=`” 而不是 “`=`”。但是，**除非您想在指定的值中包含变量引用或函数调用，否则创建哪种类型的变量没有区别**。

有一种方法，makefile 可以更改您已覆盖的变量。这是使用 `override` 指令，即 `override variable = value`(见 [6.7 The override Directive](https://www.gnu.org/software/make/manual/make.html#Override-Directive))。

## 9.6 测试一段程序的编译

通常，当执行 shell 命令时发生错误时，*make* 立即放弃，返回非零状态。不会为任何目标执行进一步的配方。该错误意味着无法正确重做终点目标，并且 *make* 在知道后立即报告。

你可能不仅仅是想要编译刚刚更改的程序。相反，您宁愿 *make* 尝试编译每个可以尝试的文件，以向您显示尽可能多的编译错误。

在这些情况下，您应该使用 “`-k`” 或 “`--being-go`” 标志。这告诉 *make* 继续考虑待处理目标的其他先决条件，如果有必要，在它放弃并返回非零状态之前重新创建它们。例如，在编译一个目标文件时出错后，“`make -k`” 将继续编译其他目标文件，即使 *make* 已经知道链接它们是不可能的。除了在失败的 shell 命令后继续，“`make -k`” 在发现它不知道如何制作目标或先决条件文件后会尽可能地继续。这总是会导致错误消息，但如果没有 “`-k`”，这是一个致命的错误（请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/make.html#Options-Summary)）。

*make* 的通常行为假设您的目的是使终点目标保持最新; 一旦 *make* 了解到这是不可能的，它不妨立即报告失败。'`-k`' 标志表示，真正的目的是尽可能多地测试一下程序中所做的更改，也许是为了找到几个独立的问题，以便您可以在下一次尝试编译之前将它们全部更正。这就是 [Emacs](https://www.gnu.org/software/emacs/) 的 `M-x compile` 命令默认传递 '`-k`' 标志的原因。

## 9.7 临时文件

在某些情况下，*make* 需要创建自己的临时文件。*make*，包括所有递归调用的 *make* 实例，在运行时不得干扰这些文件。

如果设置了环境变量 `MAKE_TMPDIR`，则 *make* 创建的所有临时文件都将放置在那里。

如果未设置 `MAKE_TMPDIR`，则将使用当前操作系统的临时文件的标准位置。对于 POSIX 系统，这将是在 `TMPDIR` 环境变量中设置的位置，否则将使用系统默认位置（例如 `/tmp`）。在 Windows 上，首先检查 `TMP`，然后检查 `TEMP`，然后检查 `TMPDIR`，最后使用系统默认临时文件位置。

请注意，此目录必须已经存在，否则 *make* 将失败：*make* 不会尝试创建它。

这些变量不能从 makefile 中设置：GNU *make* 必须在开始读取 makefile 之前可以访问到此位置。

## 9.8 选项汇总
以下是 *make* 理解的所有选项的表格：

- `-b`<br>`-m`
    为了与其他版本的make兼容，将忽略这些选项。

- `-B`<br>`--always-make`
    认为所有目标都已过时。GNU *make* 继续使用正常算法来考虑目标及其先决条件；然而，无论其先决条件的状态如何，所考虑的所有目标都会重新生成。为了避免无限递归，如果将 `MAKE_RESTARTS`（请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)）设置为大于 0 的数字，则在考虑是否重新生成 *makefile* 时会禁用此选项（请参阅 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/make.html#Remaking-Makefiles)）。

- `-C dir`<br>`--directory=dir`
    在读取 makefile 之前更改到目录 *dir*。如果指定了多个 “`-C`” 选项，则每个选项都相对于前一个进行解释：`-C / -C etc` 等价于 `-C /etc`。这通常与 make 的递归调用一起使用。参阅 [5.7 Recursive Use of make](https://www.gnu.org/software/make/manual/make.html#Recursion)

- `-d`
    除了正常处理之外，还打印调试信息。调试信息说明正在考虑重新创建哪些文件，正在比较哪些文件时间，与什么结果进行比较，哪些文件实际上需要重新创建，考虑了哪些隐式规则，应用了哪些规则——关于make如何决定做什么的所有有趣的事情。`-d`选项等价于'`--debug=a`'（见下文）。

- `--debug[=options]`
    除正常处理外，还打印调试信息。可以选择各种级别和类型的输出。在没有参数的情况下，打印 “basic” 级别的调试。可能的参数如下; 只考虑第一个字符，并且值必须以逗号或空格分隔。
    * a (all)
        启用所有类型的调试输出。这相当于使用'`-d`'。
    * b (basic)
        基本调试打印每个被发现过期的目标，以及构建是否成功。
    * v (verbos)
        “basic”之上的级别；包括有关解析了哪些 makefile 的消息、不需要重建的先决条件等。此选项还启用“basic”消息。
    * i (implicit)
        打印描述每个目标的隐式规则搜索的消息。此选项还启用“basic”消息。
    * j (jobs)
        打印提供有关调用特定子命令的详细信息的消息。
    * m (makefile)
        默认情况下，在尝试重新生成 makefil e时不会启用上述消息。此选项也会在重建 makefile 时启用消息。请注意，“all”选项确实启用了此选项。此选项还启用了“basic”消息。
    * p (print)
        打印要执行的配方，即使配方通常是静默的(由于 `.SILENT` 或 ‘`@`’). 还打印定义配方的 makefile 名称和行号。
    * w (why)
        通过显示哪些先决条件比目标更新，解释为什么必须重新创建每个目标。
    * n (none)
        禁用当前启用的所有调试。如果在此之后遇到其他调试标志，它们仍然会生效。

- `-e`<br>`--environment-overrides`
    从环境中获取的变量优先于 makefile 中的变量。参阅 [6.10 Variables from the Environment](https://www.gnu.org/software/make/manual/make.html#Environment)

- `-E string`<br>`--eval=string`
    评估 *string* 作为makefile语法的正确性。这是 eval 函数的命令行版本 (请参阅 [8.10 The eval Function](https://www.gnu.org/software/make/manual/make.html#Eval-Function))。在定义默认规则和变量之后、但在读取任何 makefile 之前执行评估。

- `-f file`<br>`--file=file`<br>`--makefile=file`
    将名为 *file* 的文件作为 makefile 读取。参阅 [3 Writing Makefiles](https://www.gnu.org/software/make/manual/make.html#Makefiles)

- `-h`<br>`--help`
    提醒您 make 理解的选项，然后退出。

- `-i`<br>`--ignore-errors`
    忽略为重制作文件而执行的配方中的所有错误。参阅 [5.5 Errors in Recipes](https://www.gnu.org/software/make/manual/make.html#Errors)

- `-I dir`<br>`--include-dir=dir`
    指定用于搜索包含的 makefile 的目录 *dir*。请参阅 [3.3 Including Other Makefiles](https://www.gnu.org/software/make/manual/make.html#Include)。如果使用多个“`-I`”选项来指定多个目录，则按照指定的顺序搜索这些目录。如果目录 *dir* 是单个破折号（-），则将丢弃到该点之前的任何已指定目录（包括默认目录路径）。您可以通过 `.INCLUDE_DIRS` 变量检查要搜索的当前目录列表。

- `-j [jobs]`<br>`--jobs[=jobs]`
    指定要同时运行的配方 (作业) 数。在没有参数的情况下，make 会同时运行尽可能多的配方。如果有多个 '-j' 选项，最后一个是有效的。有关如何运行配方的更多信息，请参见 [5.4 Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel)。请注意，在 MS-DOS 上忽略此选项。

- `--jobserver-style=[style]`
    选择要使用的 jobserver 的样式。此选项仅在启用并行构建时才有效 (请参阅 [5.4 Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel))。在 POSIX 系统上，*style* 可以是 fifo(默认) 或 pipe 之一。在 Windows 上，唯一可接受的样式是 sem(默认)。如果您需要使用旧版本的GNU make，或者需要特定 jobserver 样式的其他工具，则此选项非常有用。

- `-k`<br>`--keep-going`
    出错后尽可能继续。虽然失败的目标和依赖于它的目标不能被重做，但是这些目标的其他先决条件可以被完全相同地处理。参阅 [9.6 Testing the Compilation of a Program](https://www.gnu.org/software/make/manual/make.html#Testing)

- `-l [load]`<br>`--load-average[=load]`<br>`--max-load[=load]`
    指定如果有其他配方正在运行且平均负载至少为 *load* (浮点数)，则不应启动任何新配方。如果没有参数，则移除先前的负载限制。参阅 [5.4 Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel)

- `-L`<br>`--check-symlink-times`
    在支持符号链接(symbolic links)的系统上，此选项使 *make* 除了考虑任何符号链接上的时间戳以外，还考虑 符号链接引用的文件上的时间戳。提供此选项时，文件和符号链接两者中的最新时间戳将作为此目标文件的修改时间。

- `-n`<br>`--just-print`<br>`--dry-run`<br>`--recon`
    打印将要执行的配方，但不要执行它（除非在某些情况下）。参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/make.html#Instead-of-Execution)

- `-o file`<br>`--old-file=file`<br>`--assume-old=file`
    即使文件 *file* 比其先决条件更旧也不要重做, 也不要因为 *file* 的更改而重做任何东西。本质上，文件被视为非常旧，其规则被忽略。参阅 [9.4 Avoiding Recompilation of Some Files](https://www.gnu.org/software/make/manual/make.html#Avoiding-Compilation)

- `-O[type]`<br>`--output-sync[=type]`

- `-p`<br>`--print-data-base`
    打印读取 makefile 产生的数据库（规则和变量值）；然后照常执行或按其他指定执行。这也打印 `-v` 开关给出的版本信息（见下文）。要打印数据库而不尝试重新制作任何文件，请使用 `make-qp`。要打印预设规则和变量的数据库，请使用 `make -p -f /dev/null`。数据库输出包含配方和变量定义的文件名和行号信息，因此它可以成为复杂环境中有用的调试工具。

- `-q`<br>`--question`
    “提问模式”。不会运行任何配方或打印任何内容；仅仅返回退出状态：如果指定的目标已经是最新的，则返回为零，如果需要重新制作，则返回 1，如果遇到错误，则返回 2。参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/make.html#Instead-of-Execution)

- `-r`<br>`--no-builtin-rules`
    消除对内置隐式规则的使用（请参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)）。您仍然可以通过编写模式规则来定义自己的规则（请参阅 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules)）。“`-r`” 选项还清除了后缀规则的默认后缀列表（请参阅 [10.7 Old-Fashioned Suffix Rules](https://www.gnu.org/software/make/manual/make.html#Suffix-Rules)）。但是您仍然可以使用 `.SUFFIXES` 的规则定义自己的后缀，然后定义自己的后缀规则。请注意，只有**规则**受到 `-r` 选项的影响；默认变量仍然有效（请参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)）；请参阅下面的“`-R`”选项。

- `-R`<br>`--no-builtin-variables`
    消除使用内置规则特定变量（请参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)）。当然，您仍然可以定义自己的变量。“`-R`” 选项也会自动启用 “`-r`” 选项（见上文），因为没有对隐式规则使用的变量进行任何定义的隐式规则是没有意义的。

- `-s`<br>`--silent`<br>`--quiet`
    静默操作；不要在执行配方时打印配方。参阅 [5.2 Recipe Echoing](https://www.gnu.org/software/make/manual/make.html#Echoing)。

- `-S`<br>`--no-keep-going`<br>`--stop`
    取消 '`-k`' 选项的效果。这永远是不必要的，除非在递归 *make* 中，'`-k`'可能通过变量 `MAKEFLAGS` 从顶级 make 继承（请参阅 [5.7 Recursive Use of make](https://www.gnu.org/software/make/manual/make.html#Recursion)），或者如果您在环境中的 `MAKEFLAGS` 中设置'`-k`'。

- `--shuffle[=mode]`
    此选项使能对先决条件关系进行模糊测试。当启用并行性（`-j`）时，构建目标的顺序变得不那么确定。如果在 makefile 中没有完全声明先决条件，这可能会导致间歇性且难以追踪的构建失败。
    '`--shuffle`' 选项强制 *make* 有目的地重新排序终点目标和先决条件，以便目标和先决条件之间的关系仍然有效，但给定目标的先决条件的排序被重新排序，如下所述。
    此选项不会更改在自动变量中列出先决条件的顺序。
    `.NOTPARALLE` 伪目标(pseudo-target)禁用该 makefile 的洗牌。还有任何包含 `.WAIT` 的先决条件列表将不会被洗牌。请参阅 [5.4.1 Disabling Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel-Disable)。
    `--shuffle=` 选项接受这些值：
    * random
        随机选择一个种子用于随机洗牌。如果未指定模式，则默认为这个。所选种子也提供给sub-make命令。该种子包含在错误消息中，以便在以后的运行中可以重复使用，以重现问题或验证问题是否已解决。

    * reverse
        颠倒目标和先决条件的顺序，而不是随机洗牌。

    * seed
        使用使用指定种子值初始化的 “`random`” 洗牌。种子是一个整数。

    * none
        禁用洗牌。这否定了任何以前的'`--shuffle`'选项。

- `-t`<br>`--touch`
    touch 文件 (将它们标记为最新，而无需真正更改它们)，而不是运行它们的配方。这是用来假装配方已经完成，以愚弄未来的 *make* 调用。参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/make.html#Instead-of-Execution)

- `--trace`
    显示用于执行 *make* 的跟踪信息。`-trace` 是 `--debug=print,why` 的简写

- `-v`<br>`--version`
    打印 make 程序的版本、版权，作者列表、没有授权的通知， 然后退出。

- `-w`<br>`--print-directory`
    在执行 makefile 之前和之后打印包含工作目录的消息。这对于跟踪来自复杂的、嵌套的递归 *make* 命令错误可能很有用。请参阅 [5.7 Recursive Use of make](https://www.gnu.org/software/make/manual/make.html#Recursion)。（实际上，您很少需要指定此选项，因为 *make* 会为您指定；请参阅 [5.7.4 The ‘--print-directory’ Option](https://www.gnu.org/software/make/manual/make.html#g_t_002dw-Option)）

- `--no-print-directory`
    禁用 `-w` 下的对工作目录的打印。当 `-w` 自动打开时，但您不希望看到额外的消息时，此选项很有用。参阅 [5.7.4 The ‘--print-directory’ Option](https://www.gnu.org/software/make/manual/make.html#g_t_002dw-Option)

- `-W file`<br>`--what-if=file`<br>`--new-file=file`<br>`--assume-new=file`
    假设目标 *file* 刚刚被修改。当与 '`-n`' 标志一起使用时，这会显示如果您要修改该文件会发生什么。如果没有 '`-n`'，它几乎与在运行make之前对给定文件运行 `touch` 命令相同，只是修改时间仅在 *make* 的想象中更改。参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/make.html#Instead-of-Execution)

- `--warn-undefined-variables`
    每当 make 看到对未定义变量的引用时发出警告消息。当您尝试调试以复杂方式使用变量的 makefile 时，这会很有帮助。


# 10 Using Implicit Rules

我们经常使用某些重新制作目标文件的标准方法。例如，制作目标文件的一种常用方法是使用C编译器 *cc* 从C源文件中获取。

*隐式规则*(*Implicit rules*) 告诉 *make* 如何使用惯用技术，以便您在想要使用它们时不必详细指定它们。例如，C编译有一个隐式规则。文件名决定运行哪些隐式规则。例如，C编译通常使用 *.c* 文件生成 *.o* 文件。因此，*make* 在看到这种文件名结尾组合时应用C编译的隐式规则。

一系列隐式规则可以按顺序应用。例如，*make* 从一个 *.y* 文件，借助 *.c* 文件，重新制作 *.o* 文件

内置的隐式规则在其配方中使用了几个变量，因此，通过更改变量的值，可以更改隐式规则的工作方式。例如，变量 `CFLAGS` 控制C编译的隐式规则下给C编译器的 标志(flags) 。

您可以通过编写 _pattern rules_ 来定义自己的隐式规则。

后缀规则( _suffix fules_ )是一种定义隐式规则的更有限的方法。模式规则( _pattern rules_ )更通用、更清晰，但为了兼容性保留了后缀规则。

## 10.1 使用隐式规则

为了让 make 找到更新目标文件的惯用的方法，你所要做的就是不要自己指定 recipes。要么写一个没有 recipes 的规则，要么根本不写规则。

例如，假设makefile如下所示：

```makefile
foo : foo.o bar.o
    cc -o foo foo.o bar.o $(CFLAGS) $(LDFLAGS)
```

因为您提到 *foo.o* 但没有给出规则，*make* 将自动查找用于更新它的隐式规则。无论文件 *foo.o* 当前是否存在，都会发生这种情况。

如果找到隐式规则，它可以同时提供配方和一个或多个先决条件（源文件）。如果您需要指定隐式规则无法提供的附加条件（例如头文件），您可能希望为 *foo.o* 编写一个没有配方的规则。


每个隐式规则都有一个 _目标模式_(target pattern) 和 _先决条件模式_(prerequisite patterns) 。可能有许多隐式规则具有相同的 _目标模式_。例如，许多规则生成 “. o” 文件：一个来自带有C编译器的 “.c” 文件；另一个来自带有Pascal编译器的 “.p” 文件；等等。实际适用的规则是其先决条件存在或可以创建的规则。因此，如果您有一个文件 *foo.c*，make将运行C编译器；否则，如果您有一个文件 *foo.p*，make将运行Pascal编译器；等等。

当然，当您编写makefile时，您知道要使用哪个隐式规则，并且您知道它会选择那个，因为您知道哪些可能的先决条件文件应该存在。有关所有预定义隐式规则的目录，请参阅 [10.2 Catalogue of Built-In Rules](https://www.gnu.org/software/make/manual/make.html#Catalogue-of-Rules)。

上面，我们说如果所需的先决条件“存在或可以创建”，则隐式规则适用。如果文件在makefile中作为目标或先决条件明确提及，或者如果可以递归地找到隐式规则来创建它，则文件“可以创建”。当隐式先决条件是另一个隐式规则的结果时，我们说链接(chaining)正在发生。参见 [10.4 Chains of Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Chained-Rules)。

通常，*make* 会为每个没有配方的目标、每个没有配方的双冒号规则搜索一个隐式规则。一个仅作为先决条件提及的文件会被视为规则未指定任何内容的目标（target），因此会对其进行隐式规则搜索。有关如何完成搜索的详细信息，请参阅 [10.8 Implicit Rule Search Algorithm](https://www.gnu.org/software/make/manual/make.html#Implicit-Rule-Search)。

请注意，显式先决条件不会影响隐式规则搜索。例如，考虑这个显式规则

```makefile
foo.o: foo.p
```

*foo.p* 上的先决条件不一定意味着 *make* 将根据从Pascal源文件(.p 文件)生成目标文件(.o 文件)的隐含规则重新生成 *foo.o*。例如，如果 *foo.c* 也存在，则使用C源文件制作目标文件的隐式规则代替，因为它出现在预定义隐式规则列表中的 Pascal 规则之前（请参阅 [10.2 Catalogue of Built-In Rules](https://www.gnu.org/software/make/manual/make.html#Catalogue-of-Rules)）。

如果您不希望将隐式规则用于没有配方的目标，您可以通过编写一个分号的方式，为该目标提供一个空配方

## 10.2 内置规则目录

这是一个预定义的隐式规则目录，除非 makefile 显式地覆盖或取消它们，否则这些规则始终可用。有关取消或覆盖隐式规则的信息，参阅 [10.5.6 Canceling Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Canceling-Rules) . `-r` 或 `--no-builtin-rules` 选项取消所有预设规则。

本手册仅记录了基于 POSIX 的操作系统上可用的默认规则。其它操作系统，如 VMS, Windows, OS/2等有不同的默认规则集，在终端中使用 `make -p` 以查看您的 GNU make 版本中可用的默认规则和变量的完整列表。

即使没有给出 “`-r`” 选项，这些规则也并非所有都会被定义。许多预定义的隐式规则在 *make* 中作为后缀规则实现，因此将定义哪些规则取决于*后缀列表*(suffix list)（特殊目标 `.SUFFIXES` 的先决条件列表）。默认的*后缀列表*是：`.out, .a, .ln, .o, .c, .cc, .C, .cpp, .p, .f, .F, .m, .r, .y, .l, .ym, .lm, .s, .S, .mod, .sym, .def, .h, .info, .dvi, .tex, .texinfo, .texi, .txinfo, .w, .ch .web, .sh, .elc, .el`。下面描述的所有先决条件具有这些后缀之一的隐式规则实际上都是*后缀规则*。如果修改后缀列表，唯一有效的预定义*后缀规则*将是由您指定的列表中的一个或两个后缀命名的规则；后缀不在列表中的规则将被禁用。有关*后缀规则*的完整详细信息，请参阅 [10.7 Old-Fashioned Suffix Rules](https://www.gnu.org/software/make/manual/make.html#Suffix-Rules)。

**Compiling C programs**
_n.o_ 是使用如 `$(CC) $(CPPFLAGS) $(CFLAGS) -c` 形式的 recipe 从 _n.c_ 自动制作的

**Compiling C++ programs**
*n.o* 由 *n.cc*、*n.cpp* 或 *n.c* 使用形式为 `$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c`的配方自动生成。我们鼓励您对 C++ 源文件使用后缀 '.cc' 或 '.cpp'，而不是 '.c'，以更好地支持不区分大小写的文件系统。

**Compiling Pascal programs**

**Compiling Fortran and Ratfor programs**

**Preprocessing Fortran and Ratfor programs**

**Compiling Modula-2 programs**

**Assembling and preprocessing assembler programs**

**Linking a single object file**
_n_ 是通过运行C编译器链接程序从 *n.o* 自动生成的。使用的精确 recipe 是 `$(CC) $(LDFLAGS) n.o $(LOADLIBES) $(LDLIBS)` 。

对于只有一个源文件的简单程序，此规则会做出正确的操作。如果有多个目标文件（可能来自各种其他源文件），其中一个具有与可执行文件匹配的名称，它也会做正确的操作。因此：

```makefile
x: y.o z.o
```

当 `x.c`, `y.c` `z.c` 都存在时会执行：

```sh
cc -c x.c -o x.o
cc -c y.c -o y.o
cc -c z.c -o z.o
cc x.o y.o z.o -o x
rm -f x.o
rm -f y.o
rm -f z.o
```

**在更复杂的情况下，例如当没有名称源自可执行文件名的目标文件时，您必须为链接编写显式配方。**

每一种自动制作成 “*.o*” 目标文件的文件都将通过使用编译器（`$(CC)`、`$(FC)` 或 `$(PC)`；C编译器 `$(CC)` 用于组装 *.s* 文件）被自动链接，而不需要 `-c` 选项。这可以通过使用 “*.o*” 目标文件作为中介来完成，但是一步完成编译和链接更快，所以这就是它的完成方式。(译者注，在C语言中，一个 .c 文件要经历 编译、链接 这两个过程，当然还有预处理、汇编这两个过程，但这编译、链接都是通过同一个编译器来进行的，与其先生成 .o 再链接，不如一步完成来的更有效率)

**Yacc for C programs**

**Lex for C programs**

**Lex for Ratfor programs**

**Making Lint Libraries from C, Yacc, or Lex programs**

**TeX and Web**

**Texinfo and Info**

**RCS**

**SCCS**

通常，您想更改的仅仅是上表中列出的变量，这些变量记录在下一节中。

但是，内置隐式规则中的配方实际上使用了 `COMPILE.c`、`LINK.p` 和 `PREPROCESS.S` 等变量，其值包含上面列出的配方。

*make* 遵循约定——编译 *.x* 源文件的规则使用变量 `COMPILE.x`。类似地，从 *.x* 文件生成可执行文件的规则使用 `LINK.x`；预处理 *.x* 文件的规则使用 `PREPROCESS.x`.。

生成目标文件的每个规则都使用变量 `OUTPUT_OPTION`。*make* 将此变量定义为包含 "`-o $@`"，或者为空，具体取决于编译时的选项。您需要 “`-o`” 选项来确保当源文件位于不同目录时，输出进入正确的文件，正如使用 `VPATH` 时（请参阅 [4.5 Searching Directories for Prerequisites](https://www.gnu.org/software/make/manual/make.html#Directory-Search)）。但是，某些系统上的编译器不接受目标文件的 “`-o`” 开关。如果您使用这样的系统并使用 `VPATH`，某些编译会将其输出放在错误的位置。解决此问题的一个可能的解决方法是将 `; mv $*.o $@` 赋值给 `OUTPUT_OPTION`。

## 10.3 隐式规则所使用的变量

内置隐式规则中的配方可以自由使用某些预定义变量。您可以在 *makefile* 中更改这些变量的值，结合传递给 make 的参数，或在环境中的参数，更改隐式规则的工作方式，而无需重新定义规则本身。使用 `-R` 或 `--no-builtin-variables` 选项可取消隐式规则使用的所有变量。

例如，用于编译C源文件的配方实际上是 "`$(CC) -c $(CFLAGS) $(CPPFLAGS)`"。使用的变量的默认值是 "*cc*"和空值，导致命令其实是 "`cc -c`"。通过将 "*cc*" 重新定义为 "*ncc*"，您可以使 "*ncc*" 用于隐式规则执行的所有C编译。通过将 "`CFLAGS`" 重新定义为 "*-g*"，您可以将 "*-g*" 选项传递给每个编译。所有进行C编译的隐式规则都使用 "`$(CC)`"来获取编译器的程序名称，并且都在给编译器的参数中包含 "`$(CFLAGS)`"。

隐式规则中使用的变量分为两类：程序名称（如`CC`）和包含程序参数的变量（如`CFLAGS`）。（“程序名称”也可能包含一些命令参数，但它必须以实际的可执行程序名称开头。）如果变量值包含多个参数，用**空格**分隔它们。

下表描述了一些更常用的预定义变量。此列表并非详尽无遗，此处显示的默认值可能不是您的环境下 *make* 选择的值。要查看GNU *make* 实例的预定义变量的完整列表，您可以在没有 *makefile* 的目录中运行 `make -p`。

以下是一些在内置规则中用作**程序名称**的更常见变量的表：

**AS**
编译汇编文件(assembly files)的程序；默认为 "*as*"。

**CC**
编译 C 代码的程序；默认为 "*cc*"。

**CXX**
用于编译 C++ 程序的程序；默认为 "*g++*"。

**CPP**
运行 C 预处理器的程序，结果会传递给标准输出；默认为 "`$(CC) -E`"。

**YACC**
用于将 Yacc 语法转换为源代码的程序；默认 "*yacc*"。

**LINT**
用于在源代码上运行 lint 的程序；默认的 "*lint*"。

**RM**
删除文件的命令；默认为 "`rm -f`"。

以下是变量表，其值是上述程序的附加参数。除非另有说明，否则所有这些的**默认值都是空字符串**。

**ASFLAGS**
给汇编器的额外标志（在 ".s" 或 ".S" 文件上显式调用时）。

**CFLAGS**
给 C 编译器的额外标志。

**CXXFLAGS**
给 C++ 编译器的额外标志。

**CPPFLAGS**
提供给 C 预处理器和使用它的程序（C 和 Fortran 编译器）的额外标志。

**LDFLAGS**
当编译器应该调用链接器 "`ld`" 时，要给编译器的额外标志，例如`-L`。然而库（*-lfoo*）应该添加到 _LDLIBS_ 变量中。

**LDLIBS**
当编译器应该调用链接器"`ld`"时，给编译器的库标志或名称。`LOADLIBES`已弃用（但仍受支持），替代方案是 `LDLIBS`。非库链接器标志，例如 `-L`，应该放在 `LDFLAGS` 变量中。

**LINTFLAGS**
给 lint 的额外的标志。

## 10.4 隐式规则链

有时可以通过一系列隐式规则制作文件。例如，可以通过首先运行 Yacc 然后运行 cc 从 *n.y* 制作文件 *n.o* 。这样的序列称为链（*chain*）。

如果文件 *n.c* 存在，或者在 *makefile* 中提到，则不需要进行特殊搜索：*make* 发现目标文件可以通过 C 编译从 *n.c* 制作；稍后，在考虑如何制作 *n.c* 时，*make* 运行 Yacc 的规则。最终更新了 *n.c* 和 *n.o*。

然而，即使 *n.c* 不存在并且没有被提及，*make* 也知道如何将其设想为 *n.o* 和 *n.y* 之间缺失的链接！在这种情况下，*n.c* 被称为**中间文件**(intermediate file)。一旦 *make* 决定使用中间文件，*make* 就会像 *n.c* 在 *makefile* 中提到过一样，伴随着一个说明如何创建 *n.c* 的隐式规则，进入到数据库中。

中间文件使用它们的规则重新制作，就像所有其他文件一样。但是与其它文件相比，中间文件有两种不同的处理方式。

第一个不同之处是如果中间文件不存在会发生什么。如果一个普通的文件 b 不存在，并且 *make* 认为一个目标依赖于 b，则 *make* 总是创建 b，然后从 b 更新目标。但是，如果 b 是一个中间文件，那么 *make* 会直接忽视它: 除非它(译者注，我理解的这个它代指文件 b)的先决条件之一是过时的，否则 *make* 不会创建b。这意味着依赖于 b 的目标也不会被重建，除非有其他原因更新该目标: 例如目标不存在或不同的先决条件比目标更新。

第二个不同之处是，如果 make 为了更新其他东西而确实创建了 b，它会在不再需要 b 后删除 b。因此，在 `make` 之前不存在的中间文件在 `make` 之后也不存在。*make* 通过打印一个 `rm` 命令向您报告删除情况，该命令显示它正在删除哪个文件。

您可以通过将文件列为特殊目标 "`.INTERMEDIATE`" 的先决条件来显式地将其标记为中间文件。即使文件以其他方式显式提及，这也会生效。

如果文件在 *makefile* 中作为目标或先决条件被提及，则该文件不能是中间文件，因此避免删除中间文件的一种方法是将其作为先决条件添加到某个目标。然而，这样做可能会导致 *make* 在搜索模式规则时做额外的工作（请参阅 [10.8 Implicit Rule Search Algorithm](https://www.gnu.org/software/make/manual/make.html#Implicit-Rule-Search)）。

作为替代方案，列出文件作为特殊目标 “`.NOTINTERMEDIATE`” 的先决条件会迫使它不被视为中间 (就像任何其他提及文件一样)。此外，将*模式规则*的*目标模式*列为 “`.NOTINTERMEDIATE`” 的先决条件,可确保使用该*模式规则*生成的目标不会被视为中间目标。

您可以通过提供 “`.NOTINTERMEDIATE`” 作为没有先决条件的目标，在 *makefile* 中完全禁用中间文件: 在这种情况下，它适用于 *makefile* 中的每个文件。

如果您不希望 make 仅仅因为文件不存在而创建该文件，但也不希望 make 自动删除该文件，则可以将其标记为 **次要文件**(secondary file)。为此，将其列为特殊目标 “`.SECONDARY`” 的先决条件。将文件标记为**次要文件**也会将其标记为中间文件。

一条**链**可以调用两个以上的隐式规则。例如，可以通过运行 RCS、Yacc 和 cc 从 *RCS/foo.y,v* 制作文件 *foo*。那么 *foo.y* 和 *foo.c* 都是最后删除的中间文件。

没有一条隐式规则可以在**链**中出现多次。这意味着 *make* 甚至不会考虑通过运行两次链接器从 *foo.o.o* 制作 *foo* 这样荒谬的事情。这个约束还有一个额外的好处，那就是防止在搜索隐式规则链时出现任何无限循环。

有一些特殊的隐式规则来优化某些情况，否则这些情况将由规则链处理。例如，*foo.c* 可以通过使用 *foo.o* 作为中间文件、使用单独的链式规则编译和链接，来制作 *foo*。但实际上发生的是，这种情况的特殊规则使用单个 cc 命令进行编译和链接。优化后的规则优先于分步链使用，因为它在规则排序中出现得更早。

最后，出于性能原因，*make* 不会考虑非终端 match-anything rules（即 '`%:`') 在搜索规则以构建隐式规则的先决条件时（请参阅 [10.5.5 Match-Anything Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Match_002dAnything-Rules)）。

## 10.5 定义和重定义模式规则（pattern rules）

您可以通过编写**模式规则**来定义**隐式规则**。**模式规则**( *pattern rule* )与普通规则不同之处在于,其目标包含字符 `%` （仅有一个）。目标被认为是匹配文件名的模式；“%”可以匹配任何非空子字符串，而其他字符只能匹配它们自己。先决条件同样使用 `%` 来显示它们的名称与目标名称的关系。

因此，模式规则如 `%.o : %.c` 说明如何从另一个文件 *stem.c* 制作任何文件 *stem.o* （stem 词干）

请注意，在**模式规则**中使用 "`%`" 的展开发生在任何变量或函数展开**之后**，变量或函数展开发生在读取 *makefile* 时。详细见 [6 How to Use Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables) 和 [8 Functions for Transforming Text](https://www.gnu.org/software/make/manual/make.html#Functions) 。

### 10.5.1 模式规则介绍

模式规则在目标中包含字符 `%`（仅有一个）；否则，它看起来与普通规则完全一样。目标是匹配文件名的模式；"`%`" 匹配任何非空子字符串，而其他字符只匹配它们自己。

例如，"*%.c*" 作为模式，匹配任何以 "*.c*" 结尾的文件名。"*s.%.c*" 作为模式，匹配任何以 "*s.*" 开头、以 "*.c*" 结尾且长度至少为五个字符的文件名（必须至少有一个字符匹配 "`%`".)。 "`%`" 匹配的子字符串称为词干（stem）。

**模式规则**的先决条件中的 "`%`" 代表与目标中的 "`%`" 匹配的相同词干。为了**模式规则**的应用，其**目标模式**必须与考虑的文件名匹配，并且其所有先决条件（模式替换后）必须命名存在或可以创建的文件。这些文件成为目标的先决条件。

因此，这样形式的规则

```Makefile
%.o : %.c ; recipe…
```

指定如何以另一个文件 *n.c* 作为先决条件，创建文件 *n.o*，前提是 *n.c* 存在或可以创建。

也可能有不使用 "`%`" 的先决条件；这样的先决条件附加到此**模式规则**创建的每个文件。这些不变的先决条件偶尔很有用。

**模式规则**不需要任何包含 "`%`" 的先决条件，或者实际上根本不需要任何先决条件。这样的规则实际上是一个通用通配符。它提供了一种创建与**目标模式**匹配的任何文件的方法。请参阅 [10.6 Defining Last-Resort Default Rules](https://www.gnu.org/software/make/manual/make.html#Last-Resort)。

可能有多个**模式规则**与同一目标匹配。在这种情况下，make将选择“最适合”规则。请参阅 [10.5.4 How Patterns Match](https://www.gnu.org/software/make/manual/make.html#Pattern-Match)。

**模式规则**可能有多个目标；但是，每个目标必须包含一个 "`%`" 字符。**模式规则**中的多个目标模式始终被视为组目标（请参阅 [4.10 Multiple Targets in a Rule](https://www.gnu.org/software/make/manual/make.html#Multiple-Targets)），无论它们是否使用 `:` 或 `&:` 分隔符。

有一个例外：如果一个 *模式目标* 过期或不存在并且 *makefile* 不需要构建它，那么它不会导致其他目标被认为过期（这个历史异常将在 GNU *make* 的未来版本中被删除，不要这样写你的 *makefile*）。如果检测到这种情况make将生成一个警告 "pattern recipe did not update peer target"；但是 *make* 无法检测到所有这样的情况。

### 10.5.2 模式规则举例

以下是 *make* 中实际预定义的**模式规则**的一些示例。首先，将 "*.c*" 文件编译为 "*.o*" 文件的规则：

```makefile
%.o : %.c
    $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

定义一个规则，可以从 *x.c* 生成任何文件 *x.o*。该配方使用自动变量 "`$@`" 和 "`$<`" 在规则适用的每种情况下替换目标文件和源文件的名称。参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)

这是第二个内置规则：

```makefile
% :: RCS/%,v
    $(CO) $(COFLAGS) $<
```

定义一个规则，该规则可以从子目录RCS中的相应文件 *x,v* 生成任何文件 *x*。由于目标是 "%" ，只要存在适当的先决条件文件，此规则将适用于任何文件。双冒号是规则结束，这意味着它的先决条件可能不是中间文件。参阅 [10.5.5 Match-Anything Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Match_002dAnything-Rules)。

此模式规则有两个目标：

```makefile
%.tab.c %.tab.h: %.y
    bison -d $<
```

这告诉 *make*，内容为 "`bison -d x.y`" 的配方可以制作 "*x.tab.c*" "*x.tab.h*" 这两个内容。如果文件 *foo* 依赖于 "*parse.tab.o*"、"*scan.o*" 以及 文件 "*scan.o*" 依赖于 "*parse.tab.h*"，当 "*parse.y*" 发生改变时，那么内容为 "`bison -d parse.y`" 的配方仅会被执行一次，"*parse.tab.o*" 和 "*scan.o*" 的先决条件就会被满足。（根据推测，"*parse.tab.o*" 会从 "*parse.tab.c*" 被重新编译，"*scan.o*" 会从 "*scan.c*" 被重新编译，"*foo*" 会从 "*parse.tab.o*", "*scan.o*" 和他的其它先决条件连接而来，之后它将会顺利地执行）

### 10.5.3 自动变量

假设您正在编写一个模式规则，将一个 “*.c*” 文件编译成一个 “*.o*” 文件：您如何编写 “`cc`”命令，以便它在正确的源文件名上运行？您不能在 recipe 中写入名称，因为每次应用隐式规则时名称都不同。

您所做的是使用 make 的一个特殊功能，即自动变量(Automatic Variables)。这些变量具有根据规则的目标和先决条件为执行的每个规则重新计算的值。在此示例中，您将使用 "`$@`" 作为目标文件名，使用 "`$<`" 作为源文件名。

认识到自动变量值可用的范围是有限的非常重要：它们仅在配方中有值。特别是，您不能在规则的目标列表中使用它们，它们在那里没有值，并且会展开为空字符串。此外，它们不能直接在规则的先决条件列表中被访问。一个常见的错误是尝试在先决条件列表中使用 `$@`,这是行不通的。但是，GNU make有一个特殊的功能，二次扩展（请参阅 [3.9 Secondary Expansion](https://www.gnu.org/software/make/manual/make.html#Secondary-Expansion)），它将允许在先决条件列表中使用自动变量值。

自动变量如下表所示：

- `$@`
	规则(rule)的目标(target)的文件名。
	如果目标是存档成员，则`$@`是存档文件的名称。在具有多个目标的模式规则（[10.5.1 Introduction to Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Intro)）中， `$@` 是导致规则的配方(recipe)运行的目标名称。
- `$%`
	当目标是存档成员时，该符号表示目标成员的名称。见 [11 Using make to Update Archive Files](https://www.gnu.org/software/make/manual/make.html#Archives)。	例如，目标是 *foo.a(bar.o)* ，则 `$%` 代表 bar.o， `$@` 是 foo.a 。	当目标不是存档成员时， `$%` 是空。
- `$<`
	第一个先决条件的名称。如果目标通过隐式规则获取指令，则`%<`是通过隐式规则（参阅 [10 Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)）添加的第一个先决条件。

- `$?`
	新于目标的所有先决条件的名称，它们之间有空格。如果目标不存在，所有先决条件都将包括在内。当先决条件是存档成员时，仅使用命名成员（见 [11 Using make to Update Archive Files](https://www.gnu.org/software/make/manual/make.html#Archives)）。
	当想仅对已更改的先决条件进行操作时，`$?`即使在显式规则中也很有用。例如，假设一个名为 *lib* 的存档应该包含多个目标文件的副本。此规则仅将更改的目标文件复制到存档中：
    ```makefile
    lib: foo.o bar.o lose.o win.o
        ar r lib $?
    ```

- `$^`
	所有先决条件的名称，它们之间有空格。
	当先决条件是存档成员时，只使用被指名的成员（见 [11 Using make to Update Archive Files](https://www.gnu.org/software/make/manual/make.html#Archives)）。无论每个文件作为先决条件列出多少次，一个目标在它所依赖的每个文件上只有一个先决条件。因此，如果您多次列出目标的先决条件，`$^` 的值只包含该名称的一个副本。此列表不包含任何*仅声明顺序的先决条件*(order-only prerequisites)；对于这些先决条件，请参阅下面的 `$|` 变量。
- `$+`
	这就像 `$^`, 但是列出多次的先决条件会按照它们在 *makefile* 中列出的顺序重复。这主要用于链接命令，在这些命令中以特定顺序重复库文件名是有意义的。
- `$|`
	所有*仅声明顺序的先决条件*(order-only prerequisites)的名称，它们之间带有空格。
- `$*`
	隐式规则匹配的词干（请参阅 [10.5.4 How Patterns Match](https://www.gnu.org/software/make/manual/make.html#Pattern-Match)）。如果目标是`dir/a.foo.b`，目标模式是 `a.%.b`，则词干是 `dir/foo`。词干对于构建相关文件的名称很有用。
    在静态模式规则(static pattern rule)中，词干是在目标模式(target pattern)中与“`%`”匹配的文件名的一部分。
	在显式规则中，没有词干；因此不能以这种方式确定`$*`。相反，如果目标名称以可识别的后缀结尾（参见 [10.7 Old-Fashioned Suffix Rules](https://www.gnu.org/software/make/manual/make.html#Suffix-Rules), 则 `$*` 被设置为目标名称减去后缀。例如，如果目标名称是 “foo. c”，则将`$*`设置为 “foo”，因为 “.c” 是后缀。GNU make 做这个奇怪的事情只是为了与make的其他实现兼容。**除了在隐式规则或静态模式规则中，您通常应该避免使用** `$*`。
	如果显式规则中的目标名称不以可识别的后缀结尾, 则在该规则中 `$*` 被设置为空字符串。

在上面列出的变量中，四个具有单个文件名的值，三个具有文件名列表的值。这七个变量具有变体，这些变体可以仅获取文件的目录名或目录中的文件名。变体变量的名称分别由附加 “D” 或 “F” 组成。函数 `dir` 和 `notdir` 可用于获得类似的效果（参见 [8.3 Functions for File Names](https://www.gnu.org/software/make/manual/make.html#File-Name-Functions)）。但是请注意，“D”变体都省略了总是出现在 `dir` 函数输出中的尾随斜杠。

- `$(@D)`
	目标文件名的目录部分，删除了尾随斜杠。如果 `$@`的值是 *dir/foo.o*，则 `$(@D)` 是 *dir*。如果`$@`不包含斜杠，则这个值是 `.` 。

- `$(@F)`
	目标文件名的“目录内文件“部分。如果 `$@` 的值是 *dir/foo.o*，则 `$(@F)` 是 *foo.o*。`$(@F)` 等价于`$(notdir $@)` 。

- `$(*D)` <br> `$(*F)`
	词干的目录部分和“目录内文件“部分；在本例中分别对应 *dir* 和 *foo*。

- `$(%D)`  <br> `$(%F)`
	目标存档成员名称的目录部分和“目录内文件“部分。这仅对 `archive(member)` 形式的存档成员目标有意义，并且仅在 `member` 可能包含目录名时有用。（请参阅 [11.1 Archive Members as Targets](https://www.gnu.org/software/make/manual/make.html#Archive-Members)）

- `$(<D)` <br> `$(<F)`
	第一个先决条件的目录部分和“目录内文件“部分。

- `$(^D)` <br> `$(^F)`
	所有先决条件的目录部分和“目录内文件“部分的列表。

- `$(+D)` <br> `$(+F)`
	所有先决条件的目录部分和“目录内文件“部分的列表，包括重复先决条件的多个实例。

- `$(?D)` <br> `$(?F)`
	新于目标的所有先决条件的目录部分和“目录内文件“部分的列表。

请注意，当我们谈论这些自动变量时，我们使用了一种特殊的风格约定；我们写 “`$<` 的值”，而不是 “变量`<`”，就像我们为 `objects` 和 `CFLAGS` 等普通变量写的那样。我们认为这种约定在这种特殊情况下看起来更自然。请不要假设它有很深的意义;`$<` 引用的是名为 `<` 的变量，就像 `$(CFLAGS)` 引用的是名为 `CFLAGS` 的变量一样。也可以用 `$(<)` 代替 `$<`。

### 10.5.4 模式(pattern)匹配的方式

目标模式由前缀、"`%`" 和后缀组成，前缀和后缀其中一个或两个可以为空。只有当文件名以前缀开头并以后缀结尾，且前后缀不会重叠时，模式才会匹配文件名。前缀和后缀之间的文本称为词干。因此，当模式 "*%.o*" 与文件名 *test.o* 匹配时，词干就是 "*test*"。模式规则的先决条件通过将字符 "`%`" 替换为词干，变成实际的文件名。因此，如果在同一示例中，其中一个先决条件写成 "*%.c*"，它将扩展为 "*test.c*"。

当目标模式不包含斜杠（通常不包含）时，文件名中的目录名在与目标前缀和后缀进行比较之前从文件名中删除。在将文件名与目标模式进行比较之后，目录名以及结束它们的斜杠，被添加到从模式规则的先决条件模式和文件名生成的先决条件文件名中。这些目录被忽略，只是为了找到非该规则被应用时的要使用的隐式规则。因此，"`e%t`" 与文件名 "*src/eat*" 匹配，"*src/a*" 会作为词干。当先决条件转换为文件名时，词干中的目录被添加到前面，而词干的其余部分被替换为 "`%`"。带有先决条件模式 "*c%r*" 的词干 "*src/a*" 给出了文件名 *src/car*。

只有当存在与文件名匹配的目标模式时，模式规则才能用于构建给定文件，并且该规则中的所有先决条件要么存在，要么可以被构建。您编写的规则优先于内置规则。但是请注意，无需链接其他隐式规则（例如，没有先决条件或其先决条件已经存在或被提及的规则）即可满足的规则总是优先于必须通过链接其他隐式规则来创建先决条件的规则。

可能有多个模式规则符合这些条件。在这种情况下，*make* 将选择具有**最短词干**的规则（即最具体匹配的模式）。如果多个模式规则具有最短词干，*make* 将选择在 *makefile* 中找到的第一个。

该算法导致更具体的规则优于更通用的规则；例如：

```makefile
%.o: %.c
    $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@

%.o : %.f
    $(COMPILE.F) $(OUTPUT_OPTION) $<

lib/%.o: lib/%.c
    $(CC) -fPIC -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```

给定这些规则并要求在 *bar.c* 和 *bar.f* 都存在的情况下构建 *bar.o*，*make* 将选择第一个规则并将 *bar.c* 编译为 *bar.o*。在 *bar.c* 不存在的相同情况下，*make* 将选择第二个规则并将 *bar.f* 编译为 *bar.o*。

如果 *make* 被要求构建 *lib/bar.o* 并且 *lib/bar.c* 和 *lib/bar.f* 都存在，则将选择第三条规则，因为该规则的词干（"*bar*"）比第一条规则的词干（"*lib/bar*"）短。如果 *lib/bar.c* 不存在，则第三条规则不符合条件，将使用第二条规则，即使词干较长。

### 10.5.5 Match-Anything 的模式规则

当**模式规则**的目标只是 "`%`" 时，它匹配任何文件名。我们称这些规则为 *match-anything* 规则。它们非常有用，但是 *make* 可能需要很多时间来考虑它们，因为它必须考虑这样的每个文件名的每个规则，是作为目标列出，还是作为先决条件列出。

假设 makefile 提到 *foo.c*。对于这个目标，make 必须考虑，是通过链接目标文件 *foo.c.o*，还是通过 C 一步完成编译并链接 *foo.c.c*，或者通过 Pascal  一步完成编译并链接 *foo.c.p* 以及许多其他可能性，来实现此文件。

我们知道这些可能性是荒谬的，因为 *foo.c* 是C源文件，而不是可执行文件。如果 *make* 确实考虑了这些可能性，它最终会拒绝它们，因为像 *foo.c.o* 和 *foo.c.p* 这样的文件不存在。但是这些可能性太多了，如果必须考虑它们，*make* 会运行得非常慢。

为了提高速度，我们在 make 考虑 *match-anything* 规则的方式上设置了各种约束。可以应用两种不同的约束，__每次定义 *match-anything* 规则时，您都必须为该规则选择两个限制其中之一__。

一种选择是通过用双冒号定义 *match-anything* 规则来将其标记为终结规则(terminal)。当规则是终结规则时，除非其先决条件确实存在，否则它不应用。可以用其他隐式规则创建的先决条件也不行。换句话说，不允许在终结规则之外进行进一步的链接。

例如，从 RCS 和 SCCS 文件中提取源的内置隐式规则是终结规则；因此，如果文件 *foo.c,v* 不存在，*make* 甚至不会考虑尝试将其作为来自 *foo.c,v.o* 或 *RCS/SCCS/s.foo.c,v* 的中间文件。RCS 和 SCCS 文件通常是最终源文件，不应该从任何其他文件中重制；因此，*make* 可以通过不寻找重制它们的方法来节省时间。

如果未将 *match-anything* 规则标记为终结规则，则该规则是非终结规则。非终结 *match-anything* 规则不能应用于隐式规则的先决条件，也不能应用于指示特定数据类型的文件名。如果某个非 *match-anything* 的隐式规则目标与文件名匹配，则文件名指示特定类型的数据。

例如，文件名 *foo.c* 与模式规则 "`%.c : %.y`"（运行 Yacc 的规则）的目标相匹配。无论该规则是否实际适用（仅当有文件 *foo.y* 时才会发生），其目标匹配的事实足以防止考虑文件 foo.c 的任何非终端 *match-anything* 规则。因此，make 甚至不会考虑尝试将 foo.c 设为 *foo.c.o*, *foo.c.c*, *foo.c.p* 等的可执行文件。

这种约束的动机是，非终端 *match-anything* 规则用于制作包含特定类型数据（例如可执行文件）的文件，并且具有可识别后缀的文件名表示其他特定类型的数据（例如C源文件）。

提供特殊的内置虚拟(dummy)*模式规则*仅用于识别某些文件名，因此不会考虑非终端 *match-anything* 规则。这些虚拟规则没有先决条件，也没有配方，并且出于所有其他目的都会被忽略。例如，内置的隐式规则

```makeifle
%.p :
```

存在是为了确保 Pascal 源文件（例如foo. p）匹配特定的目标模式，从而防止浪费时间寻找 *foo.p.o* 或 *foo.p.c*。

为列出为有效的每个后缀创建虚拟模式规则，例如 "`%.p`"的规则，以便在后缀规则中使用。（请参阅 [10.7 Old-Fashioned Suffix Rules](https://www.gnu.org/software/make/manual/make.html#Suffix-Rules)）。


### 10.5.6 取消隐式规则

您可以通过定义具有相同目标和先决条件但配方不同的新**模式规则**来覆盖内置隐式规则（或您自己定义的规则）。定义新规则时，将替换内置规则。新规则在隐式规则序列中的位置取决于您编写新规则的位置。

您可以通过定义具有相同目标和先决条件但没有配方的模式规则来取消内置隐式规则。例如，以下操作将取消运行汇编器的规则：

``` Makefile
%.o : %.s
```

## 10.6 定义 Last-Resort Default 规则

您可以通过编写没有先决条件的终端(terminal) *match-anything pattern rule* 来定义 *last-resort implicit rule*（请参阅 [10.5.5 Match-Anything Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Match_002dAnything-Rules)）。这就像任何其他**模式规则**一样；它唯一的特别之处在于它将匹配任何目标。因此，这样一个规则的配方用于所有没有自己的配方并且没有其他隐式规则适用的目标和先决条件。

例如，在测试 makefile 时，您可能不关心源文件是否包含真实数据，只关心它们是否存在。然后你可能会这样做：

```makefile
%::
    touch $@
```

以自动创建所需的所有源文件 (作为先决条件)。

或者，您可以定义一个配方，用于根本没有规则的目标，甚至没有指定配方的目标。您可以通过为目标 “`.DEFAULT`” 编写规则来做到这一点。这样的规则配方用于在任何显式规则中不作为目标出现的所有先决条件，并且没有隐式规则适用。自然地，除非您编写一个，否则没有 `.DEFAULT` 规则。

如果使用 “`.DEFAULT`” 而没有配方或先决条件:

```makefile
.DEFAULT:
```

之前为 `.DEFAULT` 存储的配方被清除。然后 *make* 像从未定义过 `.DEFAULT` 一样进行操作。

如果您不希望目标从 *match-anything pattern rule* 或 `.DEFAULT` 获取配方，但您也不希望为该目标运行任何配方，您可以给它一个空配方（请参阅 [5.9 Using Empty Recipes](https://www.gnu.org/software/make/manual/make.html#Empty-Recipes)）。

您可以使用 *last-resort* 规则覆盖另一个 makefile 的一部分。请参阅 [3.6 Overriding Part of Another Makefile](https://www.gnu.org/software/make/manual/make.html#Overriding-Makefiles)。

## 10.7 过时的后缀规则

**后缀规则**(Suffix rules)是 make 定义隐式规则的老式方法。后缀规则已经过时，因为**模式规则**更通用、更清晰。GNU make 支持它们，以便与旧 makefile 兼容。它们有两种：双后缀和单后缀。

双后缀规则由一对后缀定义：目标后缀和源后缀。它匹配名称以目标后缀结尾的任何文件。相应的隐式先决条件是通过将文件名中的目标后缀替换为源后缀来实现的。具有两个后缀的规则 "`.c.o`"（其目标和源后缀是 “.o” 和 “.c”）等价于模式规则 "`%.o : %.c`"。

单后缀规则由单后缀定义，即源后缀。它匹配任何文件名，通过附加源后缀来制作相应的隐式先决条件名称。源后缀为 '.c' 的单后缀规则等价于模式规则 "`% : %.c`"。

通过将每个规则的目标与已定义的已知后缀列表进行比较的方式，来识别后缀规则定义。当 make 看到规则的目标是已知后缀时，该规则被视为单后缀规则。当 make 看到规则的目标是两个已知后缀连接的规则时，该规则被视为双后缀规则。

例如，“.c” 和 “.o” 都在已知后缀的默认列表中。因此，如果您定义了一个目标为 “`.c.o`” 的规则，make 将其视为具有源后缀 “.c” 和目标后缀 “.o” 的双后缀规则。这是定义编译C源文件规则的过时的方法：

```makefile
.c.o:
    $(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<
```

后缀规则不能有任何自己的先决条件。如果有，它们被视为具有滑稽名称的普通文件，而不是后缀规则。因此，规则：

```makefile
.c.o: foo.h
    $(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<
```

告诉 make 如何从先决条件文件 *foo.h* 制作文件 *.c.o*，一点也不像模式规则：

```makefile
%.o: %.c foo.h
        $(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<
```

它告诉 make 如何从 “.c” 文件制作 “.o” 文件，并使用此模式规则制作所有 “.o” 文件也依赖于文件 *foo.h*。

没有配方的后缀规则也没有意义。它们不会像没有配方的模式规则那样删除以前的规则（请参阅 [10.5.6 Canceling Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Canceling-Rules )）。它们只是在数据库中输入作为目标连接的后缀或后缀对。

已知后缀只是特殊目标 `.SUFFIXES` 的先决条件的名称。您可以通过给 `.SUFFIXES` 编写规则，添加更多先决条件，来添加自己的后缀，如：

```makefile
.SUFFIXES: .hack .win
```

它将 “.hack” 和 “.win” 添加到后缀列表的末尾。

如果您希望消除默认已知后缀，而不仅仅是添加后缀，请为 `.SUFFIXES` 编写没有先决条件的规则。通过特殊豁免，这消除了 `.SUFFIXES` 的所有现有先决条件。然后，您可以编写另一个规则来添加所需的后缀。例如，

```makefile
.SUFFIXES:            # Delete the default suffixes
.SUFFIXES: .c .o .h   # Define our suffix list
```

'`-r`' 或 '`--no-builtin-rules`'标志会导致默认后缀列表为空。

在 make 读取任何 makefile 之前，变量 `SUFFIXES` 被定义到默认后缀列表中。您可以使用特殊目标 `.SUFFIXES` 的规则更改后缀列表，但这不会改变此变量。

## 10.8 隐式规则搜索算法

下面是 *make* 搜索目标 *t* 的隐式规则的过程。对于每个没有配方的双冒号规则、没有配方的普通规则的每个目标，以及不是任何规则的目标的每个先决条件，都遵循这个过程。在搜索规则链时，对于来自隐式规则的先决条件，它也递归地遵循。

此算法中没有提到后缀规则，因为一旦读取了 makefile，后缀规则就会转换为等效的模式规则。

对于格式为 "*archive(member)*" 的存档成员目标，以下算法会运行两次，首先使用整个目标名称 *t*，如果第一次运行未发现规则，则第二次使用 "*(member)*" 作为目标 *t*。

1. 将 *t* 拆分为一个目录部分，称为 *d*，其余部分称为 *n*。例如，如果 *t* 是 'src/foo.o'，则 *d* 是 'src/'，*n* 是 'foo.o'。

2. 列出其中一个目标与 *t* 或 *n* 匹配的所有模式规则。如果目标模式包含斜杠，则与 *t* 匹配；否则与 *n* 匹配。

3. 如果该列表中的任何规则都不是 *match-anything* 规则，或者如果 *t* 是隐式规则的先决条件，则从列表中删除所有 *non-terminal match-anything* 规则。

4. 从列表中删除所有没有配方的规则。

5. 对于列表中的每个模式规则：

    1. 找到词干 *s*，它是 *t* 或 *n* 与目标模式中的 `%` 匹配的非空部分，。
    2. 通过用 *s* 替换为 `%` 来计算先决条件名称；如果目标模式不包含斜杠，则将 *d* 附加到每个先决条件名称的前面。
    3. 测试所有先决条件是否存在或应该存在。（如果文件名在 makefile 中作为目标被提及，或作为目标 *T* 的显式先决条件提及，那么我们说它应该存在。）
    如果所有先决条件都存在或应该存在，或者没有先决条件，则适用此规则。 

6. 如果到目前为止还没有找到模式规则，请进一步寻找。对于列表中的每个模式规则：

    1. 如果规则是最终规则(terminal)，则忽略它并继续下一个规则。
    2. 像以前一样计算先决条件名称。
    3. 测试所有先决条件是否存在或应该存在。
    4. 对于每个不存在的先决条件，递归地遵循此算法，以查看是否可以通过隐式规则创建先决条件。
    5. 如果所有先决条件都存在、应该存在或可以由隐式规则制定，则此规则适用。 

7. 如果没有找到模式规则，则修改“应该存在”的定义：如果文件名作为目标被提及、或作为任何目标的显式先决条件被提及，那么它“应该存在”，并再次尝试步骤5和步骤6。此检查仅用于与旧版 GNU make 的向后兼容性: 我们不建议依赖它。

8. 如果没有隐式规则适用，则`.DEFAULT`的规则（如果有）适用。在这种情况下，向 *t* 给出与 `.DEFAULT` 相同的配方。否则，*t* 没有配方。

一旦找到适用的规则，对于该规则的每个目标模式(而不是匹配 *t* 或 *n* 的模式)，模式中的 "`%`" 被替换为 *s*，并存储结果文件名，直到重新制作目标文件 *t* 的配方被执行。配方执行后，这些存储的文件名中的每一个都被输入数据库，并标记为已更新并具有与文件 *t* 相同的更新状态。

当对 *t* 执行模式规则的配方时，自动变量被设置为对应于目标和先决条件。参阅 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)

# 11 Using `make` to Update Archive Files

存档文件(_Archive files_)是包含称为成员的命名子文件的文件；它们由程序 *ar* 维护，它们的主要用途是作为链接的子程序库。

## 11.1 Archive Members as Targets

存档文件的单个成员可以用作make中的目标或先决条件。您在存档文件*archinve*中指定名为*member*的成员，如下所示：

``` Makefile
archive(member)
```

此构造仅在目标和先决条件中可用，而在指令中不可用！只有 *ar* 和其他专门为档案操作而设计的程序才能这样做。因此，更新存档成员目标的有效指令必须使用 *ar* 。例如，此规则说通过复制文件 hack. o 在存档 foolib 中创建成员 hack.o：

``` Makefile
foolib(hack.o) : hack.o
	ar cr foolib hack.o
```

事实上，几乎所有存档成员目标都是以这种方式更新的，并且有一个隐式规则可以为您执行此操作。请注意：如果存档文件不存在，则需要 *ar* 的“c”标志。

要在同一个存档中指定多个成员，您可以将所有成员名称一起写在括号之间。例如：


``` Makefile
foolib(hack.o kludge.o)
```

等价于

``` Makefile
foolib(hack.o) foolib(kludge.o)
```

您还可以在存档成员引用中使用 shell 样式的通配符。请参阅 [4.4 Using Wildcard Characters in File Names](https://www.gnu.org/software/make/manual/make.html#Wildcards)。例如，`foolib(*.o)` 展开到名称以“.o”结尾的 foolib 存档的所有现有成员；也许是 `foolib(hack.o) foolib(kludge.o)` 。

# 16 Makefile公约

## 16.6 用户的标准目标

所有 GNU 程序在其 Makefile 中都应该有以下目标：

- ‘all’
- ‘install’
- ‘install-html’<br>‘install-dvi’<br>‘install-pdf’<br>‘install-ps’
- ‘uninstall’
- ‘install-strip’
- ‘clean’
- ‘distclean’
- ‘mostlyclean’
- ‘maintainer-clean’
- ‘TAGS’
- ‘info’
- ‘dvi’<br>‘html’<br>‘pdf’<br>‘ps’
- ‘dist’