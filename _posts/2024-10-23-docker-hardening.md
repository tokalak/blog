---
title: "Docker Image Hardening"
last_modified_at: 2024-10-23T11:20:02-05:00
categories:
  - Blog
  - DevOps
tags:
  - Docker 
  - Security
  - DevOps
toc: true
classes: wide
---

# Docker Image Hardening?
Usually all docker images are based on a root image. 
Given the `Dockerfile` below:

```yaml
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```
We're basing our docker image on base image `nginx:alphine`.  
It could be that some of the tools and packages contained in that pre-built image have security issues.  
To be sure that the image is safe to use from a security perspectiv, we'd to check all contained packages for Common Vulnerabilities and Exposures (CVEs).
Doing that by hand is a lot of work - almost not possible.

## Available Tools 
Luckily there are some tools, which are specialized on this topic.
- [trivy](https://github.com/aquasecurity/trivy)
- [anchore](https://anchore.com)
- [Snyk Container](https://snyk.io/product/container-vulnerability-management/)
- [Docker Scout](https://www.docker.com/products/docker-scout)

## trivy
trivy is the only tool from the list above, which is open-source.
It can scan: 
- Container Image
- Filesystem
- Git Repository (remote)
- Virtual Machine Image
- Kubernetes
- AWS

## Demo
The installation is very easy for my setup. Please check the project site for the installation guide, which is more appropriate for you.    
I'll use `brew` to install it. 
```bash
 brew install trivy
```

Scan the image:

```bash
 trivy image nginx:alpine
2024-10-23T11:20:48+03:00	INFO	[vuln] Vulnerability scanning is enabled
2024-10-23T11:20:48+03:00	INFO	[secret] Secret scanning is enabled
2024-10-23T11:20:48+03:00	INFO	[secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-10-23T11:20:48+03:00	INFO	[secret] Please see also https://aquasecurity.github.io/trivy/v0.56/docs/scanner/secret#recommendation for faster secret detection
2024-10-23T11:20:50+03:00	INFO	Detected OS	family="alpine" version="3.20.3"
2024-10-23T11:20:50+03:00	INFO	[alpine] Detecting vulnerabilities...	os_version="3.20" repository="3.20" pkg_num=66
2024-10-23T11:20:50+03:00	INFO	Number of language-specific files	num=0
2024-10-23T11:20:50+03:00	WARN	Using severities from other vendors for some vulnerabilities. Read https://aquasecurity.github.io/trivy/v0.56/docs/scanner/vulnerability#severity-selection for details.

nginx:alpine (alpine 3.20.3)

Total: 2 (UNKNOWN: 0, LOW: 2, MEDIUM: 0, HIGH: 0, CRITICAL: 0)
```
![trivy scan result](/blog/assets/images/trivy-scan-result.png)



