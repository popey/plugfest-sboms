# Anchore Plugfest SBOMs

SBOMs for submission to the [SBOM Harmonization Plugfest 2024](https://resources.sei.cmu.edu/news-events/events/sbom/participate.cfm). These were built using [these scripts](https://github.com/popey/plugfest-scripts) using [Syft](https://github.com/anchore/syft) to output in various formats.

The plugfest organisers requested 'a couple' of SBOMs. We figured this would be a good opportunity to exercise all of the output format options in Syft, so we generated every supported format for every repo.

SBOMs are generated in Syft-versioned folders, so this could be run multiple times with different releases of Syft for comparison. For the final submission, I used [Syft](https://github.com/anchore/syft/) [v1.18.0](https://github.com/anchore/syft/releases/v1.18.0).

## Introduction

* SBOMs were generated using Syft 1.18.0 
* All defaults were used in the execution of Syft, other than the `--enrich` option, detailed below under "Enrichment"
* Repos refernced were pulled from the Plugfest [partitipate](https://resources.sei.cmu.edu/news-events/events/sbom/participate.cfm) page.

## Syft Methodology

Syft is a fully-offline, open-open source SBOM generator. It requires no login or API key to operate. 

Syft does not analyze:

- Source code for licenses
- Search for other SBOMs within the search scope
- Binary analysis

## Enrichment

Syft has an `--enrich` option (which is off by default) that can leverage some online resources to embellish SBOMs.

`--enrich stringArray     enable package data enrichment from local and online sources (options: all, golang, java, javascript)`

Syft was run both with and without the `--enrich` option, but only some projects will have seen any gains, as currently a limited set of online resources are queried with this feature.

For example:

* Un-enriched SBOM: `./source/syft/1.18.0/httpie/cli/spdx-json.json`
* Enriched SBOM: `./source/syft/1.18.0/httpie/cli/spdx-json_enriched.json`
* SBOM validation: `./source/syft/1.18.0/httpie/cli/spdx-json.json_pyspdxtools.txt`

This results in potentially greater detail in the generated SBOM.

For example, (at the time of writing) compare the line count of the un-enriched vs enriched versions of the CycloneDX SBOM for the container build of `dependency-track`

```bash
$ wc -l ./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json.json \ 
        ./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json_enriched.json 
  5494 ./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json.json
  6117 ./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json_enriched.json
```

Below is a snippet, showing the side-by-side `diff` of an un-enriched vs enriched CycloneDX SBOM.

(Spaces truncated for readabiliy.)

```bash
$ diff -y ./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json.json \
          ./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json_enriched.json \
          | grep '>' | head -n 15 | awk '{$1=$1};1'
> {
> "license": {
> "name": "Apache 2",
> "url": "http://www.apache.org/licenses/LICENSE-2.
> }
> }
> ],
> "cpe": "cpe:2.3:a:org.sonatype.oss:JUnitParams:1.1.1:*:
> "name": "syft:cpe23",
> "value": "cpe:2.3:a:JUnitParams:JUnitParams:1.1.1:*
> },
> {
> "name": "syft:cpe23",
> "value": "cpe:2.3:a:sonatype:JUnitParams:1.1.1:*:*:
> },
```

## SBOM formats

Syft supports numerous output formats. The SBOM generation scripts attempt to output in every supported format.

* cyclonedx-json
* cyclonedx-xml
* github-json
* spdx-json
* spdx-tag-value
* syft-json
* syft-table
* syft-text

## Environment

The SBOMs were created on an x86_64 Linux host running Debian 12 (bookworm).

All Syft JSON output passed through `jq`, and xml through `xmllint --format` before writing to files, to make the files human readable, rather than being compressed on one line. 

*No other external transformation was performed.*

We also generated binary SBOMs (and thus building the code in the repos), which required the installation and use of these tools.

* docker
* docker-compose
* go
* gradle, openjdk-8-jdk (for minecolonies, docs recommend 8, I used openjdk-17-jdk on Debian)
* cmake (for opencv)
* cargo, rustup, rust => 1.64.0 (for hexyl)
* uv (for the venv/pip install of httpie)

**Note:** We decided to build using whatever docker container configuration was available in the repo. If none was present, we used best-endeavour to build each project as a user might. This leverages Syft's common usage pattern of scanning docker images, or directories which contain binary assets.

The following table summarises how each package is built, if at all:

| Package | Build tool |
| ------- | ---------- |
| DependencyTrack/dependency-track | docker |
| gin-gonic/gin | Not built as it's a library |
| httpie/cli | pip install |
| jqlang/jq | docker |
| ldteam/minecolonies | gradle |
| opencv | cmake |
| PHPMailer/PHPMailer | Not built as it's php |
| sharkdp/hexyl | cargo |
| snyk-labs/nodejs-goof | docker |


### Validation

We used two SBOM validation tools to check some of the SBOM formats. 

* [CycloneDX/sbom-utility](https://github.com/CycloneDX/sbom-utility) v0.17.1-pre to validate both SPDX and CycloneDX formatted SBOMs. 
* [spdx/pyspdxtools](https://github.com/spdx/tools-python) to validate SPDX formatted SBOMs.

**Note 1:** We were not able to validate the XML files, as there were no validators provided for those SBOM formats.

**Note 2:** We also tried the NTIA validation site, which complains about data that isn't present, or is unknown in the SBOM.

**Note 3:** There is some disparity between the results of the two tools. For example the `spdx-json_enriched.json` file gets a clean bill of health from `sbom_utility` but fails `pyspdxtools` validation.

Validation results (if any) are placed side-by-side with the SBOM file in a text file called `(sbom_file)_pyspdxtools.txt`, and `(sbom_file)_sbom_utility.txt`.

## Table of SBOMS 

Generated on 2024-12-13 at 13:05 GMT.

### Source SBOMs

Created purely from the cloned repos.

| Type | Project | Tool | Ver | SBOM | Validator | Result |
| - | - | - | - | - | - | - |
| source | dependency-track | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/DependencyTrack/dependency-track/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/DependencyTrack/dependency-track/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | dependency-track | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/DependencyTrack/dependency-track/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/DependencyTrack/dependency-track/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | dependency-track | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/DependencyTrack/dependency-track/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/DependencyTrack/dependency-track/spdx-json.json_pyspdxtools.txt) | Pass |
| source | dependency-track | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/DependencyTrack/dependency-track/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/DependencyTrack/dependency-track/spdx-json.json_sbom_utility.txt) | Pass |
| source | dependency-track | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/DependencyTrack/dependency-track/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/DependencyTrack/dependency-track/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | dependency-track | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/DependencyTrack/dependency-track/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/DependencyTrack/dependency-track/cyclonedx-json.json_sbom_utility.txt) | Pass |
| source | gin | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/gin-gonic/gin/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/gin-gonic/gin/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | gin | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/gin-gonic/gin/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/gin-gonic/gin/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | gin | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/gin-gonic/gin/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/gin-gonic/gin/spdx-json.json_pyspdxtools.txt) | Pass |
| source | gin | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/gin-gonic/gin/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/gin-gonic/gin/spdx-json.json_sbom_utility.txt) | Pass |
| source | gin | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/gin-gonic/gin/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/gin-gonic/gin/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | gin | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/gin-gonic/gin/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/gin-gonic/gin/cyclonedx-json.json_sbom_utility.txt) | Pass |
| source | cli | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/httpie/cli/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/httpie/cli/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | cli | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/httpie/cli/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/httpie/cli/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | cli | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/httpie/cli/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/httpie/cli/spdx-json.json_pyspdxtools.txt) | Pass |
| source | cli | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/httpie/cli/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/httpie/cli/spdx-json.json_sbom_utility.txt) | Pass |
| source | cli | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/httpie/cli/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/httpie/cli/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | cli | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/httpie/cli/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/httpie/cli/cyclonedx-json.json_sbom_utility.txt) | Pass |
| source | jq | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/jqlang/jq/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/jqlang/jq/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | jq | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/jqlang/jq/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/jqlang/jq/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | jq | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/jqlang/jq/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/jqlang/jq/spdx-json.json_pyspdxtools.txt) | Pass |
| source | jq | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/jqlang/jq/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/jqlang/jq/spdx-json.json_sbom_utility.txt) | Pass |
| source | jq | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/jqlang/jq/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/jqlang/jq/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | jq | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/jqlang/jq/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/jqlang/jq/cyclonedx-json.json_sbom_utility.txt) | Pass |
| source | minecolonies | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/ldtteam/minecolonies/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/ldtteam/minecolonies/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | minecolonies | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/ldtteam/minecolonies/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/ldtteam/minecolonies/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | minecolonies | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/ldtteam/minecolonies/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/ldtteam/minecolonies/spdx-json.json_pyspdxtools.txt) | Pass |
| source | minecolonies | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/ldtteam/minecolonies/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/ldtteam/minecolonies/spdx-json.json_sbom_utility.txt) | Pass |
| source | minecolonies | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/ldtteam/minecolonies/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/ldtteam/minecolonies/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | minecolonies | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/ldtteam/minecolonies/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/ldtteam/minecolonies/cyclonedx-json.json_sbom_utility.txt) | Pass |
| source | opencv | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/opencv/opencv/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/opencv/opencv/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | opencv | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/opencv/opencv/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/opencv/opencv/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | opencv | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/opencv/opencv/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/opencv/opencv/spdx-json.json_pyspdxtools.txt) | Pass |
| source | opencv | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/opencv/opencv/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/opencv/opencv/spdx-json.json_sbom_utility.txt) | Pass |
| source | opencv | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/opencv/opencv/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/opencv/opencv/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | opencv | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/opencv/opencv/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/opencv/opencv/cyclonedx-json.json_sbom_utility.txt) | Pass |
| source | PHPMailer | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/PHPMailer/PHPMailer/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/PHPMailer/PHPMailer/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | PHPMailer | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/PHPMailer/PHPMailer/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/PHPMailer/PHPMailer/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | PHPMailer | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/PHPMailer/PHPMailer/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/PHPMailer/PHPMailer/spdx-json.json_pyspdxtools.txt) | Pass |
| source | PHPMailer | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/PHPMailer/PHPMailer/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/PHPMailer/PHPMailer/spdx-json.json_sbom_utility.txt) | Pass |
| source | PHPMailer | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/PHPMailer/PHPMailer/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/PHPMailer/PHPMailer/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | PHPMailer | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/PHPMailer/PHPMailer/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/PHPMailer/PHPMailer/cyclonedx-json.json_sbom_utility.txt) | Pass |
| source | hexyl | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/sharkdp/hexyl/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/sharkdp/hexyl/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | hexyl | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/sharkdp/hexyl/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/sharkdp/hexyl/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | hexyl | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/sharkdp/hexyl/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/sharkdp/hexyl/spdx-json.json_pyspdxtools.txt) | Pass |
| source | hexyl | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/sharkdp/hexyl/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/sharkdp/hexyl/spdx-json.json_sbom_utility.txt) | Pass |
| source | hexyl | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/sharkdp/hexyl/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/sharkdp/hexyl/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | hexyl | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/sharkdp/hexyl/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/sharkdp/hexyl/cyclonedx-json.json_sbom_utility.txt) | Pass |
| source | nodejs-goof | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/snyk-labs/nodejs-goof/spdx-json_enriched.json) | [pyspdxtools](./source/syft/1.18.0/snyk-labs/nodejs-goof/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| source | nodejs-goof | syft | 1.18.0 | [spdx-json_enriched.json](./source/syft/1.18.0/snyk-labs/nodejs-goof/spdx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/snyk-labs/nodejs-goof/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| source | nodejs-goof | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/snyk-labs/nodejs-goof/spdx-json.json) | [pyspdxtools](./source/syft/1.18.0/snyk-labs/nodejs-goof/spdx-json.json_pyspdxtools.txt) | Pass |
| source | nodejs-goof | syft | 1.18.0 | [spdx-json.json](./source/syft/1.18.0/snyk-labs/nodejs-goof/spdx-json.json) | [sbom-utility](./source/syft/1.18.0/snyk-labs/nodejs-goof/spdx-json.json_sbom_utility.txt) | Pass |
| source | nodejs-goof | syft | 1.18.0 | [cyclonedx-json_enriched.json](./source/syft/1.18.0/snyk-labs/nodejs-goof/cyclonedx-json_enriched.json) | [sbom-utility](./source/syft/1.18.0/snyk-labs/nodejs-goof/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| source | nodejs-goof | syft | 1.18.0 | [cyclonedx-json.json](./source/syft/1.18.0/snyk-labs/nodejs-goof/cyclonedx-json.json) | [sbom-utility](./source/syft/1.18.0/snyk-labs/nodejs-goof/cyclonedx-json.json_sbom_utility.txt) | Pass |

### Binary SBOMs

Created from binaries or containers built from the cloned repos.

| Type | Project | Tool | Ver | SBOM | Validator | Result |
| - | - | - | - | - | - | - |
| binary | dependencytrackapiserver | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/spdx-json_enriched.json) | [pyspdxtools](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| binary | dependencytrackapiserver | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/spdx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | dependencytrackapiserver | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/spdx-json.json) | [pyspdxtools](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/spdx-json.json_pyspdxtools.txt) | Pass |
| binary | dependencytrackapiserver | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/spdx-json.json) | [sbom-utility](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/spdx-json.json_sbom_utility.txt) | Pass |
| binary | dependencytrackapiserver | syft | 1.18.0 | [cyclonedx-json_enriched.json](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | dependencytrackapiserver | syft | 1.18.0 | [cyclonedx-json.json](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json.json) | [sbom-utility](./binary/syft/1.18.0/dependency-track/dependencytrackapiserver/cyclonedx-json.json_sbom_utility.txt) | Pass |
| binary | dependencytrackfrontend | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/spdx-json_enriched.json) | [pyspdxtools](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| binary | dependencytrackfrontend | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/spdx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | dependencytrackfrontend | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/spdx-json.json) | [pyspdxtools](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/spdx-json.json_pyspdxtools.txt) | Pass |
| binary | dependencytrackfrontend | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/spdx-json.json) | [sbom-utility](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/spdx-json.json_sbom_utility.txt) | Pass |
| binary | dependencytrackfrontend | syft | 1.18.0 | [cyclonedx-json_enriched.json](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/cyclonedx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | dependencytrackfrontend | syft | 1.18.0 | [cyclonedx-json.json](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/cyclonedx-json.json) | [sbom-utility](./binary/syft/1.18.0/dependency-track/dependencytrackfrontend/cyclonedx-json.json_sbom_utility.txt) | Pass |
| binary | hexyl | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/hexyl/target_release/spdx-json_enriched.json) | [pyspdxtools](./binary/syft/1.18.0/hexyl/target_release/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| binary | hexyl | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/hexyl/target_release/spdx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/hexyl/target_release/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | hexyl | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/hexyl/target_release/spdx-json.json) | [pyspdxtools](./binary/syft/1.18.0/hexyl/target_release/spdx-json.json_pyspdxtools.txt) | Pass |
| binary | hexyl | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/hexyl/target_release/spdx-json.json) | [sbom-utility](./binary/syft/1.18.0/hexyl/target_release/spdx-json.json_sbom_utility.txt) | Pass |
| binary | hexyl | syft | 1.18.0 | [cyclonedx-json_enriched.json](./binary/syft/1.18.0/hexyl/target_release/cyclonedx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/hexyl/target_release/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | hexyl | syft | 1.18.0 | [cyclonedx-json.json](./binary/syft/1.18.0/hexyl/target_release/cyclonedx-json.json) | [sbom-utility](./binary/syft/1.18.0/hexyl/target_release/cyclonedx-json.json_sbom_utility.txt) | Pass |
| binary | jq | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/jq/plug-jq/spdx-json_enriched.json) | [pyspdxtools](./binary/syft/1.18.0/jq/plug-jq/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| binary | jq | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/jq/plug-jq/spdx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/jq/plug-jq/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | jq | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/jq/plug-jq/spdx-json.json) | [pyspdxtools](./binary/syft/1.18.0/jq/plug-jq/spdx-json.json_pyspdxtools.txt) | Pass |
| binary | jq | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/jq/plug-jq/spdx-json.json) | [sbom-utility](./binary/syft/1.18.0/jq/plug-jq/spdx-json.json_sbom_utility.txt) | Pass |
| binary | plug-jq | syft | 1.18.0 | [cyclonedx-json_enriched.json](./binary/syft/1.18.0/jq/plug-jq/cyclonedx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/jq/plug-jq/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | jq | syft | 1.18.0 | [cyclonedx-json.json](./binary/syft/1.18.0/jq/plug-jq/cyclonedx-json.json) | [sbom-utility](./binary/syft/1.18.0/jq/plug-jq/cyclonedx-json.json_sbom_utility.txt) | Pass |
| binary | minecolonies | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/minecolonies/build_libs/spdx-json_enriched.json) | [pyspdxtools](./binary/syft/1.18.0/minecolonies/build_libs/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| binary | minecolonies | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/minecolonies/build_libs/spdx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/minecolonies/build_libs/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | minecolonies | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/minecolonies/build_libs/spdx-json.json) | [pyspdxtools](./binary/syft/1.18.0/minecolonies/build_libs/spdx-json.json_pyspdxtools.txt) | Pass |
| binary | minecolonies | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/minecolonies/build_libs/spdx-json.json) | [sbom-utility](./binary/syft/1.18.0/minecolonies/build_libs/spdx-json.json_sbom_utility.txt) | Pass |
| binary | minecolonies | syft | 1.18.0 | [cyclonedx-json_enriched.json](./binary/syft/1.18.0/minecolonies/build_libs/cyclonedx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/minecolonies/build_libs/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | minecolonies | syft | 1.18.0 | [cyclonedx-json.json](./binary/syft/1.18.0/minecolonies/build_libs/cyclonedx-json.json) | [sbom-utility](./binary/syft/1.18.0/minecolonies/build_libs/cyclonedx-json.json_sbom_utility.txt) | Pass |
| binary | nodejs-goof | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/spdx-json_enriched.json) | [pyspdxtools](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/spdx-json_enriched.json_pyspdxtools.txt) | Fail |
| binary | nodejs-goof | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/spdx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | nodejs-goof | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/spdx-json.json) | [pyspdxtools](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/spdx-json.json_pyspdxtools.txt) | Fail |
| binary | nodejs-goof | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/spdx-json.json) | [sbom-utility](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/spdx-json.json_sbom_utility.txt) | Pass |
| binary | nodejs-goof | syft | 1.18.0 | [cyclonedx-json_enriched.json](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/cyclonedx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | nodejs-goof | syft | 1.18.0 | [cyclonedx-json.json](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/cyclonedx-json.json) | [sbom-utility](./binary/syft/1.18.0/nodejs-goof/plug-nodejs-goof/cyclonedx-json.json_sbom_utility.txt) | Pass |
| binary | opencv | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/spdx-json_enriched.json) | [pyspdxtools](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/spdx-json_enriched.json_pyspdxtools.txt) | Pass |
| binary | opencv | syft | 1.18.0 | [spdx-json_enriched.json](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/spdx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/spdx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | opencv | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/spdx-json.json) | [pyspdxtools](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/spdx-json.json_pyspdxtools.txt) | Pass |
| binary | opencv | syft | 1.18.0 | [spdx-json.json](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/spdx-json.json) | [sbom-utility](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/spdx-json.json_sbom_utility.txt) | Pass |
| binary | opencv | syft | 1.18.0 | [cyclonedx-json_enriched.json](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/cyclonedx-json_enriched.json) | [sbom-utility](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/cyclonedx-json_enriched.json_sbom_utility.txt) | Pass |
| binary | opencv | syft | 1.18.0 | [cyclonedx-json.json](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/cyclonedx-json.json) | [sbom-utility](./binary/syft/1.18.0/opencv/usr_local_opt_opencv/cyclonedx-json.json_sbom_utility.txt) | Pass |

