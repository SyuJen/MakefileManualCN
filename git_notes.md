# GNU make 手册 个人中译版

Copyright (C)  2024  JohnSyu.
Permission is granted to copy, distribute and/or modify this document
under the terms of the GNU Free Documentation License, Version 1.3
or any later version published by the Free Software Foundation;
with no Invariant Sections, no Front-Cover Texts, and no Back-Cover
Texts.  A copy of the license is included in the section entitled ``GNU
Free Documentation License''.

创建一个名为 `makefile` 的文件，该文件描述程序中文件之间的关系，并提供更新每个文件的命令

# 2 An Introduction to Makefiles
## 2.1 What a Rule Looks like

```make
target ... : prerquisited ...
    recipe
    ...
    ...
```

`target`通常是由程序生成的文件的名称，或要执行的操作的名称，例如 'clean'。
`prerequisites`是用作创建目标的输入的文件。
`recipe`是使 `make` 执行的动作，（译者注：在本文中将 recipe 翻译为配方，之前想将其翻译为“指令”，但和 directive 的翻译会发生重合。由于文档内容较多，校对复杂，部分位置还是保留了将 recipe 翻译为 指令 的） 。注意：需要在每个`recipe`行的开头放置一个制表符！

示例：
```
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

在示例中，'clean'不是其它rule的prerequisite，所以 `make` 不会在未指定的情况下执行clean。这种不引用文件但只是动作的target称为虚假目标（_phony targets_）。
另外，它也不含prerequisites。

## 2.3 make执行Makefle的过程

默认情况下，make从第一个target开始（不是名称以“.”开头的target，除非它们还包含一个或多个 ‘/’)。这称为默认目标（_default goal_）。
在示例中，此rule用于重新链接 _edit_ ；但是在 make 可以完全处理此rule之前，它必须处理 _edit_ 所依赖的文件的 rule。

处理其他 rule 是因为它们的 target 为 goal 的先决条件。如果goal不依赖其他rule（或goal所依赖的任何东西等），则不处理该规则，除非您告诉make这样做。就像 make clean 一样。

和 edit 相关的文件有更新或修改后，执行make时会自动 relink

## 2.4 变量

[How to Use Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables)

示例：
```
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

没有必要详细说明编译单个C源文件的方法，因为 `make` 可以弄清楚它们：它有一个隐含的规则，可以使用 `cc-c` 命令从相应命名的 . c文件中更新 .o文件。例如，它将使用recipe `cc-c main.c-o main.o` 将main.c 编译为 main.o。

[Using Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Implicit-Rules)

当以这种方式自动使用“. c”文件时，此 . c文件也会自动添加到prerequisite列表中。因此，我们可以从prerequisite中省略 .c文件，前提是我们省略了recipe。

```
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

## 2.6 Makefile的另一种形式

当 makefile 的 object **仅由隐式规则创建时**，makefile的另一种风格是可能的。在这种makefile风格中，可以按prerequisites而不是target对条目进行分组。

```
objects = main.o kbd.o command.o display.o \
          insert.o search.o files.o utils.o

edit : $(objects)
        cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h
```

这种方式不一定更好。

## 2.7 清理目录的规则

编译程序 并不是编写 makefile 的唯一目标，如何删除所有目标文件和可执行文件，以便目录是“干净的”也是其中之一。

在实践中，我们可能希望以更复杂的方式编写规则来处理意料之外的情况。我们会使用 `.PHONY` ，这可以防止make被称为 _clean_ 的实际文件混淆，并导致它在rm出现错误的情况下继续运行。

```
.PHONY : clean
clean :
        -rm edit $(objects)
```

像这样的规则不应该放在makefile的开头，因为我们不希望它默认运行。

# 3 Writing Makefiles

来自数据库中的告诉 `make` 如何重新编译系统的信息被称为 makefile

## 3.1 What Makefiles Contain

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

Makefile使用“基于行”的语法。GNU make对语句行的长度没有限制。

使用反斜杠 `\` 字符转义内部换行符。

在需要区分的地方，我们将为以换行符结尾的单行（无论它是否被转义）称为“physical lines”，而“logical line”是一个完整的语句，包括所有转义的换行符，直到第一个非转义换行符。

处理反斜杠及换行符组合的方式取决于语句是recipe行还是非recipe行。处理recipe行中的反斜杠及换行符组合将在后面讨论，见[5.1.1 Splitting Recipe Lines](https://www.gnu.org/software/make/manual/make.html#Splitting-Recipe-Lines)

在recipe行之外，反斜杠及换行符组合将转换为单个空格字符。完成后，反斜杠及换行符组合**周围的所有空格都将压缩为单个空格**：这包括反斜杠之前的所有空格、反斜杠/换行符之后行首的所有空格以及任何连续的反斜杠/换行符组合。

#### Splitting Without Adding Whitespace

如果您需要拆分一行但不希望添加任何空格，您可以利用一个微妙的技巧：使用美元符号、反斜杠和换行符三个字符：

```
var := one$\
       word
