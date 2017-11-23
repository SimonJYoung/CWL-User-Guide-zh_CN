## 在Docker中运行工具

### 总览

问题：

* 如何在Docker容器中运行工具？

目标：

* 学会在一个完全受控的环境中调用工具

-----

[Docker][docker]容器通过给软件及其依赖提供完整而良好的运行环境简化了软件的安装。但是，容器因与宿主系统隔离，为了在Docker容器中运行工具，需要有额外的工作来确保在容器中能获取到外部的输入文件并能将输出的文件从容器中取回。CWL执行器能够自动地做这个工作，让您能够使用Docker简化软件的管理，避免了调用、管理Docker容器的复杂性工作。

这个例子里，在Docker容器中运行一个简单的Node.js脚本。

*docker.cwl*

~~~
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: node
hints:
  DockerRequirement:
    dockerPull: node:slim
inputs:
  src:
    type: File
    inputBinding:
      position: 1
outputs: []

~~~

*docker-job.yml*

~~~
src:
  class: File
  path: hello.js

~~~

创建一个名为hello.js的脚本，调用CWL执行器`cwl-runner`执行命令：

~~~
$ echo "console.log(\"Hello World\");" > hello.js
$ cwl-runner docker.cwl docker-job.yml
[job docker.cwl] /tmp/tmpgugLND$ docker \
    run \
    -i \
    --volume=/tmp/tmpgugLND:/var/spool/cwl:rw \
    --volume=/tmp/tmpSs5JoN:/tmp:rw \
    --volume=/home/me/cwl/user_guide/hello.js:/var/lib/cwl/stg16848d97-e6ba-4b35-b666-4546d9965a2d/hello.js:ro \
    --workdir=/var/spool/cwl \
    --read-only=true \
    --user=1000 \
    --rm \
    --env=TMPDIR=/tmp \
    --env=HOME=/var/spool/cwl \
    node:slim \
    node \
    /var/lib/cwl/stg16848d97-e6ba-4b35-b666-4546d9965a2d/hello.js
Hello World
[job docker.cwl] completed success
{}
Final process status is success
~~~

注意，CWL执行器已经构建了一个Docker命令行来运行脚本。CWL执行器的任务之一是将输入文件在外部的路径映射成容器中的路径。在这个例子中，脚本`hello.js`在容器外部的路径为`/home/me/cwl/user_guide/hello.js`，在容器内部的路径为`/var/lib/cwl/job369354770_examples/hello.js`，`node`命令通过内部路径执行该脚本。

-----

### 要点

* 容器能够帮助简化工具需要的依赖软件的管理
* 在`hints`节中，给`DockerRequirement`指定工具的Docker镜像

[docker]: https://docker.io
