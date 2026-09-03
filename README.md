# ADR-000: The Sentient Workstation Playbook

**Status:** Accepted (Begrudgingly)  
**Context:** We needed a way to provision dev workstations. We used Ansible because bash scripts weren't sufficiently complicated. But that wasn't enough. We realized that writing YAML by hand is a solved problem, and the obvious next step was to invite a half-dozen autonomous LLM agents (Claude Code, Antigravity, OpenCode, Hermes, Vibe, and Whisper) into the repository to write the YAML for us.

We now have 1,627 nodes in the graph, 203 YAML files, and four different submodules just to install Flatpaks and configure DNF caching. 

**Decisions Made:**

1. **The Great Partitioning (`devworkstation` vs `llmops`)**
   We originally grouped all the AI agents together because they all run on Matrix math. But as they expanded, they needed Node, Python virtual environments, Rust, and bizarre system dependencies. We stripped the *installations* of the agents into a unified `coding_agents` role inside the `devworkstation` collection. 
   
   The `llmops` collection now exists purely as a quarantine zone for "skills, static context, agent definition instructions, and observability suites." We are literally writing configuration files to teach the agents how to write configuration files for the configuration files.

2. **Conditional DNF Makecache**
   We added conditional state hooks for `dnf makecache`. Why? Because when you run this playbook 30 times an hour to test an agent's latest hallucination, those 15 seconds matter. It’s a minor performance optimization, a `⚡️ perf` commit to salvage our collective sanity.

3. **Submodule Bloat (Modularity)**
   Because we need isolation from ourselves (and the agents), `rhel_builder`, `devworkstation`, and `llmops` are submodules. This ensures that when an agent inevitably hallucinates and breaks its own persona instructions, it doesn't accidentally revert the `base` workstation network configs.

**Consequences:**
- We have successfully built a workstation framework that is entirely focused on sustaining the tools required to build the workstation.
- We have introduced a meta-layer where the `llmops` agents read their own documentation. 
- I spend 80% of my time reviewing diffs where an agent tried to format `containerd` tasks, and 20% of my time wondering if it was Claude or Antigravity that decided to unconditionally flush iptables.
- The cognitive load has officially shifted from "knowing how Linux works" to "knowing how to prompt Hermes to remember how Linux works."

We accept this complexity, not because it is efficient, but because the alternative is writing YAML by hand. 

## The Artifacts of our Hubris

### Collection Overview

* **`b08x.devworkstation`**: The workhorse. It provisions the base system, desktop environments, networking, libvirt, SELinux, and now, ironically, the engine mounts for the LLM agents (`coding_agents` role). 
* **`b08x.llmops`**: The brain. Inference engines, CLI assistants, and agentic frameworks. It assumes the `devworkstation` has already prepared a fertile environment.

### Usage

If you really want to unleash this:

```bash
# Provision everything and hope for the best
ansible-playbook site.yml

# Just build the base system (Safe mode)
ansible-playbook site.yml --tags "base"

# Install the LLM Agents (You asked for it)
ansible-playbook site.yml --tags "coding_agents"

# Configure the LLM Agents with their static context
ansible-playbook site.yml --tags "llmops"
```

## Configuration & Testing

* **Inventory:** `inventory/hosts.ini` or `inventory/hosts.yml`.
* **Testing:** We have a three-tier testing approach (`pre-commit`, `molecule`, `pytest`), because if we don't lint the YAML the AI generates, the Ansible parser throws a stack trace that the AI can't read. 

## License
GNU General Public License v3.0 or later.
