<!--# cspell: ignore SSOT CMDB -->
# AGENTS.md

Ensure that all practices and instructions described by
<https://raw.githubusercontent.com/ansible/ansible-creator/refs/heads/main/docs/agents.md>
are followed.

## Project Structure

Four Ansible Collections under `collections/ansible_collections/b08x/`, each a git submodule:
- **devworkstation** — base, desktop, libvirt, networking, containerd, tuning, selinux, run, coding_agents (antigravity, claude, crush, opencode, vibe, skills)
- **llmops** — run, ollama, hermes, dify, langfuse, tts
- **rhel_builder** — image_builder, run
- **context** — run, plus plugins (action, cache, filter, inventory, lookup, modules, test)

Submodules must be initialized before a first run:

```bash
git submodule update --init --recursive
```

`.gitignore` ignores `collections/ansible_collections/b08x/*` and re-includes each
collection explicitly. Adding a fifth collection requires a matching `!` negation
line there as well as a `.gitmodules` entry — otherwise it is silently untracked.

Main playbook: `site.yml` targets `workstations` group (tinybot, gir, soundbot).
Standalone deployment playbooks live at the repo root (e.g. `langfuse-podman-tinybot.yml`).

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

- Inventory: `inventory/hosts.ini` (default) or `inventory/hosts.yml`
- Group vars: `group_vars/dev.yml` (and `group_vars/all.yml`)
- Host vars: `host_vars/{{ inventory_hostname }}.yml` (e.g. `host_vars/tinybot.yml`)
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

### Variable Naming & Scoping
- **Tier 1 (Host & Domain Scope)**: Semantic, unprefixed names (`user.*`, `intel_oneapi_install`, `enable_third_party_repos`, `use_containers`, `use_kvm`) represent machine hardware, primary user identity (SSOT), or system-wide capabilities in `group_vars/` or `host_vars/`. Do NOT force artificial role prefixes onto them, as doing so distorts their semantic meaning and forces redundant multi-role copies. `var-naming[no-role-prefix]` is deliberately skipped in `.ansible-lint` for this reason.
- **Tier 2 (Role Scope)**: Variables defined in role `defaults/main.yml`, `vars/main.yml`, and `set_fact` must be role-prefixed (`<role>_*`) to maintain clean namespaces and avoid cross-role leakage.
- **Task Registrations**: Variables created via `register:` must be role-prefixed (`<role>_*`) because registered variables have global host scope in Ansible and will collide across roles if left generic.

### Firewall & Port Management
- Port and firewall rules are **co-located** within the specific role or application task that requires them (e.g. using `ansible.posix.firewalld`), rather than centralized into a standalone firewall role. This ensures services remain self-contained, modular, and manage their own ingress needs directly.

## Agent Skills

Repo-local skills live in `.agents/skills/<name>/SKILL.md` and are loaded on demand:
- **trackboi** — project/board/card tracking via MCP (see block below)
- **langfuse** — Langfuse tracing, datasets, evaluation; pairs with the `b08x.llmops.langfuse` role

<trackboi>
### trackboi Skill

When trackboi MCP tools are available, agents can load `.agents/skills/trackboi/SKILL.md` for details, then call `orient_agent` to catch up before updating cards, tracks, boards, or handoff notes. If `.trackboi`, `.etc/.trackboi`, or `.etc/trackboi` files are present but MCP tools are not available, agents may read those files to catch up on local context. Do not manually create, update, or delete trackboi records in the filesystem; use MCP tools for mutations.
</trackboi>
