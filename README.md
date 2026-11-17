# RHEL 9 Hardening Demo

Companion repository for the "RHEL Hardening" video series:

- **Part 1** builds the hardening playbook with AI + Ansible DevTools in VS Code.
- **Part 2** runs the same content as policy in Ansible Automation Platform:
  a workflow of pre-flight check mode run, approval gate, apply, and verify.

## Contents

| File | Purpose |
|---|---|
| `hardening.yml` | Applies the baseline: SSH (drop-in + `sshd -t` validation), firewalld (SSH only, target DROP), password policies (pwquality + login.defs), disables cups/avahi |
| `verify.yml` | Read-only audit mirroring the same CIS controls, reports PASS/FAIL per control, exits non-zero on any failure. Safe to schedule for drift detection |
| `inventory/inventory.ini.example` | Inventory template, copy to `inventory/inventory.ini` |
| `requirements.yml` | Collections (`ansible.posix`) |

## Usage

```bash
ansible-galaxy collection install -r requirements.yml
cp inventory/inventory.ini.example inventory/inventory.ini
# edit inventory.ini with your RHEL 9 hosts

ansible-playbook hardening.yml                 # apply
ansible-playbook verify.yml                    # audit, read-only
ansible-playbook hardening.yml --tags cis_5.1.20   # single CIS control
```

Targets RHEL 9 (and EL9 rebuilds); both playbooks fail early on anything else.

## CIS mapping

Tasks are tagged with CIS Red Hat Enterprise Linux 9 Benchmark v2.0.0
control numbers:

| Area | Controls |
|---|---|
| SSH | 5.1.1, 5.1.4, 5.1.9, 5.1.20 |
| firewalld | 4.2.1 |
| Password quality | 5.3.3.2.x |
| Password aging | 5.4.1.x |
| Services (cups, avahi) | 2.1.2, 2.1.11 |

`PasswordAuthentication no` is also applied in the sshd drop-in; it is not a
numbered v2.0.0 control on RHEL 9 but complements 5.1.20.

## Notes

- These are demo defaults (roughly CIS L1 Server intent). Review every value
  against your own environment and benchmark version before using for real.
- The SSH drop-in lands in `/etc/ssh/sshd_config.d/`, which RHEL 9 includes
  at the top of `sshd_config`, so those values take precedence. Run
  `hardening.yml` from a session you can afford to lose access from if you
  get `PasswordAuthentication` wrong.
