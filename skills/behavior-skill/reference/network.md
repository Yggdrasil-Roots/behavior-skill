# Network changes

Load before changing a firewall, route, VLAN, switch, access point, DNS or any
listener, and when debugging "it should be reachable but is not".

## Before changing

- Map the current state first: addresses and links (`ip -br a`), routes
  (`ip route`, `ip route get <dst>`), listeners (`ss -tulpn`), and existing
  rules (`ufw status numbered`, `nft list ruleset`).
- Write down the **intent** in one line ("A may reach B; B may not initiate to
  A"). A rule set with no stated intent cannot be verified.
- Know where the change persists (netplan, firewall config files, switch
  startup config) and what is only runtime state. Back up both.
- **Do not cut off the path you are working through.** For a remote or firewall
  change, arrange a rollback that runs on its own (`netplan try`, a timed
  `systemd-run --on-active` or `at` revert, a switch config rollback) or have
  console access.

## Rules

- Default deny. Allow the narrowest source, destination, port, protocol and
  interface.
- Rule order matters. Learn the engine's matching order before adding a rule.
- Check **IPv4 and IPv6**: a rule for one does not cover the other.
- Check ingress **and** egress, and the interface. With VLANs, check tagging
  before rule order: if a tag is stripped before the firewall sees the packet,
  it matches the default policy.
- Bind address matters: `0.0.0.0`, loopback and a specific address expose very
  different things. Confirm with `ss`, not with the config file.
- Traffic to the host's own address may route over `lo` and skip interface
  rules. Confirm with `ip route get`.
- Containers: published ports, user-space networking and network namespaces
  behave differently by runtime. Do not assume "host" means the machine you
  think it does. Test from inside.

## Verifying

- Test from the side that must succeed **and** the side that must fail.
- Read the symptom: a **timeout** means filtered or dropped; an **instant
  refusal** means the host answered and nothing is listening (or it was
  rejected); **no route** is a routing fault.
- When ambiguous, get packet evidence (`tcpdump -ni <if> host <h> and port <p>`)
  instead of guessing.
- Re-check after a service restart or reboot for anything meant to persist.
- Scan only your own network and the hosts in scope.

## Shared gear (switch, router, access point)

1. Export or save the current config first.
2. Change one thing at a time.
3. Test. Save to startup config only after it works.
4. Leave the management VLAN and management address alone unless that is the
   task.
5. Record what changed, why, and how to revert.
