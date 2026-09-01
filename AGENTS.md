<!--# cspell: ignore SSOT CMDB -->
# AGENTS.md

Ensure that all practices and instructions described by
<https://raw.githubusercontent.com/ansible/ansible-creator/refs/heads/main/docs/agents.md>
are followed.

## Project Structure

Two Ansible Collections under `collections/ansible_collections/b08x/`:
- **devworkstation** — base system, desktop, virt, networking, tuning, selinux, run
- **llmops** — crush, opencode, run, antigravity, vibe, ollama, hermes, claude, whisper

Main playbook: `site.yml` targets `workstations` group (tinybot, gir, soundbot).

## Commands

```bash
# Run main playbook
ansible-playbook site.yml

# Run specific role only
ansible-playbook site.yml --tags "base"

# Lint (from collection dir)
ansible-lint

# Run molecule tests (from collection dir)
molecule test

# Run pre-commit on all files
pre-commit run --all-files
```

## Configuration

- Inventory: `inventory/hosts.yml`
- Group vars: `inventory/group_vars/all.yml`
- User vars define: `user.name`, `user.home`, `user.shell`
- ansible.cfg enables: fact caching (jsonfile), profile_tasks callback, pipelining
- Logs go to `.logs/` and `/tmp/ansible.log`

## Testing

- tox-ansible skips Python 3.7/3.8 and ansible-core < 2.14
- molecule scenarios in `extensions/molecule/`
- pytest: `pytest -vvv -n 2` (parallel execution)
- Requires: ansible-core >= 2.15.0

## Linting

Pre-commit hooks (both collections):
- **black** (line-length=100)
- **isort** (import sorting)
- **flake8** (Python linting)
- **prettier** (YAML/TOML formatting)
- **ansible-lint** (Ansible best practices)

## Conventions

- Roles follow standard structure: tasks/, defaults/, vars/, meta/, handlers/
- Distribution-specific tasks in `tasks/distro/{{ ansible_distribution }}.yml`
- Tags: use `tags: ['always']` for critical tasks, domain tags for filtering
- Variable precedence: role defaults < group_vars < host_vars < extra vars
