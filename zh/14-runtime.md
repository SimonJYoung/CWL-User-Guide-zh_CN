## 在运行时创建文件

### 总览

问题：

* 如何在运行时，通过输入参数创建需要的输入文件？


目标：

* 学会在运行时创建文件

-----

有时需要通过输入参数创建及时的文件，比如工具想通过读配置文件而非读命令行，就在`InitialWorkDirRequirement`域中做相应的描述。

*createfile.cwl*

~~~YAML
#!/usr/bin/env cwl-runner

class: CommandLineTool
cwlVersion: v1.0
baseCommand: ["cat", "example.conf"]

requirements:
  InitialWorkDirRequirement:
    listing:
      - entryname: example.conf
        entry: |
          CONFIGVAR=$(inputs.message)

inputs:
  message: string
outputs: []

~~~

*echo-job.yml*

~~~YAML
message: Hello world!

~~~

调用`cwl-runner`执行命令：

~~~
$ cwltool createfile.cwl echo-job.yml
[job 140528604979344] /home/example$ cat example.conf
CONFIGVAR=Hello world!
Final process status is success
{}
~~~

-----

要点：

* 使用`InitialWorkDirRequirement`指定需要的工具运行时创建的输入文件。
