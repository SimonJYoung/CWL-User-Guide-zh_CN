## 第一个例子

### 总览

问题：

* 如何封装一个简单的命令行工具？

目标：

* 了解CWL的基本结构

---

最简单的“hello world”程序。该程序接受一个输入参数，输出一条消息到终端或工作日志文件，不产生其他固定输出文件。 CWL文档使用JSON或YAML或二者混合的格式书写。

_1st-tool.cwl_

```
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: echo
inputs:
  message:
    type: string
    inputBinding:
      position: 1
outputs: []
```

在另一个文件中使用YAML或JSON格式描述任务的输入：

_echo-job.yml_

```
message: Hello world!
```

现在调用CWL执行器`cwl-runner` 执行封装工具的命令行：

```
$ cwl-runner 1st-tool.cwl echo-job.yml
[job 1st-tool.cwl] /tmp/tmpmM5S_1$ echo \
    'Hello world!'
Hello world!
[job 1st-tool.cwl] completed success
{}
Final process status is success
```

接下来让我们分解一下CWL：

```
cwlVersion: v1.0
class: CommandLineTool
```

`cwlVersion`域指示该文档使用的CWL的版本。 class域指示该文档描述的是命令行工具。

```
baseCommand: echo
```

`baseCommand`域提供了实际执行的程序的名字为echo。

```
inputs:
  message:
    type: string
      inputBinding:
        position: 1
```

`inputs`节描述工具的输入。这里可以是一个输入参数的列表，每个参数包含标识、`type`（数据类型）和 `inputBinding`（用于描述该输入参数该如何出现在命令行中）。 在这个例子中，position域指示了其在命令行中该出现的位置。

```
outputs: []
```

该工具没有正式的输出，因此`output`节为空列表。

---

### 要点

* CWL文档用YAML和（或）JSON格式书写
* 将调用的程序名指定给`baseCommand`
* 在`inputs`节中描述每一个期望的输入
* 输入的值用另一个YAML文件指定
* 工具描述和输入文件提供给CWL执行器作为参数



