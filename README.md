# Ansible node configuration

Ansible prepares Ubuntu 24.04 hosts with SSH hardening, UFW, sysctl, unattended security upgrades, and containerd; installs node-exporter as a systemd service; and renders Prometheus file-SD targets, alert rules, and Alertmanager configuration on the monitoring host. The playbook is idempotent.

## Setup and run

```bash
ansible-galaxy collection install -r requirements.yml
```

Edit `inventory/hosts.ini` with reachable Ubuntu host addresses, or use the AWS EC2 inventory after tagging node instances `Role=kubernetes-node`, the monitoring host `Role=monitoring`, and all instances `Environment=capstone`. For AWS inventory, ensure AWS credentials are available through the normal AWS SDK credential chain.

Before applying SSH hardening, verify key-based SSH and sudo access. Change the sample `192.0.2.x` documentation addresses and check the allowed ports against your network. Run:

```bash
ansible-inventory --graph
ansible-playbook -i inventory/hosts.ini playbooks/site.yml --syntax-check
ansible-playbook -i inventory/hosts.ini playbooks/site.yml --check --diff
ansible-playbook -i inventory/hosts.ini playbooks/site.yml
```

For dynamic AWS inventory, substitute `-i inventory/aws_ec2.yml`. The monitoring configuration roles expect Prometheus and Alertmanager service accounts and installations to exist separately. Include `/etc/prometheus/file_sd/node-targets.yml` and `/etc/prometheus/rules.d/alert-rules.yml` from the Prometheus main config, then validate/reload both monitoring services with your deployment process.

## Secrets

`alertmanager_slack_webhook_url` is only a placeholder. Store its real value using Ansible Vault, for example:

```bash
ansible-vault encrypt_string --name alertmanager_slack_webhook_url 'https://hooks.slack.com/services/REPLACE'
ansible-playbook -i inventory/hosts.ini playbooks/site.yml --ask-vault-pass
```

Do not commit a real webhook or plaintext Vault password. Restrict SSH and node-exporter firewall access to trusted networks before production use.
