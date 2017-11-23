## 文件格式

### 总览

问题：

* 如何检查输入文件和输出文件的格式？

目标：

* 学会清楚地指定`File`对象的格式

-----

工具和流程均能输入和输出文件，即`File`类型对象。我们推荐使用`type`域指示文件的格式。这能够对输入和输出的文件做格式上的检查，帮助别人来使用您的工具。

我们推荐使用现有的格式本体库（比如[EDAM][EDAM] 、[IANA][IANA] ），在和别人分享工具时，不要为了开发的快捷，轻易定义自己的文件格式。

指示文件的格式后，`cwltool`会做基本的格式检查，当有明显错配时会做出提示。

*metadata_example.cwl*

~~~
#!/usr/bin/env cwl-runner
cwlVersion: v1.0
class: CommandLineTool

label: An example tool demonstrating metadata.

inputs:
  aligned_sequences:
    type: File
    label: Aligned sequences in BAM format
    format: edam:format_2572
    inputBinding:
      position: 1

baseCommand: [ wc, -l ]

stdout: output.txt

outputs:
  report:
    type: stdout
    format: edam:format_1964
    label: A text file that contains a line count

$namespaces:
  edam: http://edamontology.org/
$schemas:
  - http://edamontology.org/EDAM_1.18.owl

~~~

下面是以上CWL工具的一个输入配置文件样例，也使用`type`域指示文件的格式，作为工具的检查点。

*sample.yml*

~~~
aligned_sequences:
    class: File
    format: http://edamontology.org/format_2572
    path: file-formats.bam
bamstats_report:
    class: File
    path: /tmp/bamstats_report.zip

~~~

调用`cwl-runner`执行任务：

~~~
$ cwl-runner metadata_example.cwl sample.yml
/usr/local/bin/cwl-runner 1.0.20161114152756
Resolved 'metadata_example.cwl' to 'file:///media/large_volume/testing/cwl_tutorial2/metadata_example.cwl'
[job metadata_example.cwl] /tmp/tmpNWyAd6$ /bin/sh \
    -c \
    'wc' '-l' '/tmp/tmpBf6m9u/stge293ac74-3d42-45c9-b506-dd35ea3e6eea/file-formats.bam' > /tmp/tmpNWyAd6/output.txt
Final process status is success
{
  "report": {
    "format": "http://edamontology.org/format_1964",
    "checksum": "sha1$49dc5004959ba9f1d07b8c00da9c46dd802cbe79",
    "basename": "output.txt",
    "location": "file:///media/large_volume/testing/cwl_tutorial2/output.txt",
    "path": "/media/large_volume/testing/cwl_tutorial2/output.txt",
    "class": "File",
    "size": 80
  }
}
~~~

-----

要点：

* 使用`format`域指定输入、输出文件的格式
* 使用CWL封装工具成熟时，推荐使用现有的格式本体库（比如EDAM）指定文件格式

[IANA]: https://www.iana.org/assignments/media-types/media-types.xhtml
[EDAM]: http://www.ebi.ac.uk/ols/ontologies/edam/terms?iri=http%3A%2F%2Fedamontology.org%2Fformat_1915
