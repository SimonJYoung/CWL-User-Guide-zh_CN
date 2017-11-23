## 队列型输入

### 总览

问题：

* 如何指定队列型输入参数？

目标：

* 学会给工具提供队列型参数作为输入
* 学会组织命令行中的队列型参数

-----

给命令行添加队列型输入比较容易。在`type`域下，嵌套指定`type`成队列，给`items`指定对应的数据类型，即可指定队列型输入参数。

*array-inputs.cwl*

~~~
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
inputs:
  filesA:
    type: string[]
    inputBinding:
      prefix: -A
      position: 1

  filesB:
    type:
      type: array
      items: string
      inputBinding:
        prefix: -B=
        separate: false
    inputBinding:
      position: 2

  filesC:
    type: string[]
    inputBinding:
      prefix: -C=
      itemSeparator: ","
      separate: false
      position: 4

outputs: []
baseCommand: echo

~~~

*array-inputs-job.yml*

~~~
filesA: [one, two, three]
filesB: [four, five, six]
filesC: [seven, eight, nine]

~~~

调用`cwl-runner`执行命令：

~~~
$ cwl-runner array-inputs.cwl array-inputs-job.yml
[job 140334923640912] /home/example$ echo -A one two three -B=four -B=five -B=six -C=seven,eight,nine
-A one two three -B=four -B=five -B=six -C=seven,eight,nine
Final process status is success
{}
~~~

`inputBinding`可以出现在外层和内层的参数定义中，外层定义队列，内层定义元素，不同的地方产生不同的行为。

另外，如果提供`itemSeparator`域，即可指定队列元素间的分隔方式，将一个字符串参数拆分成队列。

注意，在`array-inputs-job.yml`中，用方括号`[]`也可用来指定队列型输入参数。

队列元素可以是队列、记录或其他复杂的类型。

-----

### 要点

* 限定队列型参数时将`type`域赋值成队列
* 队列型参数在命令行中的出现方式依赖于提供的`inputBinding`域
* 使用`itemSeparator`域来定义队列型参数元素间的连接字符
