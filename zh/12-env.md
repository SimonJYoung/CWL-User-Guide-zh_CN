## 环境变量

### 总览

问题：

* 如何设置工具执行时环境变量的值？

目标：

* 学会设置工具运行时的环境变量

-----

CWL工具运行在一个环境变量很少的限制型环境中，没有继承父任务的大多数环境变量。我们可以使用`EnvVarRequirement`域设置工具运行时的环境变量。

*env.cwl*

~~~
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
baseCommand: env
requirements:
  EnvVarRequirement:
    envDef:
      HELLO: $(inputs.message)
inputs:
  message: string
outputs: []

~~~

*echo-job.yml*

~~~
message: Hello world!
~~~

调用`cwl-runner`执行命令：

~~~
$ cwl-runner env.cwl echo-job.yml
[job 140710387785808] /home/example$ env
PATH=/bin:/usr/bin:/usr/local/bin
HELLO=Hello world!
TMPDIR=/tmp/tmp63Obpk
Final process status is success
{}
~~~

-----

要点：

* 工具运行在一个环境变量很少的限制型环境中
* 使用`EnvVarRequirement`域设置工具运行时的环境变量
