
# Production-Ready Linux Server Management & Automation

## Overview

This project is a hands-on Linux server management and automation environment built using AWS EC2, RHEL 9, and Ansible.

The main idea is to manage multiple Linux servers from a single Ansible control node instead of configuring each server manually. I automated common administration tasks such as system baseline configuration, user management, SSH hardening, firewall configuration, SELinux management, and web server deployment.

I also built health checks to monitor important system resources and services, generate fleet health reports, and identify issues such as high disk usage. To make the project more realistic, I simulated problems such as disk pressure, SELinux access denials, and Apache configuration errors, then investigated and recovered from them.

Overall, this project helped me practice Linux administration, Ansible automation, security, troubleshooting, monitoring, and Git-based configuration management in a production-style environment.

---

## Architecture

```text
                         AWS CLOUD
                             │
                       Production VPC
                        10.0.0.0/16
                             │
                       Public Subnet
                        10.0.1.0/24
                             │
          ┌──────────────────┴──────────────────┐
          │                                     │
   Ansible Control Node                   RHEL 9 Fleet
          │                              ┌──────┼──────┐
          │                              │      │      │
          │                             web01  web02  web03
          │                             Nginx  Nginx  Apache
          │
          └──────────── SSH / Ansible ────────────────┘

                 CENTRALIZED AUTOMATION
                         │
       ┌─────────────────┼──────────────────┐
       │                 │                  │
 Configuration      Security &         Health &
 Management         Hardening          Monitoring
       │                 │                  │
       ├─ Services       ├─ SELinux       ├─ Memory
       ├─ Web servers    ├─ SSH           ├─ Disk
       ├─ Firewall       ├─ Firewall      ├─ HTTP
       └─ Time sync      └─ Validation    └─ Fleet report

```
![Production Linux Server Management Architecture](docs/screenshots/architecture.png)

## Tech Stack

| Category | Technologies |
|---|---|
| Cloud Platform | AWS |
| Compute | Amazon EC2 |
| Operating System | Red Hat Enterprise Linux (RHEL) 9 |
| Automation & Configuration Management | Ansible |
| Remote Management | SSH |
| Web Servers | Nginx, Apache HTTP Server |
| Security | SELinux, firewalld, SSH hardening |
| Time Synchronization | chrony |
| Scripting & Administration | Bash / Linux Shell |
| Networking | VPC, Subnet, Internet Gateway, Route Table |
| Monitoring & Health Checks | Ansible-based health checks and fleet health reports |
| Version Control | Git, GitHub |

## Infrastructure Setup

The infrastructure for this project was created on AWS using a custom VPC, a public subnet, an Internet Gateway, and a dedicated route table. This network provides the environment in which the Ansible control node and RHEL servers are deployed and managed.

### AWS VPC

A custom VPC named `production-linux-vpc` was created in the AWS Mumbai (`ap-south-1`) region.

The VPC uses the IPv4 CIDR block `10.0.0.0/16`, providing a private address space for the project infrastructure.

**Configuration:**

- VPC Name: `production-linux-vpc`
- IPv4 CIDR: `10.0.0.0/16`
- Region: `ap-south-1` (Mumbai)
- Tenancy: Default
- DNS Resolution: Enabled

![AWS VPC Configuration](docs/screenshots/01-vpc-configuration.png)


### Public Subnet

A public subnet named `production-public-subnet` was created inside the production VPC.

The subnet uses the CIDR block `10.0.1.0/24` and is located in Availability Zone `ap-south-1a`.

**Configuration:**

- Subnet Name: `production-public-subnet`
- IPv4 CIDR: `10.0.1.0/24`
- Availability Zone: `ap-south-1a`
- VPC: `production-linux-vpc`

![Public Subnet Configuration](docs/screenshots/02-public-subnet.png)

### Internet Gateway

An Internet Gateway named `production-igw` was created and attached to the `production-linux-vpc`.

The Internet Gateway provides a path between the VPC and the internet for resources that have the appropriate routing and public network configuration.

**Configuration:**

- Internet Gateway: `production-igw`
- Status: Attached
- VPC: `production-linux-vpc`

![Internet Gateway](docs/screenshots/03-internet-gateway-attached.png)


### Route Table

A dedicated route table named `production-public-rt` was created for the public subnet.

The route table contains the default local VPC route and an internet route through the Internet Gateway.

**Routes:**

| Destination | Target |
|---|---|
| `10.0.0.0/16` | `local` |
| `0.0.0.0/0` | `production-igw` |

