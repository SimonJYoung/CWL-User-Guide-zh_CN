## 高级输入

### 总览

问题：

* 如何描述依赖型参数和互斥型参数？

目标：

* 学会使用记录来描述输入文件之间的关系

-----

有时候工具的有些参数需要同时提供（它们有依赖关系），而有些又不能放在一起（它们相互排斥）。这个时候可以使用`record`类型和类型单元将参数组织在一起，分别描述这两种情况。

*record.cwl*

~~~
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: CommandLineTool
inputs:
  dependent_parameters:
    type:
      type: record
      name: dependent_parameters
      fields:
        itemA:
          type: string
          inputBinding:
            prefix: -A
        itemB:
          type: string
          inputBinding:
            prefix: -B
  exclusive_parameters:
    type:
      type: record
      name: itemC
      fields:
        itemC:
          type: string
          inputBinding:
            prefix: -C
      type: record
      name: itemD
      fields:
        itemD:
          type: string
          inputBinding:
            prefix: -D
outputs: []
baseCommand: echo

~~~

*record-job1.yml*

~~~
dependent_parameters:
  itemA: one
exclusive_parameters:
  itemC: three

~~~

~~~
$ cwl-runner record.cwl record-job1.yml
Workflow error:
  Error validating input record, could not validate field `dependent_parameters` because
  missing required field `itemB`
~~~

在以上的例子中，不能只提供`itemA`，而不提供`itemB`。

*record-job2.yml*

~~~
dependent_parameters:
  itemA: one
  itemB: two
exclusive_parameters:
  itemC: three
  itemD: four

~~~

~~~
$ cwl-runner record.cwl record-job2.yml
[job 140566927111376] /home/example$ echo -A one -B two -C three
-A one -B two -C three
Final process status is success
{}
~~~

在以上的例子中，`itemC`和`itemD`相互排斥，因此，只会将`itemC`加入命令行，而忽略`itemD`。

*record-job3.yml*

~~~
dependent_parameters:
  itemA: one
  itemB: two
exclusive_parameters:
  itemD: four

~~~

~~~
$ cwl-runner record.cwl record-job3.yml
[job 140606932172880] /home/example$ echo -A one -B two -D four
-A one -B two -D four
Final process status is success
{}
~~~

在以上的例子中，因为只提供了`itemD`，所以它才出现在了命令行中。

-----

要点：

* 使用`record`域将参数组织在一起
* 同一个参数描述内的多个`record`之间是互斥的
