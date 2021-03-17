---
title: 使用Python求Select集
date: 2020-07-16 17:53:22
updated: 2020-07-16 17:53:22
tags: ["编译原理","Python","学习日记"]
comments: true
mathjax: true
---

刚刚整完编译原理的课设报告，突然想起来求Select的时候遇到的各种坑，想到当时网上各种资料的错误和不全面，然后又想到万年没更新过的博客，特地来更新一下。

<!-- more -->

## 准备工作

> 既然要用`Python`求Select集，那至少得知道手动怎么求吧（笑

众所周知，求Select集首先需要求得First集和Follow集，先来看一看定义：

### First集合

顾名思义，First集就是一个文法符号串，所可能推导出的符号串中，第一个终结符构成的集合。

如果还是不理解的话，举个例子：First(X) 就是求X所有能推导出内容中第一个终结符的集合。

求First集分为：

1. 求终结符的First集：终结符的First集是他本身。

2. 求非终结符的First集：

   + 若A->a，则根据定义，a $\in$ First(A)。

   + 若A->B，同样根据定义，First(B) $\subseteq$ First(A)。

3. 求符号串的First集（例如ABC）：

   1. 第一步：看串首是否为终结符或不能推导出ε（空）的非终结符。
      + 若是终结符，则First(ABC) = { 该终结符 }；
      + 若是不能推导出ε的非终结符，则First(ABC) =  First(A)；
      + 若非上述情况，则First(A) - ε $\subseteq$ First(ABC)，并进行下一步。
   2. 第二步：若串首非终结符能推出ε，当其推出空时，符号串将变为BC。
      + 重复第一步的操作，即查看随后的符号是否为终结符或或不能推导出ε的非终结符，并按第一步操作。
      + 若不能推出ε则结束循环。

注意事项：First集合中的符号一定是终结符，终结符也包括空串ε。

### Follow集合

同样，顾名思义，Follow集就是非终结符后面可能跟随的终结符的集合（不包括ε）。

求Follow集分为：

1. `A->...Bc...`型：根据定义，c $\in$ Follow(B)。

2. `A->...BC...`型：根据定义，First(C) $\subseteq$ Follow(B)。

3. `A->...C`型：文法左部的Follow集包含于串尾非终结符的Follow集。本例中Follow(A) $\subseteq$ Follow(C)。

4. `A->...BC`这一类较为特殊，处理方法如下：

   + 该类型是`A->...BC...`型和`A->...C`型的特例情况，除了要按`A->...BC...`型的情况处理，将First(C) $\subseteq$ Follow(B)，还要进行以下操作：

     1. 第一步：看串尾C能否推导出ε。

        + 若不能推导出空，则同`A->...C`型操作，将Follow(A) $\subseteq$ Follow(C)。

        + 若能推导出空，则Follow(A)也属于Follow(B)，即Follow(A) $\subseteq$ Follow(B)，并进行下一步。

     2. 第二步：若串尾非终结符能推导出ε，当其推空时，符号串将变为`A->...B`。

        + 重复第一步操作，即判断当前串尾符号B能否推导出ε，并按第一步操作。
        + 若不能推出ε则结束循环。

注意事项：Follow集合中的符号一定是终结符，并且不包括空串ε，而且定义**开始符号** 的Follow集合包含 `#` 。

### Select集合

Select集合则是产生式左部的可能的推导结果的起始符号。

举个例子：Select(A->B)就是求这个文法产生式中A可能推导出起始符号集合。

求Select集分为：

1. `A–>X（X为任意串且X不能推导出空）`型：根据定义，Select(A–>X) = First(X)。
2. `A–>X（X为任意串且X能推导出空）`型：
   + 当X不推导出空时，First(X) $\subseteq$ Select(A–>X)
   + 当X推导出空时，A推导出的符号串就会是A后面的符号的推导结果，即Follow(A)，故Follow(A) $\subseteq$ Select(A–>X)
   + 所以 Select(A–>X) = First(X) $\cup$ Follow(A) 

## 利用Python自动求Select集

> 有了上面这些前置知识，就可以着手用程序求这些集合了！

程序求法与人工求法有些许不同，具体表现在程序是顺序执行的，没有人工求那么灵活，所以可能需要运行多次才能得出正确结果~

### 大致思路

1. 求First集：

   + 先求First集第一个部分：针对->直接推出第一个字符为终结符部分，直接将其加入非终结符的First集中。

   + 再循环求出First集第二部分：针对A -> B型，反复把B的first集加到A的first集合中并去重，直到集合不再扩大为止。