```

make删除反斜杠及换行符并将以下行压缩为一行后，这相当于：

```
var := one$ word
```

然后make将执行变量扩展。变量引用'$'指的是一个不存在的单字符名称“”（空格）的变量，因此扩展为空字符串，给出一个最终赋值，相当于：

```
var := oneword
```

## 3.2 Makefile文件的名称

- GNU make
    不推荐。如果您有一个特定于GNU make的makefile，并且不会被其他版本的make理解，则应该使用此名称。
- makefile
- Makefile
    更推荐此名称

如果使用了非标准名称的makefile，则可以使用`-f name` 或 `--file=name` 告诉make。

## 3.3 Including Other Makefiles

## 3.4 变量 `MAKEFILES`

如果定义了环境变量`MAKEFILES`，make会将其值视为要在其他文件之前读取的附加makefile的名称列表（由空格分隔）。

## 3.5 How Makefiles Are Remade

## 3.6 Overriding Part of Another Makefile

## 3.7 How `make` Reads a Makefile

GNU make的工作分为两个不同的阶段。在第一阶段，它读取所有makefile，并内化**所有**变量及其值以及隐式和显式规则，并构建所有目标及其 prerequisites 的依赖关系图。在第二阶段，make使用这些内化数据来确定哪些 targets 需要更新，并运行更新它们所需的 recipe。

在第一阶段，变量及函数拓展是 immedia，相对的，也有 deferred 的拓展。理解这两个概念是重要的，详见 [3.7 How make Reads a Makefile](https://www.gnu.org/software/make/manual/make.html#Reading-Makefiles)

对于shell赋值操作符 `!=` , 右侧立即计算并交给shell。结果存储在左侧命名的变量中，该变量被视为递归扩展变量（因此将在每个引用上重新计算）。
## 3.8 How Makefiles Are Parsed

宏定义应定义在一物理行内，否则使用 $ 对变量进行展开时，不会重新拆分行。

## 3.9 Secondary Expansion

# 4 Writing Rules
规则(*rule*)会出现在makefile中，并说明何时以及如何重新创建某些文件（即该规则的目标(*target*)，通常每个规则只有一个目标）。它会列出作为目标的先决条件(*prerequisites*)的其他文件，以及用于创建或更新目标的指令(*recipe*)。

rule 的顺序的唯一作用，是指定默认目标（*default goal*，如果您没有另行指定目标，make要运行的目标）（译者注，这里的原文是"default goal: the target for make to consider, if you do not otherwise specify one"，是本人将 consider 翻译成 运行）。第一个 makefile 中第一个 rule 的第一个 target 是默认目标。有两个例外：以句点开头的目标不是默认目标，除非它还包含一个或多个斜杠, ‘/’; 定义模式规则的目标对默认目标没有影响。（请参阅 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/make.html#Pattern-Rules)。）

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

因为美元符号(`$`)用于开始创建变量引用，所以如果您真的想在 target 或 prerequisite 中使用美元符号，您必须编写两个, 即`$$` (见 [6 How to Use Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables)）。如果您启用了二次扩展（请参阅 [3.9 Secondary Expansion](https://www.gnu.org/software/make/manual/make.html#Secondary-Expansion)）并且您想在 prerequisite列表 中使用字面上的美元符号，您必须编写四个美元符号 (`$$$$`).

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

在确定 target 是否过期时，从不检查 order-only prerequisites；即使 order-only prerequisites 被标记为假(phony)（请参见 [4.6 phony Targets](https://www.gnu.org/software/make/manual/make.html#Phony-Targets)）也不会导致重建目标。

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

文件名开头的字符 `~` 也具有特殊意义。如果是单独的，或者后面跟一个斜线，它表示您的主目录。例如，`~/bin` 扩展为 `/home/you/bin`。如果 `~` 后面跟着一个单词，则该字符串表示用该单词命名的用户的主目录。例如 `~john/bin` 扩展为 `/home/john/bin` 。在没有每个用户主目录的系统（如MS-DOS或MS Windows）上，可以通过设置环境变量 HOME 来模拟此功能。

通配符扩展由 make 在目标和先决条件中自动执行。在 recipe 中，shell 负责通配符扩展。在其他情况下，只有当您使用 *wildcard* 函数明确请求通配符扩展时，才会进行通配符扩展。

通配符的特殊意义可以通过在其前面加一个反斜杠来关闭。因此，`foo\*bar`将引用一个特定的文件，该文件的名称由“foo”、星号和“bar”组成。

### 4.4.1 通配符示例

### 4.4.2 使用通配符的缺陷

### 4.4.3 `wildcard` 函数

通配符扩展在规则(rules)中自动发生，但在**设置变量时或在函数的参数中**通常不会发生。此时则需要使用 `wildcard` 函数。

```
$(wildcard pattern…)
```

此字符串在makefile中的任何位置使用，将替换为与给定 *pattern* 之一匹配的现有文件名的以空格分隔的列表。如果没有现有文件名与 *pattern* 匹配，则 `wildcard` 函数的输出中省略该 *pattern*。请注意，这与在规则中不匹配的通配符的行为不同，在规则中，它们被逐字使用而不是忽略（请参阅使用通配符的陷阱）。

与规则中的通配符扩展一样，`wildcard` 函数的结果是排序的。但是，每个单独的表达式都是单独排序的，因此 `$(wildcard *.c *.h)` 将先排序所有匹配 '.c' 的文件，然后是所有匹配 '.h' 的文件。

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

这是通过诸如“$^”之类的自动变量来完成的（请参见自动变量）。

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

`.LIBPATTERNS` 的默认值 是 `lib%.so lib%.a`，它提供了上述默认行为。通过将此变量设置为空值，可以完全关闭链接库扩展。

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

伪目标与 make 的递归调用结合使用也很有用（请参见 [5.7  Recursive Use of make](https://www.gnu.org/software/make/manual/make.html#Recursion)）。在这种情况下，makefile通常会包含一个变量，该变量列出了要构建的多个子目录。处理这一问题的一种简单方法是定义一个 recipe 在子目录上循环的规则，如下所示：

```makefile
SUBDIRS = foo bar baz

subdirs:
        for dir in $(SUBDIRS); do \
          $(MAKE) -C $$dir; \
        done
```

然而，这种方法也存在问题。首先，此规则会忽略在 sub-make 中检测到的任何错误，因此即使失败，它也会继续生成其余目录。这可以通过添加 shell 命令来记录错误并退出来的方式克服，但即使使用 -k 选项调用make，它也会这样做，这很不幸。第二点（但也许更重要的一点），您不能充分利用 make 的并行构建目标的能力（请参阅 [5.4 Parallel Execution](https://www.gnu.org/software/make/manual/make.html#Parallel)），因为只有一个规则。每个 makefile 的目标都将并行构建，但一次只能构建一个子目录。

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

您不应该将包含的 makefile 声明为伪。伪目标并不代表真实的文件，而且由于伪目标总是被认为是过时的，所以 make 总是会重新生成它，然后重新执行它自己（请参见 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/make.html#Remaking-Makefiles)）。为了避免这种情况，如果重新构建了标记为假的包含文件，make 将不会重新执行自己。

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

在这里，目标“FORCE”满足特殊条件，因此依赖它的目标 clean 被迫运行其配方。“FORCE”这个名字并没有什么特别之处，但这是一个常用的名字。

正如您所看到的，以这种方式使用“FORCE”与使用“,PHONY: clean”具有相同的结果。

正在使用“.PHONY”更明确、更高效。但是，make的其他版本不支持“.PHONY”；因此“FORCE”出现在许多makefile中。

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

为上述三个对象文件中的每一个提供了一个附加的先决条件。它相当于写：

```makefile
kbd.o: command.h
command.o: command.h
files.o: command.h
```

- 类似的配方适用于所有目标。自动变量 `$@` 可用于替换要重新映射到命令中的特定目标（请参见[10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html)）。例如：

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

在这里，我们假设程序 `program` 生成两种类型的输出，一种是给定 “-big” 的输出，另一种是假定给定 “-little” 的输出。有关子函数的解释，请参见 [8.2 Functions for String Substitution and Analysis](https://www.gnu.org/software/make/manual/html_node/Text-Functions.html)。

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

它可以插入或取出，而无需更改真正指定如何制作对象文件的规则，如果您希望间歇性地添加额外的先决条件，则可以方便地使用它。

另一个问题是，可以使用传递给 *make* 的命令行参数中设置的变量来指定附加的先决条件（请参见 [9.5 Overriding Variables](https://www.gnu.org/software/make/manual/html_node/Overriding.html)）。例如

```makefile
extradeps=
$(objects) : $(extradeps)
```

意味着命令 “make extradeps=foo.h” 将把 foo.h 视为每个对象文件的先决条件，而普通的 “make” 则不会。

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
当 `generate` 命令运行时，`$*` 将扩展到词干 ，要么是 'big'，要么是 'little'。

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

在程序的 makefile 中，需要编写的许多规则通常只表示某个对象文件依赖于某个头文件。例如，如果 main.c 通过 `#include` 使用 defs.h ，那么您将编写：

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

对于旧的 make 程序，传统做法是通过 `make-depend` 等命令使用此编译器功能根据需要生成先决条件。该命令将创建一个包含所有自动生成的先决条件的文件 *depend*，然后 makefile 可以使用 `include` 来读取它们（请参见 [3.3 Including Other Makefiles](https://www.gnu.org/software/make/manual/html_node/Include.html)）。

在GNU make中，重新制作(译者注，原文为remake，翻译成重新执行可能会更恰当？) makefile 的功能使这种做法过时了——你永远不需要明确地告诉 make 重新制作先决条件，因为它总是重新制作任何过期的 makefile。请参见如 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/html_node/Remaking-Makefiles.html)。

**我们建议自动生成先决条件的做法是，每个源文件对应一个 _makefile_**。对于每个源文件 name.c，都有一个 makefile name.d，其中列出了目标文件 name.o 所依赖的文件。这样，只需要重新扫描已更改的源文件即可生成新的先决条件。

