# Testing

## Automated coverage matrix

This role intentionally uses KVM only because firewalld and systemd behavior
must be exercised on a full Linux guest. The KVM suite is supplemental manual
coverage and is not run by GitHub Actions.

Platform/image | Ansible or application versions | Molecule scenarios | Main coverage
---------------|---------------------------------|--------------------|--------------
Rocky Linux 9 KVM guest from the shared Rocky 9 install ISO | Ansible Core 2.20.8 (`ansible-current`) | `kvm_states` | Validation-only state; installation; present/absent; started/stopped with boot enablement checks; permanent port and service open/close; ipset create/remove, entry add/remove, port/service rich-rule open/close; reload; package-removal verification; unchanged role-only second run for `port_open`.
Rocky Linux 10 KVM guest from the shared Rocky 10 install ISO | Ansible Core 2.20.8 (`ansible-current`) | `kvm_rocky10` | Same all-states and representative `port_open` idempotence coverage as `kvm_states`, using a separate guest address.

Both KVM scenarios exercise every documented public state on their respective
Rocky guest. The second-run assertion registers only the repeated role include,
so verification tasks cannot affect the changed result. Idempotence is currently
asserted only for stable `port_open` convergence, not every mutating state. The
scenarios do not establish automated coverage for Rocky Linux 8 or any RHEL
release; those remain support claims without KVM scenario coverage. They also
do not cover check-mode behavior, optional shared filesystem/cron resources, or
invalid blueprint rejection-message coverage.

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
  molecule/kvm_states/converge.yml
molecule test -s kvm_states
molecule test -s kvm_rocky10
```

Run one scenario at a time. The Rocky 9 and Rocky 10 scenarios use distinct
static addresses (`192.168.250.221` and `192.168.250.222`) on the shared
`idarsi-test` network. If a run is interrupted, inspect the scenario's guest
and libvirt resources before retrying; `molecule destroy -s <scenario>` removes
the disposable guest.