2. 求Follow集：

   + 将非终结符后的终结符加入非终结符的Follow集
   + 将非终结符后跟非终结符的First集加入前者的Follow集
   + 当非终结符在末尾时，将生成式左侧的Follow集加入末尾非终结符的Follow集
   + 若句末非终结符的First集包含空，则也将生成式左侧的Follow集加入倒数第二个位置上的非终结符的Follow集中，重复向前直到遇到终结符或First集中不含空的非终结符。

   循环上述过程并去重，直到集合不再扩大为止。

3. 求Select集：

   针对A->X，分为两种情况讨论：

   + 若X不能推导出空，则根据定义，A的Select集应当为First（X）

   + 若X能推导出空，则根据定义， First（X）属于A的Select集，此外，Follow（A）也属于A的Select集。

### 代码实现

我个人实现的代码算是覆盖了大多数情况，暂时没有发现什么Bug。

若要使用大概需要注意：

+ 文法中各符号用空格分开
+ 非终结符用`< >`包裹(也可以通过修改`判断非终结符的函数`修改非终结符样式)
+ 结果保存在 `self.FIRST`、  `self.FOLLOW` 、 `self.SELECT`三个字典中，也可以通过去除`run_cal`函数中`self.debug_out()`前的注释符号 `#` 实现在控制台输出上述三个集合。

