# System administration changes

Load before changing a disk, a service, the firewall, boot configuration, or
anything under `/etc`.

## Principles

- **Diagnose before changing.** Report what is actually there. Machines diverge
  from defaults (service users, rootless containers, custom mounts), and the
  standard answer is often wrong here.
- **Identify disks by serial or UUID,** never by device name or size. Names
  such as `nvme1n1` can swap between boots, and two disks can share a size.
- **Check what the target holds** (`lsblk`, `df`, `ls`) before any wipe. Say what
  you found.
- **Never touch the OS disk or any system mount** in a destructive operation.
  Refuse in the script itself, not only in your head.

## Scripts that change system state

- Idempotent: running it twice is safe.
- Print the plan and the target, then ask before acting.
- Back up first (`file.bak.<epoch>`), do not delete.
- Verify against **live state** afterwards (`findmnt`, `lsblk`, `/proc/<pid>`),
  not by trusting that a write succeeded.
- Include revert steps for any service, firewall, mount or boot change.
- Write a script file for the user to run. Do not print an indented heredoc
  for copy-paste: an indented terminator never matches and the shell hangs.
- If it needs `sudo` and none is available, hand over the script. Do not
  search for a workaround.

## Mount and boot

- Mount by `UUID=`, with `nofail` for data disks, so a missing disk cannot block
  boot.
- Take a backup of `/etc/fstab` before editing.
- With encrypted root, anything involving recovery, cloning or resizing needs
  the unlock step considered first.

## Firewall and network

See [network.md](network.md). In short: state the intent first, keep a rollback
that does not depend on the session you are changing, and test from both sides.
