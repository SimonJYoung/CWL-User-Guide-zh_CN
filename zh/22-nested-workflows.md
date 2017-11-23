## 嵌套流程

### 总览

问题：

* 如何连接多个流程？

目标：

* 学会以流程的CWL文档为基础创建嵌套流程的CWL文档

-----

Workflows are ways to combine multiple tools to perform a larger operations.
We can also think of a workflow as being a tool itself; a CWL workflow can be
used as a step in another CWL workflow, if the workflow engine supports the
`SubworkflowFeatureRequirement`:


~~~YAML
requirements:
  - class: SubworkflowFeatureRequirement
~~~


Here's an example workflow that uses our `1st-workflow.cwl` as a nested
workflow:

~~~YAML
#!/usr/bin/env cwl-runner

cwlVersion: v1.0
class: Workflow

inputs: []

outputs:
  classout:
    type: File
    outputSource: compile/classout

requirements:
  - class: SubworkflowFeatureRequirement

steps:
  compile:
    run: 1st-workflow.cwl
    in:
      inp:
        source: create-tar/tar
      ex:
        default: "Hello.java"
    out: [classout]

  create-tar:
    requirements:
      - class: InitialWorkDirRequirement
        listing:
          - entryname: Hello.java
            entry: |
              public class Hello {
                public static void main(String[] argv) {
                    System.out.println("Hello from Java");
                }
              }
    in: []
    out: [tar]
    run:
      class: CommandLineTool
      requirements:
        - class: ShellCommandRequirement
      arguments:
        - shellQuote: false
          valueFrom: |
            date
            tar cf hello.tar Hello.java
            date
      inputs: []
      outputs:
        tar:
          type: File
          outputBinding:
            glob: "hello.tar"

~~~


A CWL `Workflow` can be used as a `step` just like a `CommandLineTool`, it's CWL
file is included with `run`. The workflow inputs (`inp` and `ex`) and outputs
(`classout`) then can be mapped to become the step's input/outputs.

~~~YAML
  compile:
    run: 1st-workflow.cwl
    in:
      inp:
        source: create-tar/tar
      ex:
        default: "Hello.java"
    out: [classout]
~~~

Our `1st-workflow.cwl` was parameterized with workflow inputs, so when running
it we had to provide a job file to denote the tar file and `*.java` filename.
This is generally best-practice, as it means it can be reused in multiple parent
workflows, or even in multiple steps within the same workflow.

Here we use `default:` to hard-code `"Hello.java"` as the `ex` input, however
our workflow also requires a tar file at `inp`, which we will prepare in the
`create-tar` step. At this point it is probably a good idea to refactor
`1st-workflow.cwl` to have more specific input/output names, as those also
appear in its usage as a tool.

It is also possible to do a less generic approach and avoid external
dependencies in the job file. So in this workflow we can generate a hard-coded
`Hello.java` file using the previously mentioned `InitialWorkDirRequirement`
requirement, before adding it to a tar file.

~~~YAML
  create-tar:
    requirements:
      - class: InitialWorkDirRequirement
        listing:
          - entryname: Hello.java
            entry: |
              public class Hello {
                public static void main(String[] argv) {
                    System.out.println("Hello from Java");
                }
              }
~~~


In this case our step can assume `Hello.java` rather than be parameterized, so
we can use a simpler `arguments` form as long as the CWL workflow engine
supports the `ShellCommandRequirement`:

~~~YAML
  run:
    class: CommandLineTool
    requirements:
      - class: ShellCommandRequirement
    arguments:
      - shellQuote: false
        valueFrom: >
          tar cf hello.tar Hello.java
~~~

Note the use of `shellQuote: false` here, otherwise the shell will try to
execute the quoted binary `"tar cf hello.tar Hello.java"`.

Here the `>` block means that newlines are stripped, so it's possible to write
the single command on multiple lines. Similarly, the `|` we used above will
preserve newlines, combined with `ShellCommandRequirement` this would allow
embedding a shell script.
Shell commands should however be used sparingly in CWL, as it means you
"jump out" of the workflow and no longer get reusable components, provenance or
scalability. For reproducibility and portability it is recommended to only use
shell commands together with a `DockerRequirement` hint, so that the commands
are executed in a predictable shell environment.

Did you notice that we didn't split out the `tar cf` tool to a separate file,
but rather embedded it within the CWL Workflow file? This is generally not best
practice, as the tool then can't be reused. The reason for doing it in this case
is because the command line is hard-coded with filenames that only make sense
within this workflow.

In this example we had to prepare a tar file outside, but only because our inner
workflow was designed to take that as an input. A better refactoring of the
inner workflow would be to take a list of Java files to compile, which would
simplify its usage as a tool step in other workflows.

Nested workflows can be a powerful feature to generate higher-level functional
and reusable workflow units - but just like for creating a CWL Tool description,
care must be taken to improve its usability in multiple workflows.

-----

要点：

* "A workflow can be used as a step in another workflow, if the workflow engine
supports the `SubworkflowFeatureRequirement`."
* "The workflows are specified under `steps`, with the worklow's description
file provided as the value to the `run` field."
* "Use `default` to specify a default value for a field, which can be
overwritten by a value in the input object."
* "Use `>` to ignore newlines in long commands split over multiple lines."
