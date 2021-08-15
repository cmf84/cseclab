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
4. Host OS
    - implement access control within OS
    - monitor OS for CVEs
5. Orchestration Platform
    - use access control to limit number of users and their priviliges
    - monitor orchestration platform
    - monitor pod communication

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

In this lab we will use DockerBench to identify security problems in our container host. We will then look at the changes necessary to remove the observed problems in the host and docker daemon configuration.

Please note that the tests and output of Dockerbench depend on the containers that are running on the host. We will therefore launch a container, run Dockerbench again and observe the additional security issues found in this state.

## Open Ubuntu environment and run Docker Bench
Let's go to Ubuntu Playground https://www.katacoda.com/courses/ubuntu/playground and wait for a session to open. You should now be see Ubunut CLI for us to run our tests on.

Let's download and run Dockerbench in a separate container with the following command:

```
$ docker run -it --net host --pid host \
 --cap-add audit_control \
 -v /var/lib:/var/lib \
 -v /var/run/docker.sock:/var/run/docker.sock \
 -v /usr/lib/systemd:/usr/lib/systemd \
 -v /etc:/etc --label docker_bench_security \
 docker/docker-bench-security
 ```

 PS: alternatively we can download and run the script directly on the host using these two commands:
 ```
 $ git clone https://github.com/docker/docker-bench-security.git
 
 $ docker-bench-security/docker-bench-security.sh
```

The script runs through a number of tests and gives a `INFO`, `NOTE`, `PASS`, or `WARN` result for each one. The tests are grouped into sections, and the default Docker installation in our environment will show some warnings in sections 1,2,3,4:
1. Host configuration
2. Docker Daemon configuration
3. Docker Daemon configuration files
4. Container images and Build files

The overall system score is denoted in the bottom of the output. Don't worry if the score is low, we will work on that next.

## Analyze Docker Bench output and remove warnings
In order to improve the score we will go through the above sections and change the system configuration to adhere to the safety standards that Docker Bench tests for. Please see this [blog post](https://www.digitalocean.com/community/tutorials/how-to-audit-docker-host-security-with-docker-bench-for-security-on-ubuntu-16-04) for more background information on the tests.

## 1. Correcting host configuration
### Ensure a separate partition for containers has been created
To ensure proper isolation, it’s a good idea to keep Docker containers and all of /var/lib/docker on their own filesystem partition. Since this is difficult to achieve in our playground environment we will skip this here but it should absolutely be implemented in practise.

### Ensure auditing is configured for various Docker files
To enable auditing some of Docker's files, directories and sockets we first need to install auditd by running:

`$ apt-get install auditd`

Now let's configure the daemon by inserting the following lines into the bottom of `/etc/audit/audit.rules`:
```
-w /usr/bin/docker -p wa
-w /var/lib/docker -p wa
-w /etc/docker -p wa
-w /lib/systemd/system/docker.service -p wa
-w /lib/systemd/system/docker.socket -p wa
-w /etc/default/docker -p wa
-w /etc/docker/daemon.json -p wa
-w /usr/bin/docker-containerd -p wa
-w /usr/bin/docker-runc -p wa
```
Finally, let's restart auditd so the changes can take effect:

`$ systemctl restart auditd`

Possibly, even more files need to be added to the audit list. You can go ahead and amend the audit configuration file accordingly to get rid of the remaining warnings.


## 2. Correcting Docker Daemon Configuration
This section of the audit deals with the configuration of the Docker daemon. These warnings can all be addressed by editing the Docker `daemon.json`
configuration file. 

In your Ubuntu environment paste the following the lines into `/etc/docker/daemon.json`:

```
{
    "icc": false,
    "userns-remap": "default",
    "live-restore": true,
    "userland-proxy": false,
    "no-new-privileges": true
}
```
For more details and background on these parameters see the [blog post](https://www.digitalocean.com/community/tutorials/how-to-audit-docker-host-security-with-docker-bench-for-security-on-ubuntu-16-04) from above.

Additional note: Due to limitations of our Ubuntu playground environment we won't be able to  central logging with `"log-driver": "syslog"` and deactivate insecure registries (TBC). However, this should absolutely be implemented in real-world environments.

### Enable Content Trust
Finally, in section 4 a warning is issued: `Ensure Content Trust for Docker is Enabled`. Content trust is a system for signing and verifying Docker images before running them to verify the origin and integrity of images.

To enable content trust for the current shell session, execute:

`export DOCKER_CONTENT_TRUST=0`

In order to enable content trust system-wide for every session, add the following line to the `/etc/environment` file:

```
echo "DOCKER_CONTENT_TRUST=1" | sudo tee -a /etc/environment
```

## Docker Bench output while containers are running
Docker Bench takes active container into account during its scans. Therefore the output particularly in section 4 will change significantly when containers are running.

To verify this start some container image:

`$ docker run redis`

Now repeat the Docker Bench scan and observe the output. 

Solving the additional warning is left to the reader as an exercise.