<div align="center">
  <h1>b08x workstation</h1>
  <p>Workstation configuration management with Ansible — a work-in-progress exercise in deterministic provisioning.</p>
</div>

<div align="center">

![License](https://img.shields.io/badge/license-GPL--3.0--or--later-blue)
![Ansible](https://img.shields.io/badge/ansible--core-%3E%3D2.15.0-orange)
![Status](https://img.shields.io/badge/status-WIP-yellow)

</div>

## Features

- **Collection-Based Architecture** — Separates infrastructure concerns (`devworkstation`) from AI tooling (`llmops`) using distinct Ansible Collections under the `b08x` namespace.
- **Distro-Aware Role Execution** — Dynamically loads distribution-specific tasks via `tasks/distro/{{ ansible_distribution }}.yml`, supporting Fedora and AlmaLinux without conditional sprawl.
- **Dual Inventory Support** — Maintains both YAML (`hosts.yml`) and INI (`hosts.ini`) inventory formats, allowing flexible group hierarchies and variable scoping.
- **Fact Caching** — Persists gathered facts to JSON files with 24-hour TTL, eliminating redundant discovery on subsequent runs.
- **Plugin Extensibility** — Includes custom callback, filter, and action plugins alongside standard modules, extending Ansible core with domain-specific logic.
- **Enforced Quality Gates** — Pre-commit hooks run Black, isort, Flake8, Prettier, and ansible-lint; tox-ansible and molecule handle integration testing.

## Architecture

```
ansible/
├── site.yml                              # Primary playbook (targets workstations)
├── ansible.cfg                           # Execution configuration
├── collections/ansible_collections/b08x/
│   ├── devworkstation/                   # Core infrastructure
│   │   ├── roles/
│   │   │   ├── base/                     #   System packages, shell, utilities
│   │   │   ├── desktop/                  #   Desktop environment, window manager
│   │   │   ├── networking/               #   Network configuration, DNS, firewall
│   │   │   ├── virt/                     #   Virtualization (libvirt, QEMU)
│   │   │   ├── selinux/                  #   SELinux policy management
│   │   │   ├── tuning/                   #   Kernel/sysctl tuning for dev workloads
│   │   │   └── run/                      #   Runtime execution helpers
│   │   ├── plugins/                      #   Custom modules, filters, lookups
│   │   └── extensions/molecule/          #   Test scenarios
│   └── llmops/                           # AI/ML tooling
│       ├── roles/
│       │   ├── crush/                    #   Crush utility
│       │   ├── opencode/                 #   OpenCode CLI
│       │   ├── ollama/                   #   Local inference engine
│       │   ├── claude/                   #   Claude CLI assistant
│       │   ├── hermes/                   #   Hermes agent
│       │   ├── antigravity/              #   Antigravity framework
│       │   ├── vibe/                     #   Vibe tooling
│       │   ├── whisper/                  #   Speech-to-text
│       │   └── run/                      #   Runtime execution helpers
│       └── plugins/                      #   Custom modules, filters, lookups
├── inventory/
│   ├── hosts.yml                         # Primary YAML inventory
│   ├── hosts.ini                         # Alternate INI inventory
│   └── group_vars/                       # Group variable definitions
├── plugins/                              # Root-level callback/filter plugins
├── .github/workflows/                    # CI pipelines
└── .pre-commit-config.yaml               # Linting hooks
```

### Collection Descriptions

The `b08x.devworkstation` collection handles base system provisioning: package management, shell configuration, desktop environments, networking, virtualization, SELinux, and kernel tuning. Roles follow standard Ansible structure with `argument_specs.yml` for typed parameter validation.

The `b08x.llmops` collection deploys local AI/ML tooling: inference engines (Ollama), CLI assistants (Claude, OpenCode), speech-to-text (Whisper), and agentic frameworks (Antigravity, Hermes, Vibe). These roles assume a functioning base system from `devworkstation`.

> **Note:** Collection-level READMEs currently contain auto-generated placeholder content. Role-specific documentation is minimal. Contributions to improve either are welcome.

## Prerequisites

- **Ansible Core** >= 2.15.0
- **Python** >= 3.10
- **SSH access** to target workstations (keys or agent forwarding)
- **Privilege escalation** (sudo) on targets for system-level tasks

## Installation

<details>
<summary>Clone and Install Dependencies</summary>

```bash
# Clone the repository
git clone <repository_url> ansible-project
cd ansible-project

# Install collection dependencies
ansible-galaxy collection install -r collections/requirements.yml

# Verify Ansible version
ansible --version
```

</details>

<details>
<summary>Alternative: Install Collections Individually</summary>

```bash
# Install devworkstation collection
ansible-galaxy collection install b08x.devworkstation

# Install llmops collection
ansible-galaxy collection install b08x.llmops

# Or install from a specific version
ansible-galaxy collection install b08x.devworkstation:==1.0.0
```

</details>

## Usage

```bash
ansible-playbook site.yml
```

The main playbook targets the `workstations` group (tinybot, gir, soundbot) with privilege escalation enabled. Fact gathering runs automatically; PATH is augmented with `~/.local/bin` and `~/.cargo/bin`.

### Playbook Options

- `--tags "base"`: Run only base system configuration tasks.
- `--tags "desktop"`: Provision desktop environment components.
- `--tags "antigravity"`: Deploy the Antigravity framework.
- `--tags "virt"`: Configure virtualization support.
- `--limit <host>`: Target a single host or subset.
- `--check`: Dry-run mode; show changes without applying.
- `--diff`: Show file differences alongside task output.
- `--verbose` / `-vvv`: Increase output verbosity for debugging.

### Examples

```bash
# Provision all workstations
ansible-playbook site.yml

# Provision only base system on a single host
ansible-playbook site.yml --tags "base" --limit gir

# Dry-run with diff output
ansible-playbook site.yml --check --diff

# Deploy AI tooling only
ansible-playbook site.yml --tags "antigravity"

# Run linting suite
pre-commit run --all-files

# Execute molecule tests from a collection
cd collections/ansible_collections/b08x/devworkstation
molecule test

# Run parallel pytest
pytest -vvv -n 2
```

## Configuration

Execution behavior is controlled by `ansible.cfg` at the project root.

### Key Settings

```ini
[defaults]
inventory               = ./inventory/hosts.yml
forks                   = 10
gathering               = smart
inject_facts_as_vars    = True
stdout_callback         = ansible.posix.debug
callbacks_enabled       = profile_tasks
fact_caching            = jsonfile
fact_caching_connection = /tmp/ansible_cache
fact_caching_timeout    = 86400
error_onUndefined_vars  = True
log_path                = /tmp/ansible.log

[ssh_connection]
ssh_args                = -C -o ControlMaster=auto -o ControlPersist=60s
pipelining              = True
```

| Setting | Purpose |
|---|---|
| `fact_caching = jsonfile` | Persists facts to `/tmp/ansible_cache` for 24 hours |
| `pipelining = True` | Reduces SSH overhead by keeping connections open |
| `profile_tasks` | Enables per-task timing in output |
| `stdout_callback = ansible.posix.debug` | Verbose task-level output formatting |

### Inventory

Target hosts are defined in `inventory/hosts.yml`:

```yaml
all:
  hosts:
    tinybot:
      ansible_host: 192.168.41.11
    gir:
      ansible_host: 192.168.41.10
    soundbot:
      ansible_host: 192.168.41.13
  children:
    workstations:
      hosts:
        tinybot:
        gir:
        soundbot:
```

Group variables in `inventory/group_vars/all.yml` define user-specific parameters:

```yaml
user:
  name: "b08x"
  home: "/home/b08x"
  shell: "/bin/zsh"
```

## Testing

The project uses a three-tier testing approach:

```bash
# Linting (from project root)
pre-commit run --all-files

# Molecule tests (from collection directory)
cd collections/ansible_collections/b08x/devworkstation
molecule test

# Unit tests with parallel execution
pytest -vvv -n 2

# Tox matrix (runs across Python/Ansible versions)
tox
```

### Test Structure

| Tool | Scope | Location |
|---|---|---|
| pre-commit | Code style, linting | `.pre-commit-config.yaml` |
| molecule | Role integration | `extensions/molecule/` |
| pytest | Unit, integration | `tests/unit/`, `tests/integration/` |
| tox-ansible | Version matrix | `tox-ansible.ini` |

## Contributing

Contributions require passing all pre-commit hooks and test suites. Development workflow:

```bash
# Install pre-commit hooks
pre-commit install

# Run all hooks against staged files
pre-commit run --all-files

# Run tests before submitting
pytest -vvv -n 2
```

Standards enforced: Black (line-length=100), isort (profile=black), Flake8, Prettier (YAML/TOML), ansible-lint. All tests must pass against ansible-core >= 2.15.0.

## License

GNU General Public License v3.0 or later. See [LICENSE](https://www.gnu.org/licenses/gpl-3.0.txt) for the full text.
