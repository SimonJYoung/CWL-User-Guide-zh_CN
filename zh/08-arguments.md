## 额外的参数

### 总览

问题：

* 如何指定不需要给输入值的参数？
* 如何指定运行参数？

目标：

* 学会给命令添加额外的选项
* 学会引用运行参数

-----

有时候工具需要的额外命令行选项并不确切地对应输入参数。

在该例子中，我们将封装Java编译器来编译java源文件成类文件。默认情况下，javac会在源文件同目录下创建类文件。但是，CWL的输入文件（及其所在的目录）可能是只读的，因此，我们需要设置javac将类文件生成到指定的输出目录下。

*arguments.cwl*

~~~
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
label: Example trivial wrapper for Java 7 compiler
hints:
  DockerRequirement:
    dockerPull: java:7-jdk
baseCommand: javac
arguments: ["-d", $(runtime.outdir)]
inputs:
  src:
    type: File
    inputBinding:
      position: 1
outputs:
  classfile:
    type: File
    outputBinding:
      glob: "*.class"

~~~

*arguments-job.yml*

~~~
src:
  class: File
  path: Hello.java

~~~

现在创建一个简单的Java文件，调用CWL执行器`cwl-runner`执行命令：

~~~
$ echo "public class Hello {}" > Hello.java
$ cwl-runner arguments.cwl arguments-job.yml
[job arguments.cwl] /tmp/tmpwYALo1$ docker \
 run \
 -i \
 --volume=/home/peter/work/common-workflow-language/v1.0/examples/Hello.java:/var/lib/cwl/stg8939ac04-7443-4990-a518-1855b2322141/Hello.java:ro \
 --volume=/tmp/tmpwYALo1:/var/spool/cwl:rw \
 --volume=/tmp/tmpptIAJ8:/tmp:rw \
 --workdir=/var/spool/cwl \
 --read-only=true \
 --user=1001 \
 --rm \
 --env=TMPDIR=/tmp \
 --env=HOME=/var/spool/cwl \
 java:7 \
 javac \
 -d \
 /var/spool/cwl \
 /var/lib/cwl/stg8939ac04-7443-4990-a518-1855b2322141/Hello.java
Final process status is success
{
  "classfile": {
    "size": 416,
    "location": "/home/example/Hello.class",
    "checksum": "sha1$2f7ac33c1f3aac3f1fec7b936b6562422c85b38a",
    "class": "File"
  }
}

~~~

这里我们使用`arguments`域来添加额外的参数到命令行，不用缀着特定的输入参数值。

~~~
arguments: ["-d", $(runtime.outdir)]
~~~

这个例子引用了运行时参数。运行时参数提供工具真正执行时关于硬件和软件的信息。`$(runtime.outdir)`参数是指定给输出目录的路径。其他参数包括
`$(runtime.tmpdir)`、 `$(runtime.ram)`、 `$(runtime.cores)`、 `$(runtime.outdirSize)`和`$(runtime.tmpdirSize)`。更详细说明见[运行时环境][runtime]。

-----

### 要点

* 使用`arguments`节描述不同指定输入参数的命令行选项
* 运行时参数提供关于工具真正执行时的环境详细
* 运行时参数在`runtime`命名空间下

[runtime]: http://www.commonwl.org/v1.0/CommandLineTool.html#Runtime_environment
