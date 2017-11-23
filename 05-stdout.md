## 捕获标准输出

### 总览

问题：

* 如何捕获工具的标准输出流？

目标：

* 学会捕获工具的标准输出流

-----

为了捕获工具的标准输出流，添加`stdout`域并给其指定输出流写入的文件的名字。然后给对应的输出参数添加`type: stdout`。

*stdout.cwl*

```
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: echo
stdout: output.txt
inputs:
  message:
    type: string
    inputBinding:
      position: 1
outputs:
  output:
    type: stdout

```

*echo-job.yml*

```
message: Hello world!
```

有了工具的CWL和输入对象之后，调用执行器`cwl-runner`执行命令：

```
$ cwl-runner stdout.cwl echo-job.yml
[job stdout.cwl] /tmp/tmpE0gTz7$ echo \
    'Hello world!' > /tmp/tmpE0gTz7/output.txt
[job stdout.cwl] completed success
{
    "output": {
        "checksum": "sha1$47a013e660d408619d894b20806b1d5086aab03b",
        "basename": "output.txt",
        "nameroot": "output",
        "nameext": ".txt",
        "location": "file:///home/me/cwl/user_guide/output.txt",
        "path": "/home/me/cwl/user_guide/output.txt",
        "class": "File",
        "size": 13
    }
}
Final process status is success
```

-----

### 要点

* 使用`stdout`域指定一个文件名来捕获标准输出流
* 对应的输出参数必需有`type: stdout`
