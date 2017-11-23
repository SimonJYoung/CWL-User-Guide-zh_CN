## 指定软件需求

### 总览

问题：

* "How do I specify requirements/dependencies for a job?"
* "What level of detail should I provide for a software requirement?"

目标：

* "Learn how to write software requirement descriptions."
* "Learn how to use SciCrunch to retrieve a unique identifier for a tool/version
that is required."

-----

Often tool descriptions will be written for a specific version of a software. To
make it easier for others to make use of your descriptions, you can include a
`SoftwareRequirement` field in the `hints` section.
This may also help to avoid confusion about which version of a tool the
description was written for.

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


In this example, the software requirement being described is InterProScan
version 5.21-60.

~~~YAML
hints:
  SoftwareRequirement:
    packages:
      interproscan:
        specs: [ "https://identifiers.org/rrid/RRID:SCR_005829" ]
        version: [ "5.21-60" ]
~~~


Depending on your CWL runner, these hints may be used to check
that required software is installed and available before the job is run. To enable
these checks with the reference implementation, use the [dependency resolvers configuration][dependencies].

As well as a version number, a unique resource identifier (URI) for the tool is
given in the form of an [RRID][rrid]. Resources with RRIDs can be looked up in the
[SciCrunch][scicrunch] registry, which provides a portal for finding, tracking,
and referring to scientific resources consistently. If you want to specify a
tool as a `SoftwareRequirement`, search for the tool on SciCrunch and use the
RRID that it has been assigned in the registry. (Follow [this tutorial][scicrunch-add-tool]
if you want to add a tool to SciCrunch.) You can use this RRID to refer
to the tool (via [identifiers.org][identifiers]) in the `specs` field of your
requirement description. Other good choices, in order of preference, are to
include the DOI for the main tool citation and the URL to the tool.

-----

要点：

* "Software requirements should be specified under `hints:SoftwareRequirement`."

[rrid]: https://scicrunch.org/resources/about/resource
[scicrunch]: https://scicrunch.org/
[dependencies]: https://github.com/common-workflow-language/cwltool#leveraging-softwarerequirements-beta
[identifiers]: https://identifiers.org/
[scicrunch-add-tool]: https://scicrunch.org/page/tutorials/336