The `production-public-subnet` was explicitly associated with this route table, allowing resources in the subnet to use the configured routes.

![Public Route Table](docs/screenshots/04-public-route-table.png)

## Ansible Configuration

Ansible is used as the central automation and configuration management tool for the Linux server fleet. The `ansible-control` node manages the three RHEL servers — `web01`, `web02`, and `web03` — through SSH.

### Inventory

The Ansible inventory defines the managed Linux servers and allows them to be addressed as a group.

The project uses the following hosts:

- `web01`
- `web02`
- `web03`

The control node uses this inventory to execute Ansible commands and automation tasks across the server fleet.

### Connectivity Validation

Before performing configuration and administration tasks, connectivity between the Ansible control node and all managed servers was validated using the Ansible `ping` module.

```bash
ansible all -m ping
```
![Connectivity Validation](docs/screenshots/05-ansible-fleet-connectivity.png)

## Linux Baseline Configuration

A common Linux baseline was applied across the managed RHEL 9 servers using Ansible. The purpose of the baseline is to keep the servers consistent and ensure that important administration and security settings are configured in a repeatable way.

The baseline configuration includes:

- System timezone configuration
- Installation of required administration packages
- `chronyd` time synchronization
- `firewalld` configuration
- SELinux validation
- Standard system configuration

The baseline configuration is implemented in `playbooks/baseline.yml` and can be executed using:

```bash
ansible-playbook playbooks/baseline.yml
```

## Web Server Deployment

The project uses Ansible to deploy and manage web servers across the Linux fleet. Two different web servers were deployed to demonstrate how the same automation approach can manage different services on different hosts.

| Server | Host | Purpose |
|---|---|---|
| Nginx | `web01` | Web server |
| Nginx | `web02` | Web server |
| Apache HTTP Server | `web03` | Web server |

### Nginx

Nginx was deployed on `web01` and `web02` using an Ansible role.

The Nginx role manages:

- Nginx installation
- Nginx configuration
- Web application content
- Service state
- Configuration changes
- Service restart through Ansible handlers

The deployed web page contains the server hostname and indicates that the server is managed by Ansible.

#### web01

![Nginx Web Server - web01](docs/screenshots/16-web01-nginx.png)

#### web02

![Nginx Web Server - web02](docs/screenshots/17-web02-nginx.png)

The Nginx deployment was validated from the Ansible control node using an HTTP request:

```bash
ansible nginx -b -m shell -a 'curl -s http://localhost/'
```

### Apache

Apache HTTP Server was deployed on `web03` using a dedicated Ansible role.

The Apache role manages:

- Apache installation
- Apache configuration
- Web application content
- Service state
- Configuration validation
- Service restart through an Ansible handler

The Apache configuration was validated using:

```bash
ansible web03 -b -m command -a 'httpd -t'
```
The configuration returned:

```text
Syntax OK
```
The Apache service was then verified using:
```text
ansible web03 -b -m command -a 'systemctl is-active httpd'
```
The service returned:
```text
active
```
The deployed web application was tested using:
```text
ansible web03 -b -m shell -a 'curl -s http://localhost/'
```
The application successfully returned the Apache web page displaying the hostname and showing that the server is managed by Ansible.

#### web03

![Apache Web Server - web03](docs/screenshots/18-web03-apache.png)

## Security Hardening

Security hardening was applied across the managed RHEL 9 servers using Ansible. The project focuses on three important areas of Linux server security: SELinux, SSH, and the host-based firewall.

### SELinux

SELinux was kept enabled in **Enforcing** mode across the managed servers.

The status was verified using:

```bash
ansible all -b -m command -a 'getenforce'
```

![SELinux](docs/screenshots/06-selinux-enforcing.png)

The managed servers returned:
```text
Enforcing
```
The project also configured the correct SELinux context for web content under /srv/webapp.

The web content uses the SELinux type:

```text
httpd_sys_content_t  
```

This allows the web servers to access the application content while SELinux continues to enforce access-control policies.

### SSH Hardening

SSH was hardened using Ansible to reduce unnecessary remote-access risks.

