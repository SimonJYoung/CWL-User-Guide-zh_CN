## 组织输入文件

### 总览

问题：

*  如何组织输入文件到工作目录下？

目标：

* 学会处理工具把文件输出到输入文件所在目录下的情况

-----

输入文件通常放在一个非输出目录的只读目录下。如果工具要将输出文件写到输入文件所在的目录下时，问题就来了。这个时候，可以使用`InitialWorkDirRequirement`将输入文件组织到输出目录（即Docker容器中的工作目录）下。
下面的例子中，我们使用JavaScript表达式`$(inputs.src)`获取输入文件。

*linkfile.cwl*

~~~
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
hints:
  DockerRequirement:
    dockerPull: java:7
baseCommand: javac

requirements:
  - class: InlineJavascriptRequirement
  - class: InitialWorkDirRequirement
    listing:
      - $(inputs.src)

inputs:
  src:
    type: File
    inputBinding:
      position: 1
      valueFrom: $(self.basename)

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

调用`cwl-runner`命令执行任务：

~~~
$ cwl-runner linkfile.cwl arguments-job.yml
[job 139928309171664] /home/example$ docker run -i --volume=/home/example/Hello.java:/var/lib/cwl/job557617295_examples/Hello.java:ro --volume=/home/example:/var/spool/cwl:rw --volume=/tmp/tmpmNbApw:/tmp:rw --workdir=/var/spool/cwl --read-only=true --net=none --user=1001 --rm --env=TMPDIR=/tmp java:7 javac Hello.java
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

-----

要点：

* 输入文件通常存放在只读目录下
* 使用`InitialWorkDirRequirement`安排输入文件到工作目录下
