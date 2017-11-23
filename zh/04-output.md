## 返回输出文件

### 总览

问题：

* 如何描述工具的输出？

目标：

* 学会描述和处理工具的输出

-----

工具的`outputs`节用来描述工具执行完成后返回的一系列输出。每个输出有一个标识作为参数名，以及`type`用来该参数值的有效类型。

当用CWL运行工具，启动的工作目录将被指定为输出目录。工具和脚本将以在输出目录中创建文件记录结果。CWL工具返回的输出参数要么输出文件本身，要么是来着输出文件内容的检查。

*tar.cwl*

~~~
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: [tar, xf]
inputs:
  tarfile:
    type: File
    inputBinding:
      position: 1
outputs:
  example_out:
    type: File
    outputBinding:
      glob: hello.txt
~~~

*tar-job.yml*

~~~
tarfile:
  class: File
  path: hello.tar
~~~

接着，创建一个tar格式的压缩文件作为例子，调用CWL执行器`cwl-runner`执行工具命令：

~~~
$ touch hello.txt && tar -cvf hello.tar hello.txt
$ cwl-runner tar.cwl tar-job.yml
[job tar.cwl] /tmp/tmpqOeawQ$ tar \
    xf \
    /tmp/tmpGDk8Y1/stg80bbad20-494d-47af-8075-dffc32df03a3/hello.tar
[job tar.cwl] completed success
{
    "example_out": {
        "checksum": "sha1$da39a3ee5e6b4b0d3255bfef95601890afd80709",
        "basename": "hello.txt",
        "nameroot": "hello",
        "nameext": ".txt",
        "location": "file:///home/me/cwl/user_guide/hello.txt",
        "path": "/home/me/cwl/user_guide/hello.txt",
        "class": "File",
        "size": 0
    }
}
Final process status is success
~~~

`outputBinding`域描述如何设置每个输出参数的值。

~~~
outputs:
  example_out:
    type: File
    outputBinding:
      glob: hello.txt
~~~

`glob`域由输出目录下文件的名字构成。如果提前不知道文件的具体名字，但知道其后缀为txt，可用`*.txt`模式匹配。

-----

### 要点

* CWL的输出在`outputs`节中描述
* `outputBinding`域描述如何设置每个输出参数的值。
* 通配符`*`可用做`glob`与中进行模式匹配。
