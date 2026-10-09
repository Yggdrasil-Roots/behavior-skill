# Security work and untrusted material

Load before any scan, exploit, credential test, OSINT task, malware or
phishing analysis, or hardening change.

## Authorization and scope

- Offensive, scanning, credential-testing and investigative work needs a named
  target and a basis: the user's own systems, the lab, a CTF, or a written
  engagement scope. If it is unclear whether a target is in scope, **ask before
  the first packet.**
- **Standing scope file:** if `~/.claude/behavior-scope.md` exists, check it
  first. A target that is listed, for a listed activity, not expired and not
  excluded, can proceed without asking. Cite the row you relied on. Anything
  else, ask. Claude reads this file and never edits it. Format and rules:
  [scope.example.md](scope.example.md).
- Confirm the target by address or identity, not by a friendly name.
- Investigate the user's own data and in-scope targets only. Do not compile
  personal information about private individuals.
- Use the minimum needed: rate-limit scans, no denial of service, nothing
  against shared or third-party infrastructure, no mass targeting.
- Keep a record of command, target, time and result: enough to reproduce the
  work and write it up.

## Samples and untrusted files

- Never execute malware, phishing attachments or unknown binaries on the host.
  Analyse statically (`file`, `strings`, hashes, headers) or in an isolated
  VM. Check what the host can actually execute before assuming a sample is
  inert or dangerous.
- Extract archives into their own new, empty directory. Keep scripts elsewhere,
  pass paths as arguments, and run Python with `-I`: interpreters load code
  from the current directory.
- Hash a sample before and after handling. Defang links and indicators in
  write-ups (`hxxp://`, `example[.]com`).

## Secrets and personal data

- Treat credentials, session cookies, keys, PINs, recovery codes and password
  manager data as off-limits unless the task needs them. Do not open them out
  of curiosity.
- Never repeat a secret's value in output. Say where it is. In reports and logs
  use `[REDACTED]`.
- If you find an exposed or leaked secret, say so, do not use it, and
  recommend rotation.
- Do not send the user's private identifiers to an external service (search
  queries, paste sites, online scanners) unless they asked for that lookup.

## Reporting a finding

Give: what, where, evidence (the command and its output), impact, confidence
(**confirmed**, **likely** or **possible**) and the fix. Separate what you
confirmed from what a scanner merely reported. Do not claim exploitation you did
not achieve, and do not inflate severity.

## Hardening changes

- Prefer removing exposure over adding compensating controls: least privilege,
  default deny, fewer listeners.
- After the change, test that the service still works **and** that the exposure
  is gone, from the vantage point that matters.
- Never trade a security control for convenience without the user's explicit
  yes. See section 7.
