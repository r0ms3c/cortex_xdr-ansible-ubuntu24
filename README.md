# Cortex XDR Agent — Ubuntu 24.04 (Ansible — Simple Ubuntu-only)

Ansible project to deploy the **Palo Alto Networks Cortex XDR Agent** on **Ubuntu 24.04** using SSH key authentication,
**no Jinja templates**, and the **same configuration across all servers**.

## What this does
- Creates `/etc/panw/cortex.conf` with a fixed configuration for all hosts.
- Installs the agent from a local `.deb` using `apt`.
- Ensures and starts the `traps_pmd.service` systemd unit.

## Why this approach
- Per official documentation, Linux installation supports local **.deb** on Ubuntu/Debian and reads options from `/etc/panw/cortex.conf`.
- Keep the project minimal and easy to audit.

## Prerequisites
- **Ansible** control node with network access to targets over SSH (key-based auth).
- Target hosts: **Ubuntu Server 24.04**.
- Outbound HTTPS (TCP/443) to your Cortex XDR cloud.
- The **Ubuntu .deb** installer package downloaded from the Cortex XDR console (place it at `roles/cortex_xdr/files/cortex-agent.deb`).

> Notes
> - If you prefer **User Space Mode** (no kernel module), uncomment `--no-km` in `tasks/main.yml` under the `cortex.conf` content.
> - Always confirm your Linux kernel/OS is supported for your desired mode in the Compatibility Matrix.

## Quick start

**1) Clone and adjust inventory**
```ini
[ubuntu_2404]
server1 ansible_host=10.10.10.11
server2 ansible_host=10.10.10.12

[ubuntu_2404:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa   # adjust your key path
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```

**2) Place the installer**
- Put your `.deb` at `roles/cortex_xdr/files/cortex-agent.deb`.

**3) Adjust static configuration (optional)**
- Edit `roles/cortex_xdr/tasks/main.yml` and modify the `content:` block under the *Write cortex.conf* task to match your proxy/tags/install path.

**4) Run**
```bash
ansible-playbook -i inventory.ini playbooks/deploy_xdr.yml -b
```

## Verification on a target
```bash
systemctl status traps_pmd.service
/opt/traps/bin/cytool proxy query
dpkg -l | grep cortex
```

## Repository layout
```
cortex-xdr-ansible-ubuntu24/
├── inventory.ini
├── playbooks/
│   └── deploy_xdr.yml
└── roles/
    └── cortex_xdr/
        ├── files/
        │   └── cortex-agent.deb (PLACEHOLDER path; not commit on github)
        └── tasks/
            └── main.yml
```

## References
- Install the Cortex XDR Agent for Linux — Administrator Guide: [https://docs-cortex.paloaltonetworks.com/r/Cortex-XDR/9.1/Cortex-XDR-Agent-Administrator-Guide/Install-the-Cortex-XDR-Agent-for-Linux](https://docs-cortex.paloaltonetworks.com/r/Cortex-XDR/9.1/Cortex-XDR-Agent-Administrator-Guide/Cortex-XDR-Agent-for-Linux)
- Cortex XDR — Linux Compatibility Matrix: https://docs-cortex.paloaltonetworks.com/r/Cortex-XDR/Cortex-XDR-Compatibility-Matrix/Linux
- Post-install proxy configuration via `cytool` (Broker VM/proxy): https://live.paloaltonetworks.com/t5/general-topics/how-to-install-a-cortex-xdr-agent-communicating-through-the-palo/td-p/1000112

---
**Security note**: never commit your tenant-specific installer to public repositories.
