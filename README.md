# Onboarding Task 4 — Ansible Configuration

This repository configures a web server through its private EC2 address using Ansible roles for Keycloak, OAuth2 Proxy and Nginx. AWS infrastructure is created by `task-4-infra`, while AMIs are built by `task-4-packer`.

## Prerequisites

- Ansible controller EC2 provisioned and reachable via SSH.
- Web server at private IP `10.10.1.111` (currently in a public subnet but protected by Security Groups), reachable from controller via SSH; private key at `/home/ubuntu/.ssh/ansible_ssh_private_key` (mode `0600`).
- IAM instance profile on controller with `ssm:GetParameter` access to required SSM SecureString parameters; Python AWS SDK (`boto3`/`botocore`) available.
- `amazon.aws` and `community.docker` Ansible collections as declared in `requirements.yml`.
- The Web Server AMI must have Nginx, Docker, Docker Compose v2,
and OAuth2 Proxy 7.15.3 preinstalled. Ansible manages service configuration, Keycloak realm import, systemd services, and Nginx virtual hosts. It does not install these base packages.

## Run from Ansible controller

The EC2 user-data bootstrap currently clones the `onb-4-ansible` branch; after merge the infrastructure bootstrap must point to `main`.

```bash
cd /home/ubuntu/task-4-ansible
git branch --show-current
ansible-galaxy collection install -r requirements.yml
ansible -i inventory.ini web_server -m ansible.builtin.ping
ansible-playbook -i inventory.ini playbook.yml --syntax-check
ansible-playbook -i inventory.ini playbook.yml
```

## Runtime checks on web server

OAuth2 Proxy listens on `127.0.0.1:4180` for Keycloak OIDC and `127.0.0.1:4181` for direct GitHub OAuth. Keycloak listens locally on `127.0.0.1:8080`; public HTTPS is terminated by ALB.

In a browser, verify that authorized users can access `https://app.coinops-d.pp.ua`, users without `site-user` cannot, and the GitHub/GitLab and direct GitHub login flows work. Use separate fresh sessions.

## Secrets and roles

`playbook.yml` reads secret values from AWS SSM Parameter Store with `no_log: true`; do not commit credentials in `group_vars`, print them during tests, or expose rendered OAuth2/Keycloak config files. Runtime configs and realm import data are generated with Ansible templates.