以下是从名为 name.c 的C源文件生成名为 name.d b的先决条件文件（即makefile）的模式规则：

```makefile
%.d: %.c
    @set -e; rm -f $@; \
     $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
     sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' <$@.$$$$ > $@; \
     rm -f $@.$$$$
```

有关定义模式规则的信息，请参见 [10.5 Defining and Redefining Pattern Rules](https://www.gnu.org/software/make/manual/html_node/Pattern-Rules.html)。如果 `$(CC)` 命令（或任何其他命令）失败（以非零状态退出），shell的 `-e` 标志将使其立即退出。

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
（此示例使用替换变量引用将源文件 “foo.c bar.c” 的列表转换为先决条件生成文件 “foo.d bar.d” 的列表。有关替换引用的完整信息，请参阅 [6.3.1 Substitution References](https://www.gnu.org/software/make/manual/html_node/Substitution-Refs.html)。）由于 “.d” 文件与其他文件一样都是 makefile ，make 将根据需要重新制作它们，而无需您做进一步的工作。请参见 [3.5 How Makefiles Are Remade](https://www.gnu.org/software/make/manual/html_node/Remaking-Makefiles.html)。

请注意，“.d” 文件包含目标定义；您应该确保将 `include` 指令放在 makefile 中的第一个默认目标之后，或者冒着让随机对象文件成为默认目标的风险。请参见 [2.3 How make Processes a Makefile](https://www.gnu.org/software/make/manual/html_node/How-Make-Works.html)。

# 5 Writing Recipes in Rules
规则的配方由一个或多个shell命令行组成，这些命令行将按照出现的顺序一次执行一个。通常，执行这些命令的结果是使规则的目标成为最新的。

用户使用许多不同的shell程序，但除非 makefile 另有规定，否则 makefile 中的配方始终由 /bin/sh 执行。请参阅 [5.3 Recipe Execution](https://www.gnu.org/software/make/manual/html_node/Execution.html)。

## 5.1 配方的语法
Makefile有一个不同寻常的特性，即在一个文件中实际上有两种不同的语法。makefile 中大部分内容使用 make 语法（请参阅 [3 Writing Makefiles](https://www.gnu.org/software/make/manual/html_node/Makefiles.html)）。然而，配方是由 shell 解释的，因此它们是使用 shell 语法编写的。make 程序并不试图理解 shell 语法：在将配方交给 shell 之前，它只对配方的内容执行很少的特定翻译。

配方中的每一行都必须以一个制表符（或 `.RECIPEPREFIX` 变量值中的第一个字符；请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/html_node/Special-Variables.html)）开头，但第一个配方行可以用分号连接到目标-先决条件行。makefile 中以 tab 开头并出现在“规则上下文”中的任何一行（即，在一个规则启动之后，直到另一个规则或变量定义）都将被视为该规则的配方的一部分。在配方行中可能会出现空行和仅注释行，它们被忽略。

这些规则的一些后果包括：

- 以制表符开头的空行不是空白的：这是一个空配方（请参阅 [5.9 Using Empty Recipes](https://www.gnu.org/software/make/manual/make.html#Empty-Recipes)）。
- **配方中的注释不是注释；它将按原样传递给 shell**。shell 是否将其视为注释取决于您的 shell。
- “规则上下文”中(即以制表符作为行上第一个字符)的变量定义，将被视为配方的一部分，而不是 make 变量定义，并传递给 shell。
- “规则上下文”中(即以制表符作为行上第一个字符)的条件表达式（ifdef、ifeq等，请参阅 [7.2 Syntax of Conditionals](https://www.gnu.org/software/make/manual/make.html#Conditional-Syntax)）将被视为配方的一部分，并传递给 shell。

### 5.1.1 拆分配方行
*make* 解释配方的为数不多的方法之一是检查换行符前的反斜杠。与正常的 *makefile* 语法一样，通过在每个换行符之前放置一个反斜杠，可以将 makefile 中的单个逻辑配方行拆分为多个物理行。像这样的一系列行被视为单个配方行，将调用shell的一个实例来运行它。

但是，与 *makefile* 中其他地方的处理方式不同（请参见 [3.1.1 Splitting Long Lines](https://www.gnu.org/software/make/manual/make.html#Splitting-Lines)），“反斜杠/换行符对”不会从配方中删除。反斜杠和换行符都会被保留并传递给 shell。如何解释反斜杠/换行符取决于您的shell。如果反斜杠/换行后下一行的第一个字符是配方前缀字符（默认情况下为制表符；请参阅 [6.14 Other Special Variables](https://www.gnu.org/software/make/manual/make.html#Special-Variables)），则会删除该字符（并且仅删除该字符）。配方中从不添加空白。

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

请注意，反斜杠/换行符对是如何在用双引号（" "）引起来的字符串内删除的，而不是从用单引号（' '）引出来的字符串中删除的。这是默认 shell（/bin/sh）处理“反斜杠/换行符对”的方式。如果在 makefile 中指定了不同的 shell，则可能会对它们进行不同的处理。

有时，您希望在单引号内拆分一条长行，但不希望反斜杠/换行符出现在引用的内容中。当将脚本传递给 Perl 等语言时，通常会出现这种情况，脚本中多余的反斜杠可能会改变其含义，甚至是语法错误。一种简单的处理方法是将带引号的字符串，甚至整个命令放入 make 变量中，然后在配方中使用该变量。在这种情况下，将使用 makefile 的换行规则，并删除反斜杠/换行符。如果我们使用此方法重写上面的示例：

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
*make* 处理配方的另一种方法是扩展其中的任何变量引用（请参阅 [6.1 Basics of Variable References](https://www.gnu.org/software/make/manual/make.html#Reference)）。这发生在 *make* 完成所有 makefile 的读取并且目标被确定为过期之后；因此，不被重建的目标的配方永远不会扩展。

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

要制作的 `-s` 或 `--silent` 标志可以阻值所有的回显，就好像所有的配方都以 `@` 开头一样。在 makefile 中为特殊目标 “.SILENT” 设置的不带先决条件的规则具有相同的效果。（请参阅 [4.9 Special Built-in Target Names](https://www.gnu.org/software/make/manual/make.html#Special-Targets)）

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

与大多数变量不同，变量 `SHELL` 绝不从环境中设置。这是因为环境变量 `SHELL` 用于指定您个人选择的交互式 shell 程序。个人选择会影响 makefile 的功能将是非常糟糕的。请参见 [6.10 Variables from the Environment](https://www.gnu.org/software/make/manual/html_node/Environment.html)。

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

处理递归 *make* 调用会引发并行执行的问题。有关此方面的详细信息，请参见 [5.7.3 Communicating Options to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Options_002fRecursion.html)。

如果某个配方失败（被信号终止或以非零状态退出），并且该配方的错误未被忽略（请参阅 [5.5 Errors in Recipes](https://www.gnu.org/software/make/manual/html_node/Errors.html)），则用于重制相同目标的剩余配方行将不会运行。如果配方失败，并且没有给出“`-k`”或“`--keep-going`”选项（请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/html_node/Options-Summary.html)），则中止执行。如果 *make* 由于任何原因（包括信号）终止，但子进程正在运行，*make* 将等待它们完成，然后才真正退出。

当系统负载较重时，您可能希望运行的作业比负载较轻时更少。您可以使用“`-l`”选项告诉make根据平均负载限制同时运行的作业数。“`-l`”或“`--max load`”选项后面跟一个浮点数。例如：

```makefile
-l 2.5
```

如果平均负载高于2.5，则不会让make启动多个作业。不带后置数字的 “`-l`” 选项将删除负载限制（如果之前使用了“`-l`”选项）。

更准确地说，当make启动一个作业，并且它已经有至少一个作业在运行时，它会检查当前的平均负载；如果它不低于“`-l`”给定的限制，则进行等待，直到平均负载低于该限制，或者直到所有其他作业完成。

默认情况下，没有负载限制。

### 5.4.1 禁用并行执行

### 5.4.2 并行执行期间的输出

### 5.4.3 并行执行期间的输入

## 5.5 配方中的错误

## 5.6 中断或终止make

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

只需复制此示例，就可以编写递归 *make* 命令，但关于它们的工作方式和原因，以及 *sub-make* 与 顶层make 的关系，还有很多事情需要了解。您可能还发现将调用递归make命令的目标声明为 `.PHONY` 也很有用（有关何时有用的更多讨论，请参阅 [4.6 Phony Targets](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html)）。

为了方便起见，当GNU make启动时（在它处理了所有 `-c` 选项之后），它将变量 `CURDIR` 设置为当前工作目录的路径名。之后`make`再也不会触及此值：特别注意，如果您包含其他目录中的文件，则 `CURDIR` 的值不会更改。该值的优先级与在 *makefile* 中设置该值时的优先级相同（默认情况下，环境变量 `CURDIR` 不会覆盖该值）。请注意，设置此变量对 *make* 的操作没有影响（例如，它不会导致 *make* 更改其工作目录）。

### 5.7.1 变量`MAKE`的工作原理
递归 *make* 命令应始终使用变量 `MAKE`，而不是显式命令`make`，如下所示：

```makefile
subsystem:
    cd subdir && $(MAKE)
```

此变量的值是用于调用 *make* 的文件名。如果此文件名为 `/bin/make` ，则执行的配方为 `cd subdir&&/bin/make`。如果使用特殊版本的 *make* 来运行 顶层makefile，那么递归调用将使用相同的特殊版本来执行。

作为一项特殊功能，在规则的配方中使用变量 `MAKE` 会更改`-t`（`--touch`）、`-n`（`--just-print`）或`-q`（`-question`）选项的效果。使用 `MAKE` 变量的效果与在配方行开头使用 `+` 字符的效果相同。请参见 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/html_node/Instead-of-Execution.html)。只有当 `MAKE` 变量直接出现在配方中时，才会启用此特殊功能：如果通过扩展另一个变量引用 `MAKE` 变量，则此功能不适用。在后一种情况下，您必须使用 `+` 标记才能获得这些特殊效果。

请考虑上面示例中的命令“`make-t`”。（“`-t`”选项在不实际运行任何配方的情况下将目标标记为最新；请参阅 [9.3 Instead of Executing Recipes](https://www.gnu.org/software/make/manual/html_node/Instead-of-Execution.html)。）根据“`-t`”的常见定义，示例中的“`make -t`”命令将创建一个名为 *subsystem* 的文件，而不执行其他操作。你真正想让它做的是运行 `cd subdir && make -t`；但这需要执行配方，“`-t`”表示不执行配方。

这个特殊功能使它可以随心所欲：每当规则的配方行包含变量 `MAKE` 时，标志“`-t`”、“`-n`”和“`-q`”都不适用于该行。包含 `MAKE` 的配方行正常执行，尽管存在导致大多数配方不运行的标志。通常的 `MAKEFLAGS` 机制将标志传递给 *sub-make*（请参阅 [5.7.3 Communicating Options to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Options_002fRecursion.html)），因此您新建文件（译者注，原文中是 touch files， 但联想了一下 linux 中 `touch` 命令是用来新建也给文件的）或打印配方的请求会传播到子系统。

### 5.7.2 向 Sub-make 传递变量

顶层make 的变量值可以用显式请求通过环境传递给 sub-make。这些变量在 sub-make 中定义为默认值，但它们不会覆盖 sub-make 使用的 makefile 中定义的变量，除非使用 “`-e`” 开关（请参见 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/html_node/Options-Summary.html)）。

要向下传递或导出变量，*make* 将变量及其值添加到运行配方每一行的环境中。相应地，sub-make 使用环境来初始化其变量值表。请参见 [6.10 Variables from the Environment](https://www.gnu.org/software/make/manual/html_node/Environment.html)。

除非通过明确的请求，否则仅当变量最初在环境中定义，或者在命令行上设置并且其名称仅由字母、数字和下划线组成时，才使导出成为变量。

*make* 变量 `SHELL` 的值不会被导出。相反，调用环境中的 `SHELL` 变量的值会传递给 sub-make。您可以使用 `export` 指令强制 *make* 为 `SHELL` 导出其值，如下所述。请参见 [5.3.2 Choosing the Shell](https://www.gnu.org/software/make/manual/html_node/Choosing-the-Shell.html)。

特殊变量 `MAKEFLAGS` 始终被导出（除非您取消导出它）。将 `MAKEFILES` 设置为任意值即可导出它。

*make* 通过将命令行上定义的变量值放入 `MAKEFLAGS` 变量中，自动向下传递这些值。请参见 [5.7.3 Communicating Options to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Options_002fRecursion.html)。

如果变量是由 *make* 默认创建的，则通常不会向下传递（请参阅 [10.3 Variables Used by Implicit Rules](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html)）。sub-make将自己定义这些。

如果要将特定变量导出到 sub-make，请使用`export`指令，如下所示：

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

如果希望默认情况下导出所有变量，则可以单独使用导出：

```sh
export
```

这告诉 *make* 应该导出 `export` 和 `unexport` 指令中未明确提及的变量。`unexport` 指令中给定的任何变量仍然不会导出。

导出指令本身引发的行为是旧版本 GNU *make* 中的默认行为。如果您的 makefile 依赖于此行为，并且希望与旧版本的 make 兼容，则可以添加特殊目标 `.EXPORT_ALL_VARIABLES` 到 makefile，而不是使用 `export` 指令。这将被旧的 *make* 忽略，而`export` 指令将导致语法错误。


(部分内容尚未翻译)

### 5.7.3 将选项传达给Sub-make

### 5.7.4 "--print directory"选项

## 5.8 定义罐头配方

## 5.9 使用空配方

给出一个只包含空格的 recipe 即可（在10.1里说的是要写一个分号）。例如：

``` Makefile
target: ;
```

# 6 How to Use Variables

变量（在有些版本的make中也称作宏 macros）的值是文本字符串.
变量可以表示文件名列表、传递给编译器的选项、要运行的程序、要查找源文件的目录、要写入输出的目录或您可以想象的任何其他内容。
变量名称中不能包含 `:` `#` `=` ，虽然可以包含 字母、数字、下划线 以外的字符，但慎重。

变量名称是区分大小写的。

 [10.5.3 Automatic Variables](https://www.gnu.org/software/make/manual/make.html#Automatic-Variables)
## 6.1 变量引用基础

要替换变量的值，请在一个美元符号后紧跟括号或者大括号围起来的变量名称，例如`$(foo)` 或者 `${foo}`都是对变量 _foo_ 的有效引用。在文件名或者 recipe 中使用美元符号时，应使用 ‘`$$`’

## 6.2 变量的两种风格

### 6.2.1 递归扩展变量赋值

变量的第一种风格是递归扩展变量。此种风格的变量的定义使用 `=`（参阅 [6.5 Setting Variables](https://www.gnu.org/software/make/manual/make.html#Setting)) 或者 `define` 指令 (参阅 [6.8 Defining Multi-Line Variables](https://www.gnu.org/software/make/manual/make.html#Multi_002dLine))

```
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all:;echo $(foo)
```
结果会显示 ‘Huh?’，其过程是`$(foo)` 扩展成 `$(bar)` 又扩展成 `$(ugh)` 最后拓展成 `Huh?`

这种风格的缺点是如果对一个变量多次使用 `=` 进行赋值，make只会取其最后一次赋值；而如果想要在本身的后面追加内容，则会引起无限循环。

```make
CFLAGS = $(CFLAGS) -O # error, it will cause an infinite loop
```

### 6.2.2 简单扩展变量赋值

使用 `:=` 或 `::=` 定义的变量被称为 _Simply expanded variables_ 。
当定义变量时，简单扩展变量的值被扫描一次，扩展对其他变量和函数的任何引用。一旦扩展完成，变量的值就再也不会扩展了：当使用变量时，值被逐字复制为扩展。如果包含变量引用的值，扩展的结果将包含定义该变量时的值。

```
x := foo
y := $(x) bar
x := later
```

相当于 

```
y := foo bar # 因为在定义时就会被拓展
x := later
```

本人注：在再次使用 `:=` 或 `::=` 赋值时，会覆盖掉之前的定义。

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
结果是 `curname: zzk`，使用 `:=` 变量的真实值会被立即拓展，即使它所引用的变量的值变更也不会随之改变了
### 6.2.3 立即扩展变量赋值

使用 `:::=` 进行赋值。生成的变量是递归的：每次使用都会重新扩展。为了避免意外结果，值立即展开后会自动被转义：展开后值中 `$` 的所有实例都会转换为 `$$` 。示例：

```make
var = one$$two
OUT :::= $(var)
```
_OUT_ 的结果为 `one$$two` 。当变量被赋值时，该值被展开，因此结果是var的值的展开 `one$two` ；然后在赋值完成之前重新转义该值，得到 `one$$two` 的最终结果。

`:::=` 与 `::=` 类似，但区别在于：
1. 当使用 `+=` 向变量添加值时，`+=` 右侧的内容是 `::=` 赋值过的变量会当即拓展。
2. `:::=` 的变量的效率略低于简单扩展的变量，因为它们确实需要在使用时重新扩展，而不仅仅是复制。

```
var = one$$two
OUT :::= $(var)
OUT += $(var) # 当使用 :::= 时，+= 右侧的内容不会被当即拓展
var = three$$four
```

_OUT_ 的结果是 `one$$two $(var)` ，当 _OUT_ 被使用时，会被拓展成 `one$two three$four` 。

本人注：在再次使用 `:::=` 赋值时，会覆盖掉之前的定义。

### 6.2.4 条件变量赋值

`?=` ，只有在变量尚未定义时才有效。

```
FOO ?= bar
```
等价于
```
ifeq ($(origin FOO), undefined)
  FOO = bar
endif
```

注意，设置为空值的变量也算被定义了， `?=` 不会对该变量进行设置

## 6.3 引用变量的高级功能

### 6.3.1 替换引用

`$(var:a=b)` 或 `${var:a=b}` ，其作用是获取变量 _var_ 的值，将 _var_ 的值中的单词末尾的 _a_ 替换为 _b_，并替换最终的字符串。注意，“在单词末尾”指的是要么 _a_ 之后紧接着空格，要么出现在 _var_ 值的最后。

``` make
foo := a.o b.o l.a c.o
bar := $(foo:.o=.c)
```

最终结果是将 'bar' 设置为 ‘a.c b.c l.a c.c'

### 6.3.2 计算变量名称 

略

## 6.4 变量获取值的方式


## 6.6 将更多文本附加到变量

通常，向已定义的变量的值添加更多文本很有用。您可以使用包含如下 `+=` 的行来执行此操作：

```
objects += another.o
```

这将获取变量对象的值，并将文本 “another. o” 添加到其中（如果它已经有值，则前面有一个空格）。因此：

```
objects = main.o foo.o bar.o utils.o
objects += another.o
```

会将 'objets' 设置为 'main.o foo.o bar.o utils.o another.o'

## 6.7 `override` 指令

如果变量已使用命令参数设置（请参阅 [9.5 Overriding Variables](https://www.gnu.org/software/make/manual/make.html#Overriding)），则makefile中的普通赋值将被忽略。如果您想在makefile中设置变量，即使它是使用命令参数设置的，您可以使用覆盖指令

```
override variable = value
override variable := value
override variable += more text # To append more text to a variable defined on the command line
```

标有覆盖标志的变量赋值比除另一个覆盖外的所有其它赋值具有更高的优先级。未标记覆盖的后续赋值或附加到此变量将被忽略。

override 指令的发明是为了让您可以更改和添加用户使用命令参数指定的值。例如，假设您在运行C编译器时总是想要“-g”选项，但您希望像往常一样允许用户使用命令参数指定其他开关。则可以使用 override 指令：
```
override CFLAGS += -g
```

您还可以将 override 指令与 define 指令一起使用。

``` make
override define foo =
bar
endef
```

## 6.8 定义多行变量
略

## 6.9 取消变量定义

在 flavor 函数（[8.12 The `flavor` Function](https://www.gnu.org/software/make/manual/make.html#Flavor-Function)）和 origin 函数（[8.11 The `origin` Function](https://www.gnu.org/software/make/manual/make.html#Origin-Function)）中，一个值被设置为空或是没有被设置，是有区别的。使用 `undefine` 指令可以使变量没有被设置过。

``` make
undefine foo

# to undefine a command-line variable definition
override undefine CFLAGS
```

## 6.10 来自环境的变量
*make* 中的变量可以来自运行 *make* 的环境。*make* 启动时看到的每个环境变量都会转换为具有相同名称和值的 *make* 变量。但是，*makefile* 中的显式赋值或使用命令参数会覆盖环境。（如果指定了 “`-e`” 标志，则环境中的值将覆盖 *makefile* 中的分配。请参阅 [9.8 Summary of Options](https://www.gnu.org/software/make/manual/html_node/Options-Summary.html)。但实际中不建议这样做。）

因此，通过在环境中设置变量 `CFLAGS` ，可以使大多数 *makefile* 中的所有C编译使用您喜欢的编译器开关。这对于具有标准或传统含义的变量是安全的，因为您知道没有 *makefile* 会将它们用于其他事情。（请注意，这并不完全可靠；一些 *makefile* 显式设置 `CFLAGS`，因此不受环境中的值的影响。）

当 *make* 运行一个配方时，*makefile* 中定义的一些变量会被放入 *make* 调用的每个命令的环境中。默认情况下，只有来自 *make* 环境或在其命令行上设置的变量才会被放置到命令的环境中。您可以使用 `export` 指令来传递其他变量。有关完整的详细信息，请参阅 [5.7.2 Communicating Variables to a Sub-make](https://www.gnu.org/software/make/manual/html_node/Variables_002fRecursion.html)。

**不建议使用环境中的其他变量**。*makefile* 的功能依赖于在其控制范围之外设置的环境变量是不明智的，因为这会导致不同的用户从同一个 *makefile* 中获得不同的结果。这违背了大多数 *makefile* 的全部目的。

变量 `SHELL` 尤其可能出现这种问题，它通常存在于环境中，用于指定用户对交互式 *shell* 的选择。这种选择会影响 “*make*”，这是非常不可取的；因此，“*make*” 以一种特殊的方式处理 “`SHELL`” 环境变量；请参见 [5.3.2 Choosing the Shell ](https://www.gnu.org/software/make/manual/html_node/Choosing-the-Shell.html)。


## 6.11 特定于目标的变量值

```
target … : variable-assignment
```

可以使用`export`, `unexport`, `override`, 或 `private`。variable-assignment 可以是任何有效的赋值形式；递归 (‘=’), 简单 (‘:=’ 或 ‘::=’), 立即 (‘::=’), 附加 (‘+=’), 或条件 (‘? =’)。

特定于目标的变量与任何其他makefile变量具有相同的优先级。命令行上提供的变量（如果“-e”选项有效，则在环境中）将优先。

目标特定变量还有一个特殊特性：当您定义目标特定变量时，该变量值对该目标的所有先决条件、以及先决条件的所有先决条件也有效（除非这些先决条件用自己的目标特定变量值覆盖该变量）。

请注意，给定的先决条件最多只能在每次调用make时构建一次。如果同一个文件是多个目标的先决条件，并且每个目标对同一目标特定变量都有不同的值，那么要构建的第一个目标将导致该先决条件被构建，先决条件将从第一个目标继承目标特定值。它将忽略任何其他目标的目标特定值。
## 6.12 特定于模式的变量值

``` make
pattern … : variable-assignment
```

在其中，pattern 是 %-pattern。
除非指定了 override，否则任何命令行变量设置都将优先。

```
%.o : CFLAGS = -O
```
将会针对与模式 `%.o` 匹配的目标进行对`CFLAGS`赋值 `-O`

如果目标匹配多个模式，则首先解释具有较长词干的匹配模式特定变量。这会导致更具体的变量优先于更通用的变量。示例：
```
%.o: %.c
        $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@

lib/%.o: CFLAGS := -fPIC -g
%.o: CFLAGS := -g

all: foo.o lib/bar.o
```
在这个例子中，CFLAGS变量的第一个定义将用于更新lib/bar. o，即使第二个定义也适用于这个目标。导致相同词干长度的模式特定变量则按照它们在makefile中定义的顺序被考虑。

特定于模式的变量 会在 为该目标所显式定义的特定于目标的变量 之后被搜索，在 为父目标定义的特定于目标的变量 之前被搜索。
# 7 条件句语法

## 7.2 条件句语法

对条件进行判断有4种指令
1：
```
ifeq (arg1, arg2)
ifeq 'arg1' 'arg2'
ifeq "arg1" "arg2"
ifeq "arg1" 'arg2'
ifeq 'arg1' "arg2"
```
展开 `arg1` 和 `arg2` 中的所有变量引用并比较它们是否相同

```
ifeq ($(strip $(foo)),)
text-if-empty
endif
```

2：
```
ifneq (arg1, arg2)
ifneq 'arg1' 'arg2'
ifneq "arg1" "arg2"
ifneq "arg1" 'arg2'
ifneq 'arg1' "arg2"
```
展开 `arg1` 和 `arg2` 中的所有变量引用并比较它们是否不相同

3：
```
ifdef variable-name
```
判断 `variable-name` 是否被定义。注意：`variable-name` 仅会被展开一次，即使展开后是变量或是函数，也不会再被展开；并且如果展开的变量是空值，`ifdef` 语句也会被认为为`真`
示例：
```
bar = 
foo = $(bar)
ifdef foo # 会被判断为真

ifdef bar # 会被判断为假
```

4：
```
ifndef variable-name
```

上述4中判断指令被总结为 *conditional-directive*

完整的判断语句应该是：
*conditional-directive*
*text-if-true*
`else`
*text-if-false*
*endif*

条件指令行的开头允许并忽略额外的空格，但不允许使用制表符。如果该行以制表符开头，它将被视为规则指令(recipe for a rule)的一部分。
*make* 会在读取 Makefile 文件时评估条件语句，而自动变量是在 *reipce* 被执行后才被定义，所以条件语句中不允许使用自动变量
# 8 Functions for Transforming Text

函数的目的是在makefile中进行文本处理，以计算要操作的文件或要在 recipes 中使用的命令。

## 8.1 调用函数的语法

`$(function arguments)`
`${function arguments}`

除了调用 make 原本的函数，还可以通过内置函数 `call` 来创建自己的函数。
参数与函数名之间通过数量不限的空格或 TAB 分隔，多个参数之间通过逗号分隔。
除非下面另有说明，否则每个参数都会在调用函数之前展开。替换是按照参数出现的顺序完成的。

**特殊字符**
当使用特殊的字符作为函数参数时，您可能需要隐藏它们。GNU make不支持带有反斜杠或其他转义序列的转义字符（*疑问，不支持吗？估计指的是不支持使用反斜杠来转移这些特殊字符吧*）；但是，由于参数在展开之前会被拆分，因此您可以通过将它们放入变量来隐藏它们。
需要隐藏的特殊字符包含：
- 逗号
- 第一个参数中的初始空白
- 不匹配的小括号或大括号
- 如果您不想让它开始匹配的一对，则使用左括号或大括号

使用方法举例如下：
```
comma:= ,
# 注意 empty:=后没有空格
empty:=
space:= $(empty) $(empty)
foo:= a b c
bar:= $(subst $(space),$(comma),$(foo))
```
结果是 `bar` 变成了 `a,b,c`
## 8.2 Functions for String Substitution and Analysis

- `$(subst from,to,text)`
	将 `text` 中出现的每一个 `from` 替换成 `to`
	
- `$(patsubst pattern,replacement,text)`
    查找 *text* 中与 *pattern* 匹配的以空格分隔的单词，并用 *replacement* 替换它们。这里的 *pattern* 可能包含一个充当通配符的“%”，以匹配不限字符及数量的单个单词。如果 *replacement* 也包含一个“%”，则“%”将替换为与 *pattern* 中的“%”匹配的文本。与 *pattern* 不匹配的单词将在输出时保持不变。只有 *pattern* 和 *replacement* 中的第一个“%”被这样处理；任何后续的“%”都保持不变。

    *patsubst* 函数调用中的'%'字符可以用前置反斜杠(`\`)进行转义。而本身可能回引起对‘%’字符进行转义的反斜杠(`\`) 可以用更多的反斜杠进行转义。转义'%'字符或其他反斜杠的反斜杠在比较文件名或替换词干之前从 *pattern* 中删除。没有转义'%'字符的反斜杠不受干扰。例如，`the\%weird\\%pattern\\` 的结果是 `the%weird\`+任意单词+`pattern\\` ，最后两个反斜杠被保留的原因是它们不会影响"%"字符。
    
    单词之间的空格被折叠成单个空格字符；前导和尾随空格被丢弃。
    
    [6.3.1 Substitution References](https://www.gnu.org/software/make/manual/make.html#Substitution-Refs) 可以以更简单的方式实现 *patsubst* 函数的功能（*注，很有用*）
```
$(var:pattern=replacement)
# 等效于
$(patsubst pattern,replacement,$(var))

# suffix在这里特指后缀，一定要是后缀
$(var:suffix=replacement)
# 等效于
$(patsubst %suffix,%replacement,$(var))
```

例如
```
objects = foo.o bar.o baz.o
$(objects:.o=.c) # 等效于 $(patsubst %.o,%.c,$(objects))
```

- `$(strip string)`
	从*string*中删除前导和尾随空格，并将内部序列中的一个或多个空格字符替换为单个空格。`$(strip a b c )` 的结果是 `a b c`
	当使用`ifeq`或`ifneq`将某物与空字符''进行比较时，您通常需要一个只有空格的字符串来匹配空字符串(详见 [7 Conditional Parts of Makefiles](https://www.gnu.org/software/make/manual/make.html#Conditionals))(建议看看 bootloader 和 linux 的 makefile 写法)。例如：
```
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
```
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
```
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
```
$(subst :, ,$(VPATH))
```
结果是 `src ../headers` 。然后使用 `patsubst` 将每个目录名转换为“-I”标志。这些可以添加到变量CFLAGS的值中，该值会自动传递给C编译器，如下所示：
```
override CFLAGS += $(patsubst %,-I%,$(subst :, ,$(VPATH)))
```
效果是将文本 `-Isrc -I../headers` 附加到先前给定的CFLAGS值。使用覆盖指令，即使使用命令参数指定了CFLAGS的先前值，也会分配新值。
## 8.3 Functions for File Names

一些内置的扩展功能专门涉及拆分文件名或文件名列表。以下每个函数都对文件名执行特定的转换。函数的参数被视为一系列文件名，由空格分隔。（前导和尾随空格被忽略。）

- $(dir names...)
- $(notdir names...)
    提取 _names_ 中每个文件名的目录部分以外的所有内容。如果文件名不包含斜杠，则保持不变。否则，最后一个斜杠前的所有内容都将从中删除。
    ```
    $(notdir src/foo.c hacks)
	```
	结果是 'foo.c hacks'
- $(suffix names...)
- $(basename names...)
- $(addsuffix suffix,names...)
- $(addprefix prefix,names...)
    _prefix_ 的值被添加到每个单独名称的前面，生成的较大名称之间用单个空格连接。
    ```
    $(addprefix src/,foo bar)
	```
	结果是 'src/foo src/bar'
- $(join list1,list2)
- $(wildcard pattern)
    参数 *pattern* 是文件名模式，通常包含通配符。`wildcard` 函数的结果是一个由空格分隔的、匹配该 *pattern* 的、现有文件的名称列表。请参阅 [4.4 Using Wildcard Characters in File Names](https://www.gnu.org/software/make/manual/make.html#Wildcards)。
- $(realpath names...)
- $(abspath names...)

## 8.6 `foreach` 函数

语法
```
$(foreach var,list,text)
```

*var* 是一个局部的临时变量，只在 `foreach` 函数中有效。

首先将前两个参数 *var* 和 *list* 在做其它任何事情之前展开；然后将 *list* 中以空格分隔的字符串依次赋值给 *var* 临时变量，再执行 *text* 表达式；重复直到 *list* 的最后一个字符串。

根据上述描述可推测 *text* 需包含 *var* 变量的引用。结果是 *text* 被扩展的次数等于 *list* 中以空格分隔的单词的个数。*text* 的多个扩展被连接起来，它们之间有空格，作为 `foreach` 的结果。

```
dirs := a b c d
files := $(foreach dir,$(dirs),$(wildcard $(dir)/*))
```

的结果是（`wildcard`函数执行的结果见4.3）

```
files := $(wildcard a/* b/* c/* d/*)
```

## 8.14 `shell` 函数

此函数将shell命令作为参数并扩展到命令的输出。
# 10 Using Implicit Rules

_隐式规则_ 告诉 _make_ 如何使用惯用的方法，以便您在想要使用它们时不必详细指定它们。
一系列隐式规则可以按顺序应用。
内置的隐式规则在其 recipes 中使用了几个变量，因此，通过更改变量的值，可以更改隐式规则的工作方式。例如，变量 `CFLAGS` 通过C编译的隐式规则控制给C编译器的 flags 。
您可以通过编写 _pattern rules_ 来定义自己的隐式规则。
后缀规则( _suffix fules_ )是一种定义隐式规则的更有限的方法。模式规则( _pattern rules_ )更通用、更清晰，但为了兼容性保留了后缀规则。
## 10.1 使用隐式规则

为了让 make 找到更新目标文件的惯用的方法，你所要做的就是不要自己指定 recipes。要么写一个没有 recipes 的规则，要么根本不写规则。

如果您需要指定隐式规则无法提供的附加先决条件（例如头文件），您可能希望为foo. o编写一个没有 recipes 的规则。

每个隐式规则都有一个 _目标模式_ 和 _先决条件模式_ 。可能有许多具有 _相同目标模式_ 的隐式规则。例如，许多规则生成“. o”文件：一个来自带有C编译器的“.c”文件；另一个来自带有Pascal编译器的“.p”文件；等等。实际适用的规则是其先决条件存在或可以创建的规则。因此，如果您有一个文件foo.c，make将运行C编译器；否则，如果您有一个文件foo.p，make将运行Pascal编译器；等等。

## 10.2 内置规则目录

这是一个预定义的隐式规则目录，除非makefile明确覆盖或取消它们，否则这些规则始终可用。[10.5.6 Canceling Implicit Rules](https://www.gnu.org/software/make/manual/make.html#Canceling-Rules) . `-r` 或 `--no-builtin-rules` 选项取消所有预设规则。

不同操作系统，如POSIX-based, VMS, Windows, OS/2等有不同的默认规则，在终端中使用 `make -p` 查看完整列表。本文档仅记录了部分基于POSIX的操作系统上可用的默认规则。

**Compiling C programs**
_n.o_ 是使用如下形式的 recipe 从 _n.c_ 自动制作的
`$(CC) $(CPPFLAGS) $(CFLAGS) -c`

**Linking a single object file**
_n_ 是通过运行C编译器链接程序从 *n. o* 自动生成的。使用的精确 recipe 是 `$(CC) $(LDFLAGS) n.o $(LOADLIBES) $(LDLIBS)` 。

## 10.3 隐式规则所使用的变量

可以更改这些变量的值。使用 `-R` 或 `--no-builtin-variables` 选项可取消隐式规则使用的所有变量。

部分如下，在终端中使用 `make -p` 查看完整列表：
**AS**
编译程序集文件(assembly files)的程序；默认为"as"。
**CC**
编译C代码的程序；默认为“cc”。
**CFLAGS**
给C编译器的额外标志。
**LDFLAGS**
当编译器应该调用链接器“ld”时，要给编译器的额外标志，例如`-L`。库（-lfoo）应该添加到 _LDLIBS_ 变量中。
**LDLIBS**
当编译器应该调用链接器“ld”时，给编译器的库标志或名称。LOADLIBES是LDLIBS的弃用（但仍受支持）替代方案。非库链接器标志，例如-L，应该放在LDFLAGS变量中。

## 10.4 隐式规则链

有时可以通过一系列隐式规则制作文件。例如，可以通过首先运行 Yacc 然后运行 cc 从 *n.y* 制作文件 *n.o* 。这样的序列称为链（*chain*）。

## 10.5 定义和重定义模式规则（pattern rules）

模式规则( *pattern rule* )与普通规则不同之处在于牵扯包含字符 `%` 。target 被认为是匹配文件名的模式；`%` 可以匹配任何非空子字符串。先决条件同样使用 `%` 来显示它们的名称与目标名称的关系。
`%.o : %.c` 说明如何从另一个文件 *stem.c* 制作任何文件 *stem.o* （stem 词干）

请注意，在模式规则中使用“%”的扩展发生在任何变量或函数扩展之后，这些扩展发生在读取makefile时。详细见 [6 How to Use Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables) 和 [8 Functions for Transforming Text](https://www.gnu.org/software/make/manual/make.html#Functions) 。

### 10.5.1 模式规则介绍

例如，‘*%.c*’ 作为模式匹配任何以'*.c*'结尾的文件名。'*s.%.c*'作为模式，匹配任何以'*s.*'开头、以'*.c*'结尾且长度至少为五个字符的文件名。（必须至少有一个字符匹配 ‘%’.) '*%*'匹配的子字符串称为词干（stem）。

pattern rule 的 prerequisite 中的'*%*'代表与 target 中的'*%*'匹配的相同词干。为了应用 pattern rule，其 target pattern 必须与考虑的文件名匹配，并且其所有 prerequisites（模式替换后）必须命名存在或可以创建的文件。这些文件成为 target 的 prerequisites。

``` Makefile
%.o : %.c ; recipe…
```

也可能有不使用“*%*”的先决条件；这样的先决条件附加到此模式规则创建的每个文件。这些不变的先决条件偶尔很有用。

有一个例外：如果一个 pattern target 过期或不存在并且makefile不需要构建它，那么它不会导致其他目标被认为过期（这个历史异常将在GNU make的未来版本中被删除）。如果检测到这种情况make将生成一个警告模式配方没有更新对等目标；但是make无法检测到所有这样的情况。
### 10.5.2 模式规则举例
### 10.5.3 自动变量

假设您正在编写一个模式规则，将一个“*.c*”文件编译成一个“*.o*”文件：您如何编写“*cc*”命令，以便它在正确的源文件名上运行？您不能在 recipe 中写入名称，因为每次应用隐式规则时名称都不同。您所做的是使用 make 的一个特殊功能，即自动变量(Automatic Variables)。

自动变量仅在 recipe 中有效。

- **$@**
	规则(rule)的目标(target)的文件名。
	如果目标是存档成员，则`$@`是存档文件的名称。在具有多个目标的模式规则中， `$@` 是导致规则的指令(recipe)运行的目标名称。
- **$%**
	当目标是存档成员时，该符号表示目标成员的名称。见 [11 Using make to Update Archive Files](https://www.gnu.org/software/make/manual/make.html#Archives)
	例如，目标是 *foo.a(bar.o)* ，则 `$%` 代表 bar.o， `$@` 是 foo.a 。
	当目标不是存档成员时， `$%` 是空。
- **$<**
	第一个先决条件的名称。
	如果目标通过隐式规则获取指令，则`%<`是通过隐式规则添加的第一个先决条件
- **$?**
	新于目标的所有先决条件的名称，它们之间有空格。
	如果目标不存在，所有先决条件都将包括在内。对于作为存档成员的先决条件，仅使用命名成员（见 [11 Using make to Update Archive Files](https://www.gnu.org/software/make/manual/make.html#Archives)）。
	`$?`即使在仅对已更改的先决条件进行操作的显式规则中也很有用。例如，假设一个名为 *lib* 的存档应该包含多个目标文件的副本。此规则仅将更改的目标文件复制到存档中：
``` Makefile
lib: foo.o bar.o lose.o win.o
        ar r lib $?
```
- `$^`
	所有先决条件的名称，它们之间有空格。
	对于作为存档成员的先决条件，只使用命名成员（见 [11 Using make to Update Archive Files](https://www.gnu.org/software/make/manual/make.html#Archives)）。无论每个文件作为先决条件列出多少次，一个目标在它所依赖的每个文件上只有一个先决条件。因此，如果您多次列出目标的先决条件，`$^`的值只包含名称的一个副本。此列表不包含任何仅顺序的先决条件；对于这些，请参阅下面的`$|`变量。
- `$+`
	这就像`$^`, 但是列出多次的先决条件会按照它们在makefile中列出的顺序重复。这主要用于链接命令，在这些命令中以特定顺序重复库文件名是有意义的。
- `$|`
	所有仅顺序的先决条件的名称，它们之间带有空格。
- `$*`
	隐式规则匹配的词干（请参阅 [10.5.4 How Patterns Match](https://www.gnu.org/software/make/manual/make.html#Pattern-Match)）。
	在显式规则中，没有词干；因此不能以这种方式确定`$*`。相反，如果目标名称以可识别的后缀结尾（参见 [10.7 Old-Fashioned Suffix Rules](https://www.gnu.org/software/make/manual/make.html#Suffix-Rules)), 则 `$*` 被设置为目标名称减去后缀。例如，如果目标名称是“foo. c”，则将`$*`设置为“foo”，因为“.c”是后缀。GNU make做这个奇怪的事情只是为了与make的其他实现兼容。
	除了在隐式规则或静态模式规则中，您通常应该避免使用`$*`。
	如果显式规则中的目标名称不以可识别的后缀结尾, `$*` 则设置为该规则的空字符串。

在上面列出的变量中，四个具有单个文件名的值，三个具有文件名列表的值。这七个变量的变体仅获取文件的目录名或目录中的文件名。变体变量的名称分别由附加“D”或“F”组成。函数dir和notdir可用于获得类似的效果（参见 [8.3 Functions for File Names](https://www.gnu.org/software/make/manual/make.html#File-Name-Functions)）。但是请注意，“D”变体都省略了总是出现在dir函数输出中的尾随斜杠。
- `$(@D)`
	目标文件名的目录部分，删除了尾随斜杠。如果`$@`的值是 dir/foo. o，则`$(@D)`是 dir。这个值是 `.` 如果`$@`不包含斜杠。
- `$(@F)`
	目标文件名的“目录内文件“部分。如果`$@`的值是 dir/foo. o，则 `$(@F)` 是 foo.o。`$(@F)` 等价于`$(notdir $@)` 。
- `$(*D)` <br> `$(*F)`
	词干的目录部分和“目录内文件“部分；在本例中分别对应 dir 和 foo。
- `$(%D)`  <br> `$(%F)`
	目标存档成员名称的目录部分和“目录内文件“部分。这仅对存档（成员）形式的存档成员目标有意义，并且仅在成员可能包含目录名时有用。（请参阅 [11.1 Archive Members as Targets](https://www.gnu.org/software/make/manual/make.html#Archive-Members)）
- `$(<D)` <br> `$(<F)`
	第一个先决条件的目录部分和“目录内文件“部分。
- `$(^D)` <br> `$(^F)`
	所有先决条件的目录部分和“目录内文件“部分的列表。
- `$(+D)` <br> `$(+F)`
	所有先决条件的目录部分和“目录内文件“部分的列表，包括重复先决条件的多个实例。
- `$(?D)` <br> `$(?F)`
	新于目标的所有先决条件的目录部分和“目录内文件“部分的列表。

`$<` 指的是名为`<`的变量，就像`$(CFLAGS)`”指的是名为`CFLAGS`的变量一样。也可以用 `$(<)` 代替`$<`。

### 10.5.4 模式(pattern)匹配的方式
### 10.5.6 取消隐式规则

您可以通过定义具有相同 _target_ 和 _prerequisites_ 但不同 _recipe_ 的新 _pattern rule_ 来覆盖内置隐式规则（或您自己定义的规则）。定义新规则时，将替换内置规则。

例如，以下将取消运行汇编器的规则：
``` Makefile
%.o : %.s
```

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

您还可以在存档成员引用中使用shell样式的通配符。请参阅 [4.4 Using Wildcard Characters in File Names](https://www.gnu.org/software/make/manual/make.html#Wildcards)。例如，`foolib(*.o)` 扩展到名称以“.o”结尾的 foolib 存档的所有现有成员；也许是 `foolib(hack.o) foolib(kludge.o)` 。
## 11.2 Implicit Rule for Archive Member Targets


## 11.3 Dangers When Using Archives

## 11.4 Suffix Rules for Archive Files

# 12 Extending GNU `make`