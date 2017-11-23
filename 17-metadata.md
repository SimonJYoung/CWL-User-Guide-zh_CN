## 元数据和作者信息

### 总览

问题：

* 如何给要引用工具描述的人提供信息？

目标：

* 学会在CWL描述中添加作者信息以及其他元数据

-----

Implementation extensions not required for correct execution (for example,
fields related to GUI presentation) and metadata about the tool or workflow
itself (for example, authorship for use in citations) may be provided as
additional fields on any object.
Such extensions fields must use a namespace prefix listed in the `$namespaces`
section of the document as described in the
[Schema Salad specification][schema-salad].

For all developers, we recommend the following minimal metadata for your tool
and workflows. This example includes metadata allowing others to cite your tool.

*metadata_example2.cwl*

~~~YAML
#!/usr/bin/env cwl-runner
cwlVersion: v1.0
class: CommandLineTool

label: An example tool demonstrating metadata.
doc: Note that this is an example and the metadata is not necessarily consistent.

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

s:author:
  - class: s:Person
    s:identifier: https://orcid.org/0000-0002-6130-1021
    s:email: mailto:dyuen@oicr.on.ca
    s:name: Denis Yuen

s:contributor:
  - class: s:Person
    s:identifier: http://orcid.org/0000-0002-7681-6415
    s:email: mailto:briandoconnor@gmail.com
    s:name: Brian O'Connor

s:citation: https://dx.doi.org/10.6084/m9.figshare.3115156.v2
s:codeRepository: https://github.com/common-workflow-language/common-workflow-language
s:dateCreated: "2016-12-13"
s:license: https://www.apache.org/licenses/LICENSE-2.0

$namespaces:
  s: https://schema.org/
  edam: http://edamontology.org/

$schemas:
 - https://schema.org/docs/schema_org_rdfa.html
 - http://edamontology.org/EDAM_1.18.owl

~~~

#### Extended Example

For those that are highly motivated, it is also possible to annotate your tool
with a much larger amount of metadata. This example includes EDAM ontology tags
as keywords (allowing the grouping of related tools), hints at hardware
requirements in order to use the tool, and a few more metadata fields.

*metadata_example3.cwl*

~~~YAML
#!/usr/bin/env cwl-runner
cwlVersion: v1.0
class: CommandLineTool

label: An example tool demonstrating metadata.
doc: Note that this is an example and the metadata is not necessarily consistent.

hints:
  ResourceRequirement:
    coresMin: 4

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

s:author:
  - class: s:Person
    s:identifier: https://orcid.org/0000-0002-6130-1021
    s:email: mailto:dyuen@oicr.on.ca
    s:name: Denis Yuen

s:contributor:
  - class: s:Person
    s:identifier: http://orcid.org/0000-0002-7681-6415
    s:email: mailto:briandoconnor@gmail.com
    s:name: Brian O'Connor

s:citation: https://dx.doi.org/10.6084/m9.figshare.3115156.v2
s:codeRepository: https://github.com/common-workflow-language/common-workflow-language
s:dateCreated: "2016-12-13"
s:license: https://www.apache.org/licenses/LICENSE-2.0

s:keywords: edam:topic_0091 , edam:topic_0622
s:programmingLanguage: C

$namespaces:
 s: https://schema.org/
 edam: https://edamontology.org/

$schemas:
 - https://schema.org/docs/schema_org_rdfa.html
 - http://edamontology.org/EDAM_1.18.owl

~~~

-----

要点：

* CWL描述中可以提供元数据
* 开发者需要提供简洁的作者信息供别人进行正确引用CWL工具

[schema-salad]: http://www.commonwl.org/v1.0/SchemaSalad.html#Explicit_context
