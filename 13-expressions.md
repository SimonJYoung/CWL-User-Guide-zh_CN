## JavaScript动态表达式

### 总览

问题：

* 在CWL没有提供内建方法的地方，如何创建动态参数值？

目标：

* 学会在CWL描述中使用JavaScript动态表达式

-----

如果需要处理和修改输入参数，可以使用在`requirements`中添加`InlineJavascriptRequirement`，在合法的地方写JavaScript表达式，CWL执行器会进行相应的计算，提供参数给被封装的工具。

> 注意：
> 若非迫不得已的，尽量不用JavaScript表达式。
> 当处理文件名、文件扩展名、路径等时，可以考虑使用[内建的`File`特征][file-prop]，如`basename`、 `nameroot`、 `nameext`等替代。

*expression.cwl*

```YAML
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: echo

requirements:
  - class: InlineJavascriptRequirement

inputs: []
outputs: []
arguments:
  - prefix: -A
    valueFrom: $(1+1)
  - prefix: -B
    valueFrom: $("/foo/bar/baz".split('/').slice(-1)[0])
  - prefix: -C
    valueFrom: |
      ${
        var r = [];
        for (var i = 10; i >= 1; i--) {
          r.push(i);
        }
        return r;
      }

```

因为该工具没有要求任何输入，所以我们提供无内容的文件。

*empty.yml*

```
{}

```

`empty.yml`由一个空的JSON对象构成。JSON对象在花括号`{}`中进行描述。只剩花括号表示是空对象。

运行`expression.cwl`：

~~~
$ cwl-runner expression.cwl empty.yml
[job 140000594593168] /home/example$ echo -A 2 -B baz -C 10 9 8 7 6 5 4 3 2 1
-A 2 -B baz -C 10 9 8 7 6 5 4 3 2 1
Final process status is success
{}
~~~

注意，`requirements`节是一个队列，添加项（如这里的`class: InlineJavascriptRequirement`）时，要在前面加中横线`-`。语法和描述额外参数是一样的。

只有在一下的域中才能使用JavaScript表达式：

* `filename`
* `fileContent`
* `envValue`
* `valueFrom`
* `glob`
* `outputEval`
* `stdin`
* `stdout`
* `coresMin`
* `coresMax`
* `ramMin`
* `ramMax`
* `tmpdirMin`
* `tmpdirMax`
* `outdirMin`
* `outdirMax`

-----

要点：

* 如果指定了`InlineJavascriptRequirement`，即可在描述中使用JavaScript表达式，使其在任务执行时进行动态计算
* JavaScript表达式只能在确定的某些域中有效
* 在CWL没有提供内建方法的地方才应该使用JavaScript表达式

[file-prop]: http://www.commonwl.org/v1.0/CommandLineTool.html#File
