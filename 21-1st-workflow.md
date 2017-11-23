## 书写流程

### 总览

问题：

* 如何将工具连接成流程？

目标：

* 学会以工具的CWL文档为基础创建流程的CWL文档

-----

这个例子中的流程实现的功能是，先从tar压缩文件中提取java源文件，然后编译源文件成class文件。

*1st-workflow.cwl*

~~~YAML
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: Workflow
inputs:
  inp: File
  ex: string

outputs:
  classout:
    type: File
    outputSource: compile/classfile

steps:
  untar:
    run: tar-param.cwl
    in:
      tarfile: inp
      extractfile: ex
    out: [example_out]

  compile:
    run: arguments.cwl
    in:
      src: untar/example_out
    out: [classfile]

~~~

在另外一个文件中描述任务输入：

*1st-workflow-job.yml*

~~~YAML
inp:
  class: File
  path: hello.tar
ex: Hello.java

~~~

调用`cwl-runner`执行命令：

~~~
$ echo "public class Hello {}" > Hello.java && tar -cvf hello.tar Hello.java
$ cwl-runner 1st-workflow.cwl 1st-workflow-job.yml
[job untar] /tmp/tmp94qFiM$ tar xf /home/example/hello.tar Hello.java
[step untar] completion status is success
[job compile] /tmp/tmpu1iaKL$ docker run -i --volume=/tmp/tmp94qFiM/Hello.java:/var/lib/cwl/job301600808_tmp94qFiM/Hello.java:ro --volume=/tmp/tmpu1iaKL:/var/spool/cwl:rw --volume=/tmp/tmpfZnNdR:/tmp:rw --workdir=/var/spool/cwl --read-only=true --net=none --user=1001 --rm --env=TMPDIR=/tmp java:7 javac -d /var/spool/cwl /var/lib/cwl/job301600808_tmp94qFiM/Hello.java
[step compile] completion status is success
[workflow 1st-workflow.cwl] outdir is /home/example
Final process status is success
{
  "classout": {
    "location": "/home/example/Hello.class",
    "checksum": "sha1$e68df795c0686e9aa1a1195536bd900f5f417b18",
    "class": "File",
    "size": 416
  }
}
~~~

接下来，让我们分解一下CWL：

~~~YAML
cwlVersion: v1.0
class: Workflow
~~~

`cwlVersion`域指示该文档使用的CWL的版本。 class域指示该文档描述的是工作流。

~~~YAML
inputs:
  inp: File
  ex: string
~~~

`inputs`节描述流程的输入，一个输入参数的列表，每个参数的描述由标识、`type`（数据类型）等构成。这些参数可以作为流程某个步骤输入的源。

~~~YAML
outputs:
  classout:
    type: File
    outputSource: compile/classfile
~~~

`inputs`节描述流程的输出，一个输出参数的列表，每个输出参数的描述由标识、`type`（数据类型）等构成。`outputSource`将编译步骤的输出参数`classfile`指定为流程的输出参数`classout`。

~~~YAML
steps:
  untar:
    run: tar-param.cwl
    in:
      tarfile: inp
      extractfile: ex
    outputs: [example_out]
~~~

`steps`节描述流程的步骤。这上面的例子中，第一步是从tar压缩文件中提取java源文件，第二步是使用java编译器编译这个源文件。流程步骤的不是按其在CWL文档中罗列的顺序执行，而是根据步骤间的依赖关系确定执行的先后顺序。并且不相互依赖的步骤是可以并行执行的。

第一步解压缩（步骤标识为`untar`）执行的是CWL文档`tar-param.cwl`（在[参数应用][params]中描述过的CWL工具）。该工具有两个输入参数`tarfile`和`extractfile`，一个输出参数`example_out`。

在流程的`inputs`节中，将解压缩步骤的这两个输入参数指定为整个流程的输入参数`inp`和`ex`。这意味着，当执行解压缩步骤的时候，会将流程输入参数`inp`和`ex`的值传递给解压缩步骤的这两个输入参数`tarfile`和`extractfile`。

步骤的`outputs`节列出了该步的期望输出。

~~~YAML
  compile:
    run: arguments.cwl
    in:
      src: untar/example_out
    outputs: [classfile]

~~~

第二步编译（步骤标识为`compile`）的输入参数`src`依赖的是第一步解压缩（步骤标识为`untar`）的输出参数`example_out`，因此将`src`的值指定为`untar/example_out`。
如上所述，在流程的`outputs`节中，将第二步编译的输出`classfile`指定成了流程的输出。

-----

要点：

* 流程的每个步骤都有自己的CWL描述
* 流程最外层的输入和输出分别在`inputs`和`outputs`进行描述
* 流程的步骤在`steps`中描述
* 步骤的执行顺序有步骤间的输入、输出依赖关系决定

[params]: 06-params.md
