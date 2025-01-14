![Logo](https://resources.mend.io/mend-sig/logo/mend-dark-logo-horizontal.png)  

[![License](https://img.shields.io/badge/License-Apache%202.0-yellowgreen.svg)](https://opensource.org/licenses/Apache-2.0)
[![GitHub release](https://img.shields.io/github/release/whitesource-ft/ws-template.svg)](https://github.com/whitesource-ft/ws-template/releases/latest)  
# Repository Integration Automation Scripts
When used, these scripts will download the latest repository integration run via docker compose:
- Remediate Server
- Controller
- Scanner

## Supported Operating Systems
- **Linux (Bash):**	CentOS, Debian, Ubuntu, RedHat

## Prerequisites
- Docker, Docker Compose, git, wget, SCM Repository instance up and running

## Options
`setup.sh` options: **ghe**, **gls**, **bb**

Options Defined:  
**ghe** - GitHub Enterprise
**gls** - GitLab (self-hosted)
**bb** - Bitbucket Server

## Execution
Execution instructions:  

- Clone the repository & give setup.sh permissions to run

```git clone https://github.com/mend-toolkit/mend-examples.git && cd mend-examples/Repo-Integration && chmod +x setup.sh```
- Add your activation key as an environment variable which will be copied to the .env file which is created by setup.sh

```export ws_key='your activation key between single quotes'```
- Run the setup.sh script for your appropriate source control management system as shown in options above

```./setup.sh <option>```
- Run docker compose depending on how it was installed

```docker-compose up``` 

- Run docker compose in detached mode depending on how it was installed.

```docker-compose up -d```
