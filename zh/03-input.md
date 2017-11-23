## 基本的输入参数

### 总览

问题：

* 如何描述一个命令的输入？
* 如何指定输入在命令中的顺序？

目标：

* 学会如何描述和处理工具的输入参数和输入文件

---

给工具的`inputs`域指定是控制该工具如何执行的输入参数列表。每一个输入参数有一个`id`用于命名该参数、一个`type`用于描述什么类型的值指定给该参数是可用的。

有效的基本类型包括`string`、`int`、`long`、`float`、`double`和`null`，复杂类型包括`array`和`record`，另外还有特殊类型`File`、`Directory`和`Any`。

下面的例子用来说明某些不同类型的输入参数及其他们在命令行中出现的不同方式：

*inp.cwl*

```
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: echo
inputs:
  example_flag:
    type: boolean
    inputBinding:
      position: 1
      prefix: -f
  example_string:
    type: string
    inputBinding:
      position: 3
      prefix: --example-string
  example_int:
    type: int
    inputBinding:
      position: 2
      prefix: -i
      separate: false
  example_file:
    type: File?
    inputBinding:
      prefix: --file=
      separate: false
      position: 4

outputs: []
```

*inp-job.yml*

```
example_flag: true
example_string: hello
example_int: 42
example_file:
  class: File
  path: whale.txt
```

注意“example\_file”作为`File`类型，必需用提供一个对象，它拥有`class`和`path`两个域，且`class`的值为`File`。

接下来，创建一个名为whale.txt的文件，调用CWL执行器cwl-runner执行工具命令行：

```
$ touch whale.txt
$ cwl-runner inp.cwl inp-job.yml
[job inp.cwl] /tmp/tmpzrSnfX$ echo \
    -f \
    -i42 \
    --example-string \
    hello \
    --file=/tmp/tmpRBSHIG/stg979b6d24-d50a-47e3-9e9e-90097eed2cbc/whale.txt
-f -i42 --example-string hello --file=/tmp/tmpRBSHIG/stg979b6d24-d50a-47e3-9e9e-90097eed2cbc/whale.txt
[job inp.cwl] completed success
{}
Final process status is success
```

### 哪来的/tmp路径？

CWL参考执行器cwltool和其他执行器会创建临时目录，将输入文件软连接到其中，以确保工具不会意外地访问未明确指定的文件。  
可选的`inputBinding`域可以用来指示输入参数是否以及该如何出现在工具的命令行中。如果`inputBinding`域缺失，参数将不会出现在命令行中。看下面的例子来了解细节。

```
example_flag:
  type: boolean
  inputBinding:
    position: 1
    prefix: -f
```

布尔类型（boolean）参数被当做flag。如果输入参数`example_flag`是真，那么该前缀`-f`将被加入命令行中。否则，不加入。

```
example_string:
  type: string
  inputBinding:
    position: 3
    prefix: --example-string
```

字符串（string）类型的参数以字符串的方式出现在命令行中。前缀`prefix`是可选的，如果提供，它将在命令行中出现在参数值的前面。在上面的例子中，就是`--example-string hello`。

```
example_int:
  type: int
  inputBinding:
    position: 2
    prefix: -i
    separate: false
```

整型（int）和浮点型（float）参数值以十进制数值出现在命令行中。如果选项`separate`给的是假`false`（默认值为真），那么该参数和值将连在一起。在上面的例子中为`-i42`。

```
example_file:
  type: File?
  inputBinding:
    prefix: --file=
    separate: false
    position: 4
```

文件（File）类型参数值以文件路径的方式出现在命令行中。当参数类型以问号（?）结尾表示该参数是可选参数。上面的例子中，命令行中出现的是`--file=/tmp/random/path/whale.txt`。但是，如果如果没有给参数`example_file`赋文件，（包括参数`--file=`也）将不会出现在命令行中。

输入文件只做读取。如果想更新输入文件，您必需先将其复制到输出目录中。

参数值的位置`position`用来确定参数出现在命令行中的位置。位置是相对的，而非绝对。因此，位置可以不是连续的，位置赋了1、3、5的在命令行中出现的位置依次是第一、第二和第三。多个参数可以有相同的`position`，并且`position`域本身是可选的，其默认值为0。

`baseCommand`域在命令行中出现的所有参数的前面。

---

### 要点

* 在CWL文档的`inputs`节中描述输入
* 文件描述需要有`class: File`
* 用`inputBinding`节描述输在命令行中出现的位置和方式
