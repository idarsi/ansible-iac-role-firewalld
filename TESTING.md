# Testing

## Automated coverage matrix

This role intentionally uses KVM only because firewalld and systemd behavior
must be exercised on a full Linux guest. The KVM suite is supplemental manual
coverage and is not run by GitHub Actions.

Platform/image | Ansible or application versions | Molecule scenarios | Main coverage
---------------|---------------------------------|--------------------|--------------
Rocky Linux 9 KVM guest from the shared Rocky 9 install ISO | Ansible Core 2.20.8 (`ansible-current`) | `kvm_states` | Validation-only state; installation; present/absent; started/stopped; permanent port and service open/close; ipset create/remove, entry add/remove, port/service rich-rule open/close; reload; package-removal verification.

The scenario covers every documented public state on one Rocky Linux 9 guest.
It does not establish automated coverage for every operating system version in
`iac_supported_os`; Rocky and RHEL 8/9/10 other than Rocky 9 remain support
claims without KVM scenario coverage. It also does not yet establish full-suite idempotence,
check-mode behavior, optional shared filesystem/cron resources, or invalid
blueprint rejection-message coverage.

## CI checks

Pull requests and pushes to `main` run Ansible syntax checking and production
profile Ansible Lint. KVM/libvirt execution is not available in CI and the
Molecule scenario is not claimed as a CI check.

## Prerequisites and commands

Use the shared [`idarsi-ansible-test-environments`](https://github.com/idarsi/idarsi-ansible-test-environments)
repository. Do not create a role-local virtual environment or install tools in
the shared environment. The KVM profile requires libvirt/KVM and the selected
Rocky Linux 9 image. The shared installer exports the ISO location as
`IDARSI_TEST_KVM_INSTALL_SOURCE`. The guest disk is 30 GB to match the shared
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
```

Run one scenario at a time because the KVM guest uses a static address on the
shared `idarsi-test` network. If a run is interrupted, inspect the scenario's
guest and libvirt resources before retrying; `molecule destroy -s kvm_states`
removes the disposable guest.
