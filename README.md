<div align="center">
  <h1>devworkstation</h1>
  <p>System provisioning and LLM operations automation for local development environments.</p>
</div>

## Features

- **Automated Provisioning** — Bootstraps base system dependencies, networking, virtualization, and desktop environments consistently across nodes, eliminating artisanal system setups.
- **Local LLM Operations** — Deploys inference engines and AI assistants including Ollama, Claude, and local agentic tools directly to the host.
- **System Tuning & Hardening** — Configures SELinux and applies system tuning parameters optimized for intensive development workloads.
- **Modular Architecture** — Separates core infrastructure (`devworkstation`) from AI tooling (`llmops`) using distinct Ansible collections.
- **Robust Quality Assurance** — Enforces code standards through Molecule tests, parallel pytest execution, and strict pre-commit hooks.

## Installation

<details>
<summary>Control Node Setup</summary>

```bash
# Clone the configuration repository
git clone <repository_url> ansible-project
cd ansible-project

# Install necessary collections and dependencies
ansible-galaxy collection install -r collections/requirements.yml
```

</details>

## Usage

Execution relies on standard Ansible playbook invocations targeting the defined workstation inventory (`tinybot`, `gir`, `soundbot`).

```bash
ansible-playbook site.yml
```

### Options

- `--tags "base"`: Limit execution to base system configuration tasks.
- `--tags "desktop"`: Provision desktop environment components only.
- `--tags "always"`: Enforce execution of critical prerequisite tasks regardless of other tags.

### Examples

```bash
# Provision the entire workstation environment
ansible-playbook site.yml

# Execute only base system and networking roles
ansible-playbook site.yml --tags "base,networking"

# Run the linting suite across the codebase
pre-commit run --all-files

# Execute parallel Molecule tests from a collection directory
pytest -vvv -n 2
```

## Configuration File

Target hosts and user-specific parameters are defined within the inventory and group variables, isolating deployment specifics from the logic—because hardcoded usernames rarely survive first contact with a second machine.

```yaml
# inventory/group_vars/all.yml
user:
  name: "b08x"
  home: "/home/b08x"
  shell: "/bin/zsh"
```

### Configuration Options

- `user.name`: Primary user account name for the workstation environment.
- `user.home`: Absolute path to the designated user's home directory.
- `user.shell`: Default login shell path for the primary user.

## Devworkstation Collection

This collection manages the core infrastructure requirements for development systems, aggressively fighting the natural entropy of manual configuration drift. It standardizes the deployment of base packages, networking configurations, and desktop utilities. Execution relies on distribution-specific tasks dynamically loaded based on system facts.

- **Roles:** `base`, `desktop`, `networking`, `tuning`, `selinux`, `virt`, `run`

## LLMOps Collection

Local AI capabilities require dedicated infrastructure provisioning. This collection deploys inference engines and API-driven assistants directly onto the workstation, supporting localized models and agentic toolchains without relying solely on cloud infrastructure.

- **Roles:** `antigravity`, `claude`, `crush`, `hermes`, `ollama`, `opencode`, `vibe`, `whisper`, `run`

## Project Structure

The directory layout adheres to standard Ansible structural conventions. Collections encapsulate roles, while the root directory houses the primary playbook, inventory, and continuous integration definitions.

```
ansible-project/
├── collections/
│   └── ansible_collections/
│       └── b08x/
│           ├── devworkstation/
│           └── llmops/
├── inventory/
│   ├── group_vars/
│   └── hosts.yml
├── site.yml
└── ansible.cfg
```

## Contributing

Contributions require adherence to established linting rules and Molecule test coverage. Development standards are enforced via pre-commit hooks running Black, isort, Flake8, Prettier, and Ansible-lint. All tests must pass against Ansible-core >= 2.15.0 prior to integration.

## License

This project is distributed under the MIT License.
