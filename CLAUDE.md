# Memory Consolidation

## Strategies and Hard Rules

### Anti-patterns and Pitfalls

**Context**: When running Libvirt VMs (e.g., via Vagrant) on a host that also runs Docker.

**Pattern**: Prevent silent network failures caused by Docker's strict `FORWARD` DROP policy conflicting with Libvirt's `nftables` bridge routing.
```yaml
approach: |
  Explicitly allow Libvirt bridge interfaces (`virbr+`) to forward traffic in Docker's custom iptables chain (`DOCKER-USER`).
validation: VM should be able to ping an external IP (e.g., 8.8.8.8) or download packages without curl timeouts.
examples:
  - case: Vagrant/Libvirt VM silently hangs on network operations (e.g., dnf update)
    implementation: |
      sudo iptables -I DOCKER-USER -i virbr+ -j ACCEPT
      sudo iptables -I DOCKER-USER -o virbr+ -j ACCEPT
```

**Avoid**: Assuming Libvirt's native `nftables` rules are sufficient when Docker is installed. Docker manages `iptables` and its default `FORWARD DROP` policy supersedes `nftables` accepts.

- **Docker iptables hijack**: Causes silent drops of `virbr+` outbound NAT traffic.
- **Vagrant provisioning hangs**: Network timeouts lead to infinite hangs during inline shell scripts without clear error messages.

**Confidence**: High

**Source**: 2026-09-02 Root Cause Trace
