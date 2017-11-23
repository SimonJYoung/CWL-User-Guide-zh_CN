## 输出文件队列

### 总览

问题：

* 如何指定输出文件队列？

目标：

* 学会创建输出文件队列

-----

您可以使用`glob`捕获多个输出文件。下面的例子中，捕获的队列元素为以`.txt`结尾的两个文件，`foo.txt`和`baz.txt`。

*array-outputs.cwl*

~~~
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: touch
inputs:
  touchfiles:
    type:
      type: array
      items: string
    inputBinding:
      position: 1
outputs:
  output:
    type:
      type: array
      items: File
    outputBinding:
      glob: "*.txt"

~~~

*array-outputs-job.yml*

~~~
touchfiles:
  - foo.txt
  - bar.dat
  - baz.txt

~~~

调用`cwl-runner`执行命令：

~~~
$ cwl-runner array-outputs.cwl array-outputs-job.yml
[job 140190876078160] /home/example$ touch foo.txt bar.dat baz.txt
Final process status is success
{
  "output": [
    {
      "size": 0,
      "location": "/home/peter/work/common-workflow-language/draft-3/examples/foo.txt",
      "checksum": "sha1$da39a3ee5e6b4b0d3255bfef95601890afd80709",
      "class": "File"
    },
    {
      "size": 0,
      "location": "/home/peter/work/common-workflow-language/draft-3/examples/baz.txt",
      "checksum": "sha1$da39a3ee5e6b4b0d3255bfef95601890afd80709",
      "class": "File"
    }
  ]
}
~~~

`array-outputs-job.yml`文件中，用`-`开头标记每个元素是指定队列型参数的另一种方式。

-----

要点：

* 使用`glob`域捕获多个输出文件
* 使用通配符和文件名指定工具执行完成后要返回的输出文件
