# Controller configuration as code — RHEL hardening demo

Recreates every Controller object this demo needs, idempotently, against an
[AAP sandbox](https://developers.redhat.com/developer-sandbox) (or any AAP
2.x instance):

| Object | Details |
|---|---|
| Project `RHEL hardening demo` | SCM from this repo, update on launch |
| Inventory `RHEL hardening demo` | one host, `rhel9-node1` |
| Machine credential | username + SSH key (key read from a local file, never committed) |
| Job template `RHEL hardening - apply` | `hardening.yml`, privilege escalation on |
| Job template `RHEL hardening - verify` | `verify.yml`, read-only audit |
| Workflow `RHEL hardening - workflow` | pre-flight (check mode + diff) → approval → apply → verify |

Re-running the playbook updates the existing objects in place (same names),
so it is safe to run repeatedly.

## What you need

- The IP of a reachable RHEL 9 target and an SSH private key that works on it.
  On a Developer Sandbox with OpenShift Virtualization, create the VM with the
  sandbox's own tooling (`workflow-sandbox-rhel`), then fetch the per-VM key
  from the `aap-lab-validation-ssh` secret in your namespace and the pod IP
  from the VirtualMachineInstance status.
- API access to Controller: an OAuth token **or** admin user + password.
- Collection: `ansible.controller` (see below).

## Option A — run from your laptop

1. Install the collection from Red Hat Automation Hub:

   ```bash
   export ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN=<hub token from console.redhat.com>
   cd aap-sandbox-casc
   ansible-galaxy collection install -r requirements.yml
   ```

2. Run the playbook:

   ```bash
   ansible-playbook site.yml \
     -e controller_hostname=https://<your-aap-host> \
     -e controller_oauthtoken=<API token> \
     -e demo_target_ip=<vm ip> \
     -e demo_ssh_key_path=./ssh_key
   ```

   Password auth works too: replace `controller_oauthtoken` with
   `-e controller_username=admin -e controller_password=...`.

## Option B — run from AAP itself (no collection install)

The certified execution environments already bundle `ansible.controller`.
Bootstrap by hand once — create a project pointing at this repo and a job
template that runs `aap-sandbox-casc/site.yml` with `localhost` as the
execution target and the Controller token as an extra variable — then launch
it. Everything else in this directory is created by the platform itself.

## Variables

| Variable | Required | Default | Purpose |
|---|---|---|---|
| `controller_hostname` | yes | — | Controller base URL |
| `controller_oauthtoken` | yes* | — | OAuth token (*or username/password) |
| `controller_username` / `controller_password` | yes* | — | Alternative auth |
| `demo_target_ip` | yes | — | Target VM IP (update after recreating the VM) |
| `demo_target_user` | no | `cloud-user` | SSH user on the target |
| `demo_ssh_key_path` | yes | — | Path to the target's private key |

The SSH key file is never read into git — `ssh_key` is gitignored. Only the
credential inputs inside Controller hold it.
