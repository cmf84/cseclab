# Container Security - Overview

The security of a container environment depends on a number of factors.
This is an overview of those factors and best practices to improve security:
1. Container Image
    - use up-to-date container images & container scanning to avoid common vulnurabilities and exploits (CVEs). This was already covered in Lab 1
    - sign container images and verify signatures
2. Image Registry
    - registry should be private to provide absolute control of images themselves and access control
    - continuous monitoring of images
    - host server must be secure
3. Container Runtime
    - implement app security settings best-practices (e.g. resource quotas)
    - monitor network protocol & payload
    - monitor Host OS
4. Orchestration Platform
    - use access control to limit number of users and their priviliges
    - monitor orchestration platform
    - monitor pod communication
5. Host OS
    - implement access control within OS
    - monitor OS for CVEs

## Open Source Tools
A number of great tools exist to follow the practices outlined above, such as:
* Scanning:
    - *DockerBench*: a script that tests container against  container deployment best practices and lets you know how container does against them
    - *OpenSCAP*: allows for scheduling of continuous scans around containers
* Monitoring:
    - *Promotheus*: allows monitoring of commincation between different Node endpoints (and definition of metrics)
* FireWall:
    - *Cilium*: analyze network communication and comm. between different application surfaces.

# Lab 2

In this lab we will spin up a Docker container and use DockerBench to identify and remove security problems in our deployment.

Steps (TBC)
1. run Docker container in default configuration
    - use Ubuntu Playground https://www.katacoda.com/courses/ubuntu/playground
2. Run Docker bench pull unsigned/signed image from public registry
3. go through observed problems and discuss/fix them (TBD)


TBC: signed/unsigned container demonstration
-  run unsigned container with/without docker_content_trust enabled


