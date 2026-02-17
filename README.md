# Automation Challenge – NGINX on Azure (Terraform + Docker + TLS)

This repository contains an end-to-end solution for the automation challenge:
- Infrastructure provisioned with Terraform on Azure
- NGINX deployed as a Docker container
- HTTPS enabled with a self-signed TLS certificate for the FQDN
- Static content can be updated without rebuilding the container image
- CI pipeline (GitHub Actions) validates Terraform formatting and configuration

## Architecture (high level)
- Azure Resource Group, VNet/Subnet, NSG, Public IP, NIC, Linux VM (Ubuntu)
- Docker installed via cloud-init
- NGINX container with bind mounts:
  - `/opt/nginx/html` -> `/usr/share/nginx/html` (static content)
  - `/opt/nginx/certs` -> `/etc/nginx/certs` (TLS cert/key)
  - `/opt/nginx/conf/default.conf` -> `/etc/nginx/conf.d/default.conf` (NGINX config)

## Task 1 – Deploy an NGINX webserver
- NGINX is reachable from the local machine and serves a "Hello CGI!" page.

## Task 2 – Handle HTTPS & TLS
- HTTPS is enabled on port 443.
- TLS certificate is self-signed and generated on the VM for the FQDN.

## Bonus I – FQDN automation-challenge.cgi.com
Because public DNS is not available in the sandbox, the FQDN is mapped locally via the Windows hosts file:

`C:\Windows\System32\drivers\etc\hosts`

20.101.75.60 automation-challenge.cgi.com


Then open:
- http://automation-challenge.cgi.com
- https://automation-challenge.cgi.com

> Browser shows "Not secure" because the certificate is self-signed (expected).

## Task 3 – Containerization + update static content without image rebuild
NGINX runs as a container. Static content is mounted from the host into the container.
To update content:
- Edit `/opt/nginx/html/index.html` on the VM
- Refresh the browser
No `docker build` required.

## Bonus II – CI Pipeline
GitHub Actions workflow runs on push/PR:
- `terraform fmt -check`
- `terraform init -backend=false`
- `terraform validate`

## Notes about permissions
The Azure sandbox user has Contributor permissions and cannot create Service Principals / OIDC credentials.
Therefore, CI is implemented as validation gates. `terraform apply` is executed locally.
