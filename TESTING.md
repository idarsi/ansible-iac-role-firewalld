# Testing

## Automated coverage matrix

This role intentionally uses KVM only because firewalld and systemd behavior
must be exercised on a full Linux guest. The KVM suite is supplemental manual
coverage and is not run by GitHub Actions.

Platform/image | Ansible or application versions | Molecule scenarios | Main coverage
---------------|---------------------------------|--------------------|--------------
Rocky Linux 9 KVM guest from the shared Rocky 9 install ISO | Ansible Core 2.20.8 (`ansible-current`) | `kvm-states` | Validation-only state; installation; present/absent; started/stopped with boot enablement checks; permanent port and service open/close; ipset create/remove, entry add/remove, port/service rich-rule open/close; reload; package-removal verification.
Rocky Linux 10 KVM guest from the shared Rocky 10 install ISO | Ansible Core 2.20.8 (`ansible-current`) | `kvm-rocky10` | Same all-states coverage as `kvm-states`, using its converge and verify playbooks and a separate guest address.
Rocky Linux 9 KVM guest from the shared Rocky 9 install ISO | Ansible Core 2.20.8 (`ansible-current`) | `kvm-idempotence` | Focused installation, `port_open`, ipset-entry-present, and inbound ipset-port-rich-rule convergence; Molecule's idempotence action reruns the playbook and fails when any task reports changed.

The `kvm-states` and `kvm-rocky10` scenarios exercise every documented public
state on their respective Rocky guest. Idempotence is separately verified for
installation and stable `port_open` convergence in `kvm-idempotence`; the
Molecule idempotence action checks task-level changed results from a complete
second playbook run, rather than the aggregate result of an `include_role` task.
The `kvm-states` play also checks the permanent ipset's type and entry, then
queries the entry after both its absent and present states. Rich
rule removal assertions verify all expected inbound/outbound and legacy rule
forms are absent, while unrelated rules survive ipset removal. This does not
establish idempotence for every mutating state. The idempotence scenario also
converges an ipset entry and inbound port rich rule on each run. The states
scenario verifies shared directory, file-content, and cron-file resources;
bind-mount runtime behavior is not covered. Local validator tests require
unsafe filesystem and cron/file records to fail at their expected validation
task with a corresponding actionable error token. The
scenarios do not establish automated coverage for Rocky Linux 8 or any RHEL
release; those remain support claims without KVM scenario coverage. They also
do not cover check-mode behavior or bind-mount runtime behavior. CI syntax-checks the validator helper playbook and
the Molecule converge/idempotence playbooks; it does not execute those test
plays.

## CI checks

Pull requests and pushes to `main` run Ansible syntax checking and production
profile Ansible Lint. KVM/libvirt execution is not available in CI and the
Molecule scenario is not claimed as a CI check.

## Prerequisites and commands

Use the shared [`idarsi-ansible-test-environments`](https://github.com/idarsi/idarsi-ansible-test-environments)
repository. Do not create a role-local virtual environment or install tools in
the shared environment. The KVM profile requires libvirt/KVM and the selected
Rocky Linux 9 or 10 image. The shared installer exports Rocky 9 as
`IDARSI_TEST_KVM_INSTALL_SOURCE` and Rocky 10 as
`IDARSI_TEST_KVM_INSTALL_SOURCE_ROCKY10`. Guest disks are 30 GB to match the shared
KVM installer's `KVM_BASE_DISK_SIZE_GB` default and reuse its prepared base
image.

```bash
cd /path/to/idarsi-ansible-test-environments
./install.sh kvm ansible-current
source .venv/ansible-current/bin/activate
cd /path/to/ansible-iac-role-firewalld
ANSIBLE_ROLES_PATH=.. ansible-lint --profile production
ANSIBLE_ROLES_PATH=.. ansible-playbook --syntax-check \
  molecule/kvm-states/converge.yml
molecule test -s kvm-states
molecule test -s kvm-rocky10
molecule test -s kvm-idempotence
```

Run one scenario at a time. The Rocky 9 and Rocky 10 scenarios use distinct
static addresses (`192.168.250.221`, `192.168.250.222`, and
`192.168.250.223`) on the shared
`idarsi-test` network. If a run is interrupted, inspect the scenario's guest
and libvirt resources before retrying; `molecule destroy -s <scenario>` removes
the disposable guest.