[代码下载](https://www.qiyuex.top/download/BlogDownload/cal_select.py)

```python
# 判断非终结符的函数
def isnonterminal(symbol):
    if symbol[0] == '<' and symbol[-1] == '>':
        return True
    return False


class CalSelect(object):
    def __init__(self):
        # 初始化集合字典
        self.FIRST = {}
        self.FOLLOW = {}
        self.SELECT = {}
        # 初始化文法
        self.grammar = [
			# 文法填在此处
			"<E>-><T> <E'>",
			"<E'>->+ <T> <E'>",
			"<E'>->ε",
			"<T>-><F> <T'>",
			"<T'>->* <F> <T'>",
			"<T'>->ε",
			"<F>->( <E> )",
			"<F>->i"
        ]
        # 初始化终结符
        for i in range(0, len(self.grammar)):
            self.grammar[i] = self.grammar[i].replace('函数名', 'FT')
            self.grammar[i] = self.grammar[i].replace('变量名', 'iT')
            self.grammar[i] = self.grammar[i].replace('常数', 'CT')
            self.grammar[i] = self.grammar[i].replace('字符串', 'sT')
            self.grammar[i] = self.grammar[i].replace('字符', 'cT')
        # 初始化first集、follow集和select集字典的键值对中的值为空
        for line in self.grammar:
            part_begin = line.split("->")[0]
            part_end_temp = line.split("->")[1]
            part_end = part_end_temp.split(" ")
            self.FIRST[part_begin] = []
            self.FOLLOW[part_begin] = []
            self.SELECT[line] = []
        self.FOLLOW[self.grammar[0].split("->")[0]].append('#')

    # 求first集中第一部分：针对->直接推出第一个字符为终结符部分
    def getFirst(self):
        for line in self.grammar:
            part_begin = line.split("->")[0]
            part_end_temp = line.split("->")[1]
            part_end = part_end_temp.split(" ")
            if not isnonterminal(part_end[0]):
                self.FIRST[part_begin].append(part_end[0])

    # 求first第二部分：针对A -> B型，把B的first集加到A的first集合中
    def getFirst_2(self):
        for line in self.grammar:
            part_begin = line.split("->")[0]
            part_end_temp = line.split("->")[1]
            part_end = part_end_temp.split(" ")
            # 如果型如A -> B：则把B的first集加到A的first集中去
            if isnonterminal(part_end[0]):
                for i in range(0, len(part_end)):
                    if not isnonterminal(part_end[i]):
                        self.FIRST[part_begin].append(part_end[i])
                        break
                    list_remove = self.FIRST.get(part_end[i]).copy()
                    if 'ε' in list_remove and i is not len(part_end) - 1:
                        list_remove.remove('ε')
                    self.FIRST[part_begin].extend(list_remove)
                    if 'ε' not in self.FIRST[part_end[i]]:
                        break

    def getFirst_3(self):
        while 1:
            test = self.FIRST
            self.getFirst_2()
            # 去除重复项
            for i, j in self.FIRST.items():
                temp = []
                for word in list(set(j)):
                    temp.append(word)
                self.FIRST[i] = temp
            if test == self.FIRST:
                break

    def getFOLLOW_3(self):
        while 1:
            test = self.FOLLOW
            self.getFollow()
            # 去除重复项
            for i, j in self.FOLLOW.items():
                temp = []
                for word in list(set(j)):
                    temp.append(word)
                self.FOLLOW[i] = temp
            if test == self.FOLLOW:
                break

    # 计算follow集的第一部分，先计算 S -> A b 类型的
    def getFollow(self):
        for line in self.grammar:
            part_begin = line.split("->")[0]
            part_end_temp = line.split("->")[1]
            part_end = part_end_temp.split(" ")
            if part_begin == "<G>":
                pass
            # 如果是 S->a 直接推出终结符 则 continue
            if len(part_end) == 1 and not isnonterminal(part_end[0]):
                continue
            # 否则执行下面的操作
            else:
                # 将->后面的倒序
                part_end.reverse()
                # 最后一个为非终结符
                if isnonterminal(part_end[0]):

                    for i in range(0, len(part_end)):
                        if not isnonterminal(part_end[i]):
                            break
                        self.FOLLOW[part_end[i]].extend(self.FOLLOW.get(part_begin))
                        if 'ε' not in self.FIRST[part_end[i]]:
                            break

                    terminal_temp = part_end[0]
                    for item in part_end[1:]:
                        if not isnonterminal(item):
                            terminal_temp = item
                        else:
                            if isnonterminal(terminal_temp):
                                list_remove = self.FIRST.get(terminal_temp).copy()
                                if "ε" in list_remove:
                                    list_remove.remove("ε")
                                self.FOLLOW[item].extend(list_remove)
                            elif terminal_temp != 'ε':
                                self.FOLLOW[item].append(terminal_temp)
                            terminal_temp = item
                # 如果终结符在句型的末端
                else:
                    terminal_temp = part_end[0]
                    for item in part_end[1:]:
                        if not isnonterminal(item):
                            terminal_temp = item
                        else:
                            if isnonterminal(terminal_temp):
                                list_remove = self.FIRST.get(terminal_temp).copy()
                                if "ε" in list_remove:
                                    list_remove.remove("ε")
                                self.FOLLOW[item].extend(list_remove)
                            elif terminal_temp != 'ε':
                                self.FOLLOW[item].append(terminal_temp)
                            terminal_temp = item

    def getSelect(self):
        for line in self.grammar:
            part_begin = line.split("->")[0]
            part_end_temp = line.split("->")[1]
            part_end = part_end_temp.split(" ")
            line_first = []
            for item in part_end:
                if not isnonterminal(item):
                    line_first.append(item)
                    break
                else:
                    line_first.extend(self.FIRST[item])
                    if 'ε' not in self.FIRST[item]:
                        break
            line_first = list(set(line_first))
            can_derive_empty = True
            part_end.reverse()
            for item in part_end:
                if not isnonterminal(item):
                    if item != 'ε':
                        can_derive_empty = False
                        break
                else:
                    if 'ε' not in self.FIRST[item]:
                        can_derive_empty = False
                        break
            list_remove = line_first.copy()
            if "ε" in list_remove:
                list_remove.remove("ε")
            if can_derive_empty:
                self.SELECT[line].extend(list_remove)
                self.SELECT[line].extend(self.FOLLOW[part_begin])
            else:
                self.SELECT[line].extend(list_remove)
            self.SELECT[line] = list(set(self.SELECT[line]))

    def debug_out(self):
        for i, j in self.FIRST.items():
            str = j[0]
            for temp in j[1:]:
                str = str + ',' + temp
            print("FIRST(" + i + ")" + " = {" + str + "}")

        for i, j in self.FOLLOW.items():
            str = j[0]
            for temp in j[1:]:
                str = str + ',' + temp
            print("FOLLOW(" + i + ")" + " = {" + str + "}")
        for i, j in self.SELECT.items():
            str = j[0]
            for temp in j[1:]:
                str = str + ',' + temp
            print("SELECT(" + i + ")" + " = {" + str + "}")

    def run_cal(self):
        self.getFirst()
        self.getFirst_3()
        self.getFirst_3()
        self.getFOLLOW_3()
        self.getFOLLOW_3()
        self.getSelect()
        # self.debug_out()


if __name__ == "__main__":
    CalSelect().run_cal()

```

