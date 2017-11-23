## 参数引用

### 总览

问题：

* 如何在其他域中引用输入参数？

目标：

* 学会引用参数

-----

在上面的例子中，我们用tar程序解压文件。但是，这样的情况非常有限，因为我们我们感兴趣的文件就叫“hello.txt”。在下面的例子中，您将看到如何从其他域中动态地引用输入参数值。

*tar-param.cwl*

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
  extractfile:
    type: string
    inputBinding:
      position: 2
outputs:
  example_out:
    type: File
    outputBinding:
      glob: $(inputs.extractfile)

~~~

*tar-param-job.yml*

~~~
tarfile:
  class: File
  path: hello.tar
extractfile: goodbye.txt

~~~

创建输入文件，调用执行器`cwl-runner`执行命令：

~~~
$ rm hello.tar || true && touch goodbye.txt && tar -cvf hello.tar goodbye.txt
$ cwl-runner tar-param.cwl tar-param-job.yml
[job tar-param.cwl] /tmp/tmpwH4ouT$ tar \
    xf \
    /tmp/tmpREYiEt/stgd7764383-99c9-4848-af51-7c2d6e5527d9/hello.tar \
    goodbye.txt
[job tar-param.cwl] completed success
{
    "example_out": {
        "checksum": "sha1$da39a3ee5e6b4b0d3255bfef95601890afd80709",
        "basename": "goodbye.txt",
        "nameroot": "goodbye",
        "nameext": ".txt",
        "location": "file:///home/me/cwl/user_guide/goodbye.txt",
        "path": "/home/me/cwl/user_guide/goodbye.txt",
        "class": "File",
        "size": 0
    }
}
Final process status is success
~~~

特定的域允许使用`$(...)`的方式引用参数，其中用相应的被引用值替代。

~~~
outputs:
  example_out:
    type: File
    outputBinding:
      glob: $(inputs.extractfile)
~~~

引用使用JavaScript语法子集书写。这上面的例子中，`$(inputs.extractfile)`、 `$(inputs["extractfile"])` 和 `$(inputs['extractfile'])`的写法是等价的。

变量"inputs"的值是调用CWL工具时提供的输入对象。

注意，因为`File`参数是对象，获取一个输入文件的路径时，需要引用文件对象的`path`域，上面例子中，引用tar压缩文件路径的写法为`$(inputs.tarfile.path)`。

-----

### 要点

* 某些域允许用`$(...)`的方式引用参数
* 引用使用JavaScript语法子集书写