The configuration was validated for settings including:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 4
ClientAliveInterval 300
ClientAliveCountMax 2
```

The configuration was checked using:

```text
ansible web -b -m shell -a 'sshd -T | grep -E "^(permitrootlogin|passwordauthentication|pubkeyauthentication|maxauthtries|clientaliveinterval|clientalivecountmax)"'
```

![SSH Hardening](docs/screenshots/07-ssh-hardening-validation.png)

This configuration uses SSH key-based authentication, prevents direct root login, disables password-based SSH authentication, limits authentication attempts, and configures SSH session keepalive settings.

### Firewall

firewalld was configured and validated on the managed RHEL servers.

The firewall configuration was managed through Ansible and verified to be active.

The project allows the services required for administration and web-server operation, including SSH and HTTP.

```text
ansible web -b -m command -a 'firewall-cmd --list-services'
```

![Firewall](docs/screenshots/08-firewall-validation.png)

The firewall status was also checked as part of the automated health-check process.

## Automated Health Monitoring

An automated health-check system was implemented using Ansible to monitor the overall condition of the managed RHEL servers.

The health-check role collects and evaluates important system information including:

- Memory utilization
- Root filesystem utilization
- SELinux status
- Firewall status
- SSH service status
- Chronyd service status
- Web service status
- HTTP port 80 availability
- Overall server health

The health checks are executed using:

```bash
ansible-playbook playbooks/health_check.yml
```
The results are generated for each managed server and then combined into a fleet-level health report.

### Fleet Health Report

A consolidated report is generated on the Ansible control node:
```text
reports/production-health-report.txt
```
The report provides a quick overview of the entire Linux fleet, including resource usage, security status, services, HTTP availability, and overall health.

![Fleet Health Report](docs/screenshots/09-fleet-health-report.png)

The service and network checks also report the state of SSH, chronyd, the web service, and HTTP port 80.

### Disk Monitoring

The health-check system monitors the usage of the root filesystem and classifies the result based on configured thresholds.

A normal server state was first verified using:
```text
ansible web01 -b -m shell -a 'df -h /'
```

![Disk Before Creating Pressure](docs/screenshots/12-disk-pressure-recovery.png)


To test the monitoring logic, controlled disk pressure was simulated on web01 by creating a temporary 12 GB file:
```text
ansible web01 -b -m shell -a 'fallocate -l 12G /tmp/disk-pressure-test.img'
```

![Disk Pressure Simulation](docs/screenshots/11-disk-pressure-simulation.png)


The root filesystem usage increased from approximately 13% to approximately 77%.

The health-check playbook was then executed again:
```text
ansible-playbook playbooks/health_check.yml
```

![Disk Pressure Warning](docs/screenshots/10-fleet-health-warning.png)
The generated report detected the increased disk usage and marked web01 as:
```text
Disk Status    : WARNING
Overall Status : WARNING
```
The disk-pressure test was later cleaned up and the filesystem returned to its normal usage level.

This demonstrates a simple operational workflow for detecting abnormal disk utilization before it becomes a critical storage problem.

## Failure Simulation & Recovery

After implementing the automated health-monitoring system, a controlled failure scenario was introduced to test how the monitoring system behaves when a server experiences abnormal disk utilization.

This test was performed on `web01` and followed a simple operational workflow:

```text
Normal State
     ↓
Introduce Controlled Failure
     ↓
Detect the Problem
     ↓
Investigate
     ↓
Recover the Server
     ↓
Validate Healthy State
```
### Disk Pressure Simulation

A temporary disk-pressure condition was intentionally created on web01 using a 12 GB test file.
```text
ansible web01 -b -m shell -a 'fallocate -l 12G /tmp/disk-pressure-test.img'
```
The test increased the root filesystem utilization and created a realistic warning condition for the health-monitoring system.

![Disk Pressure Simulation](docs/screenshots/11-disk-pressure-simulation.png)

### Warning Detection

The health-check playbook was executed after introducing the disk-pressure condition:
```text
ansible-playbook playbooks/health_check.yml
```
The previously implemented health-monitoring system detected the increased disk utilization and changed the status of web01 from HEALTHY to WARNING.

![Disk Pressure Warning](docs/screenshots/10-fleet-health-warning.png)

This confirmed that the monitoring logic was able to identify the simulated resource-pressure condition.

### Recovery Validation

After the test was completed, the temporary disk-pressure file was removed from web01.

The filesystem was checked again and the health-check playbook was executed to verify that the server had returned to its normal state.

The final health report showed that web01 had returned to a healthy disk status and the overall server status was restored to HEALTHY.

![Fleet Health Report](docs/screenshots/09-fleet-health-report.png)

This test demonstrates the complete incident lifecycle:
```text
Disk Pressure
     ↓
Health Check Detects WARNING
     ↓
Problem Investigated
     ↓
Temporary Test Data Removed
     ↓
Health Check Re-run
     ↓
Server Returns to HEALTHY
```

