# Scripts
This repository contains scripts for use with Mend Unified agent scanning within a CI/CD pipeline.

- [Reports Within a Pipeline](#reports-within-a-pipeline)
- [Pipeline SBOM Generation](#pipeline-sbom-generation)
- [Adding Red Shield Comment Links to GitHub Issues](#adding-red-shield-comment-links-to-github-issues)
- [Adding Red Shield Comments Links to GitHub Issues and Closing Green Shield Issues](#adding-red-shield-comments-links-to-github-issues-and-closing-green-shield-issues)
- [Display Vulnerabilities Affecting a Project](#display-vulnerabilities-affecting-a-project)
- [Display Policy Violations Following a Scan](#display-policy-violations-following-a-scan)
- [Cache the Latest Version of the Unified Agent](#cache-the-latest-version-of-the-unified-agent)
---
> **All scripts & snippets should call [check-project-state.sh](check-project-state.sh) before running to ensure that the scan has completed.**
<br>
---

## Reports Within a Pipeline

Any report can also be published as a part of the pipeline.  
Add the following snippet after calling the Unified Agent in any pipeline file to save reports from the scanned project to the `./whitesource` logs folder, then use your [pipeline publish](../CI-CD#Pipeline-Log-Publishing) feature to save the whitesource log folder as an artifact.  

<br>

**Prerequisites:**  

* `jq` and `awk` must be installed
* ENV variables must be set
  * WS_GENERATEPROJECTDETAILSJSON: true
  * WS_USERKEY
  * WS_WSS_URL

<br>

**Execution:**  

```
export WS_PROJECTTOKEN=$(jq -r '.projects | .[] | .projectToken' ./whitesource/scanProjectDetails.json)
export WS_URL=$(echo $WS_WSS_URL | awk -F "agent" '{print $1}')
  #RiskReport-Example
curl -o ./whitesource/riskreport.pdf -X POST "${WS_URL}/api/v1.3" -H "Content-Type: application/json"  -d '{"requestType":"getProjectRiskReport","userKey":"'${WS_USERKEY}'","projectToken":"'${WS_PROJECTTOKEN}'"}'
curl -o ./whitesource/inventoryreport.xlsx -X POST "${WS_URL}/api/v1.3" -H "Content-Type: application/json"  -d '{"requestType":"getProjectInventoryReport","userKey":"'${WS_USERKEY}'","projectToken":"'${WS_PROJECTTOKEN}'"}'
curl -o ./whitesource/duediligencereport.xlsx -X POST "${WS_URL}/api/v1.3" -H "Content-Type: application/json"  -d '{"requestType":"getProjectDueDiligenceReport","userKey":"'${WS_USERKEY}'","projectToken":"'${WS_PROJECTTOKEN}'"}'
```

<br>
<hr>

## Pipeline SBOM Generation

Add the following snippet after calling the Unified Agent in any pipeline to create an SPDX tag value output from the scanned project to the `./whitesource` logs folder, then use your [pipeline publish](../CI-CD#Pipeline-Log-Publishing) feature to save the whitesource log folder as an artifact.  

<br>

**Prerequisites:**  

* `jq`, `awk`, `python3` and `python3-pip` must be installed
* ENV variables must be set
  * WS_GENERATEPROJECTDETAILSJSON: true
  * WS_USERKEY
  * WS_APIKEY
  * WS_WSS_URL

<br>

**Execution:**  

```
export WS_PROJECTTOKEN=$(jq -r '.projects | .[] | .projectToken' ./whitesource/scanProjectDetails.json)
export WS_URL=$(echo $WS_WSS_URL | awk -F "agent" '{print $1}')
pip install ws-sbom-generator
ws_sbom_generator -u $WS_USERKEY -k $WS_APIKEY -s $WS_PROJECTTOKEN -a $WS_URL -t tv -o ./whitesource
```

More information & usage regarding the [WS SBOM generator](https://github.com/whitesource-ps/ws-sbom-spdx-report)


<br>
<hr>

## Adding Red Shield Comment Links to GitHub Issues

[ghissue-eua.sh](ghissue-eua.sh)  

Add the following lines after the Unified Agent command in a GitHub action to add comments to your GitHub issues that are created by the Mend GitHub integration. These comments will indicate if the vulnerability has a red shield and provide a link to the Mend UI for further examination.  

<br>

**Prerequisites:**  

* `jq` and `awk` must be installed
* ENV variables must be set
  * WS_GENERATEPROJECTDETAILSJSON: true
  * WS_USERKEY
  * WS_PRODUCTNAME
  * WS_PROJECTNAME
  * WS_WSS_URL

<br>

**Execution:**  

```
curl -LJO https://raw.githubusercontent.com/whitesource-ft/ws-examples/main/Scripts/ghissue-eua.sh 
chmod +x ./ghissue-eua.sh && ./ghissue-eua.sh
```

<br>
<hr>

## Ignoring Alerts Based on Prioritize

[ghissue-prioritize.sh](ghissue-prioritize.sh)  

Add the following lines after the Unified Agent command in a CI/CD pipeline to ignore vulnerabilities based on Mend Prioritize Green shields in a repository that is scanned via the Github Integration.

<br>

**Prerequisites:**  

* `jq` and `awk` must be installed
* ENV variables must be set
  * WS_GENERATEPROJECTDETAILSJSON: true
  * WS_USERKEY
  * WS_PRODUCTNAME
  * WS_PROJECTNAME
  * WS_APIKEY
  * WS_WSS_URL

<br>

**Execution:**  

```
curl -LJO https://raw.githubusercontent.com/whitesource-ft/ws-examples/main/Scripts/prioritize-ignore.sh 
chmod +x ./prioritize-ignore.sh && ./prioritize-ignore.sh
```

<br>
<hr>

## Display Vulnerabilities Affecting a Project

[list-project-alerts.sh](list-project-alerts.sh)  

This script can be added to the CI/CD pipeline (or executed independently) following the WhiteSource Unified Agent scan, to list vulnerabilities affecting the last scanned project(s).  

This script parses the `scanProjectDetails.json` file to get the `name` and `projectToken` of the project(s) created/updated during the last scan, and then uses WhiteSource's [getProjectAlertsByType](https://whitesource.atlassian.net/wiki/spaces/WD/pages/1651769359/Alerts+API#Project.2) API request to retrieve all the vulnerability alerts associated with that project. It then prints them to the standard output (`stdout`), sorted by severity and optionally color-coded.  

<br>

**Prerequisites:**  

* `jq` and `curl` must be installed
* ENV variables must be set
  * `WS_GENERATEPROJECTDETAILSJSON: true`
  * `WS_USERKEY` (admin assignment is required)
  * `WS_WSS_URL`
  * `WS_UPDATEINVENTORY: true` (defaults to true)

<br>

**Execution:**  

```
./list-project-alerts.sh
```
**Sample Output:**  
```
Alerts for project: vulnerable-node
Alerts: 10 High, 4 Medium, 2 Low

[H] CVE-2017-16138 - mime-1.3.4.tgz
[H] CVE-2015-8858 - uglify-js-2.3.0.tgz
[H] CVE-2017-1000228 - ejs-0.8.8.tgz
[H] CVE-2017-1000048 - qs-4.0.0.tgz
[H] CVE-2020-8203 - lodash-4.17.11.tgz
[H] CVE-2021-23337 - lodash-4.17.11.tgz
[H] CVE-2019-5413 - morgan-1.6.1.tgz
[H] CVE-2019-10744 - lodash-4.17.11.tgz
[H] CVE-2017-16119 - fresh-0.3.0.tgz
[H] CVE-2015-8857 - uglify-js-2.3.0.tgz
[M] CVE-2020-28500 - lodash-4.17.11.tgz
[M] CVE-2017-16137 - debug-2.2.0.tgz
[M] CVE-2019-14939 - mysql-2.12.0.tgz
[M] WS-2018-0080 - mysql-2.12.0.tgz
[L] WS-2018-0589 - nwmatcher-1.3.9.tgz
[L] WS-2017-0280 - mysql-2.12.0.tgz
```

See known limitations [here](list-project-alerts.sh).  

<br>
<hr>

## Display Policy Violations Following a Scan

[list-policy-violations.sh](list-policy-violations.sh)  

This script parses the `policyRejectionSummary.json` file, following a WhiteSource Unified Agent scan, and prints to the standard output (`stdout`) the policies that where violated, as well as the libraries that violated them.  

The `policyRejectionSummary.json` file is created automatically under the agent log directory (`./whitesource`) during a scan that's configured to check policies.  
Every policy check overwrites this file, so this list is always specific to the last scan (that had policy check enabled).  

<br>

**Prerequisites:**  

* `jq` must be installed
* ENV variables must be set
  * `WS_CHECKPOLICIES: true`

<br>

**Execution:**  

```
./list-policy-violations.sh [-p|--includePath]
```
**Sample Outputs:**  
```
$ ./list-policy-violations.sh

WhiteSource Policy Violations
=============================
Product: vulnerable-node
Project: master
Total Rejected Libraries: 9

Policy Name: Reject Vuln CVSS 9+
Policy Type: VULNERABILITY_SCORE
Rejected Libraries:
  morgan-1.6.1.tgz
  pg-5.1.0.tgz
  ejs-2.7.4.tgz
  lodash-4.17.11.tgz
  ejs-0.8.8.tgz

Policy Name: Review BSD2
Policy Type: LICENSE
Rejected Libraries:
  semver-4.3.2.tgz
  source-map-0.1.43.tgz
  qs-4.0.0.tgz
  uglify-js-2.3.0.tgz

```

```
$ ./list-policy-violations.sh --includePath

WhiteSource Policy Violations
=============================
Product: easybuggy
Project: master
Total Rejected Libraries: 6

Policy Name: Reject Vuln CVSS 9+
Policy Type: VULNERABILITY_SCORE
Rejected Libraries:
  log4j-1.2.13.jar  (/build/gl/easybuggy/target/easybuggy-1-SNAPSHOT/WEB-INF/lib/log4j-1.2.13.jar)
  commons-fileupload-1.3.1.jar  (/build/gl/easybuggy/target/easybuggy-1-SNAPSHOT/WEB-INF/lib/commons-fileupload-1.3.1.jar)
  derby-10.8.3.0.jar  (/home/gl/.m2/repository/org/apache/derby/derby/10.8.3.0/derby-10.8.3.0.jar)

Policy Name: Review LGPL
Policy Type: LICENSE
Rejected Libraries:
  xom-1.2.5.jar  (/build/gl/easybuggy/target/easybuggy-1-SNAPSHOT/WEB-INF/lib/xom-1.2.5.jar)
  bsh-core-2.0b4.jar  (/build/gl/easybuggy/target/easybuggy-1-SNAPSHOT/WEB-INF/lib/bsh-core-2.0b4.jar)
  javassist-3.12.1.GA.jar  (/build/gl/easybuggy/target/easybuggy-1-SNAPSHOT/WEB-INF/lib/javassist-3.12.1.GA.jar)

```

<br>
<hr>

## Cache the Latest Version of the Unified Agent

[cache-ua.sh](cache-ua.sh)  

This script allows caching of the [WhiteSource Unified Agent](https://whitesource.atlassian.net/wiki/spaces/WD/pages/1140852201/Getting+Started+with+the+Unified+Agent), so you can periodically check for updates and download the latest version only if needed, rather than redundantly downloading prior to every scan.  

The [cache-ua.sh](cache-ua.sh) script can be added to the CI/CD pipeline on a static/hosted build agent (prior to the Unified Agent scan task), or triggered independently, manually or by a scheduled task.  

<br>

**Prerequisites:**  

* `jq` and `curl` must be installed

<br>

**Execution:**  

```
curl -LJO https://raw.githubusercontent.com/whitesource-ft/ws-examples/main/Scripts/cache-ua.sh.sh 
chmod +x ./cache-ua.sh.sh && ./cache-ua.sh.sh
```

See additional example for implementation within a build pipeline under [CI-CD](../CI-CD/README.md#caching-the-unified-agent) (`*-cached-ua.yml`).  
