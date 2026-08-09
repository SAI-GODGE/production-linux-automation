
# Production-Ready Linux Server Management & Automation

## Overview

A production-style Linux server management and automation project built on AWS EC2, RHEL 9, and Ansible.

The project demonstrates centralized Linux administration, configuration management, security hardening, web server deployment, system health monitoring, troubleshooting, configuration drift management, and automated remediation across a multi-node RHEL environment.

The environment consists of an Ansible control node managing three RHEL worker nodes through SSH and Ansible.

---

## Architecture

```text
                         AWS VPC
                      10.0.0.0/16
                           |
                    Production Subnet
                       10.0.1.0/24
                           |
          +----------------+----------------+
          |                |                |
          |                |                |
   ansible-control       web01            web02
   Ansible Control       Nginx             Nginx
          |
          |
        web03
        Apache

     Ansible Control
          |
       SSH/Ansible
          |
   +------+------+------+
   |             |      |
 web01         web02   web03
 Nginx         Nginx   Apache


