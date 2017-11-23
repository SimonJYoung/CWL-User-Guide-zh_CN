## 定制类型

### 总览

问题：

* 如何创建和导入自己定制的类型到CWL描述中？

目标：

* 学会创建自定义的CWL对象类型
* 学会将自定义的对象导入到工具描述中

-----

Sometimes you may want to write your own custom types for use and reuse in CWL
descriptions. Use of such custom types can reduce redundancy between multiple
descriptions that all use the same type, and also allow for additional
customisation/configuration of a tool/analysis without the need to fiddle with
the CWL description directly.

The example below is a CWL description of the [InterProScan][ips] tool for
simultaneously searching protein sequences against a wide variety of resources.
It is a good example of a number of good practices in CWL.

*custom-types.cwl*

~~~YAML
cwlVersion: v1.0
class: CommandLineTool

label: "InterProScan: protein sequence classifier"

doc: |
      Version 5.21-60 can be downloaded here:
      https://github.com/ebi-pf-team/interproscan/wiki/HowToDownload

      Documentation on how to run InterProScan 5 can be found here:
      https://github.com/ebi-pf-team/interproscan/wiki/HowToRun

requirements:
  ResourceRequirement:
    ramMin: 10240
    coresMin: 3
  SchemaDefRequirement:
    types:
      - $import: InterProScan-apps.yml

hints:
  SoftwareRequirement:
    packages:
      interproscan:
        specs: [ "https://identifiers.org/rrid/RRID:SCR_005829" ]
        version: [ "5.21-60" ]

inputs:
  proteinFile:
    type: File
    inputBinding:
      prefix: --input
  applications:
    type: InterProScan-apps.yml#apps[]?
    inputBinding:
      itemSeparator: ','
      prefix: --applications

baseCommand: interproscan.sh

arguments:
 - valueFrom: $(inputs.proteinFile.nameroot).i5_annotations
   prefix: --outfile
 - valueFrom: TSV
   prefix: --formats
 - --disable-precalc
 - --goterms
 - --pathways
 - valueFrom: $(runtime.tmpdir)
   prefix: --tempdir

outputs:
  i5Annotations:
    type: File
    format: iana:text/tab-separated-values
    outputBinding:
      glob: $(inputs.proteinFile.nameroot).i5_annotations

$namespaces:
 iana: https://www.iana.org/assignments/media-types/
 s: http://schema.org/
$schemas:
 - https://schema.org/docs/schema_org_rdfa.html

s:license: "https://www.apache.org/licenses/LICENSE-2.0"
s:copyrightHolder: "EMBL - European Bioinformatics Institute"

~~~

*custom-types.yml*

~~~
proteinFile:
    class: File
    path: test_proteins.fasta

~~~

On line 34, in `inputs:applications`, a list of applications to be used in the
search are imported as a custom object:

```
inputs:
  proteinFile:
    type: File
    inputBinding:
      prefix: --input
  applications:
    type: InterProScan-apps.yml#apps[]?
    inputBinding:
      itemSeparator: ','
      prefix: --applications
```


The reference to a custom type is a combination of the name of the file in which
the object is defined (`InterProScan-apps.yml`) and the name of the object
within that file (`apps`) that defines the custom type. The square brackets `[]`
mean that an array of the preceding type is expected, in this case the `apps`
type from the imported `InterProScan-apps.yaml` file

The contents of the YAML file describing the custom type are given below:

~~~YAML
type: enum
name: apps
symbols:
 - TIGRFAM
 - SFLD
 - SUPERFAMILY
 - Gene3D
 - Hamap
 - Coils
 - ProSiteProfiles
 - SMART
 - CDD
 - PRINTS
 - PIRSF
 - ProSitePatterns
 - Pfam
 - ProDom
 - MobiDBLite

~~~


In order for the custom type to be used in the CWL description, it must be
imported. Imports are described in `requirements:SchemaDefRequirement`, as
below in the example `custom-types.cwl` description:

```YAML
requirements:
  ResourceRequirement:
    ramMin: 10240
    coresMin: 3
  SchemaDefRequirement:
    types:
      - $import: InterProScan-apps.yml
```


Note also that the author of this CWL description has also included
`ResourceRequirement`s, specifying the minimum amount of RAM and number of cores
required for the tool to run successfully, as well as details of the version of
the software that the description was written for and other useful metadata.
These features are discussed further in other chapters of this user guide.

-----

要点：

* "You can create your own custom types to load into descriptions."
* "These custom types allow the user to configure the behaviour of a tool
   without tinkering directly with the tool description."
* "Custom types are described in separate YAML files and imported as needed."

[ips]: https://github.com/ebi-pf-team/interproscan
