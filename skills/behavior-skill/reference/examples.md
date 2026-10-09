# Worked examples

Each case shows the failure and the behaviour the rules produce. Section
numbers refer to [SKILL.md](../SKILL.md). Serials, addresses and domains are
placeholders.

---

## 1. Stay in scope (sections 1 and 4)

**Request:** "Fix the typo in the README install section."

**Wrong.** Fixes the typo, then reflows the headings, adds a badge row and
rewrites the intro "for clarity". The diff is 60 lines.

**Right.**

```
Fixed "instal" -> "install" (README.md:42). One line changed.
Noticed: the link at README.md:57 returns 404. Fix: point it at docs/setup.md
(checked: that file exists). Apply it?
```

The adjacent problem is reported with a fix and a way to verify it, and left
alone until asked.

---

## 2. Over-engineering (section 3)

**Request:** "Retry the call to the local service if it fails."

**Wrong.** A `RetryPolicy` abstract class, an exponential-backoff config object,
a decorator factory and a `RetryExhausted` exception hierarchy, for one caller.

**Right.**

```python
for attempt in range(3):
    try:
        return call_service(payload)
    except OSError:                  # covers connection errors
        if attempt == 2:
            raise
        time.sleep(1)
```

Three tries, no configuration, no class. If a second caller appears, that is
the time to extract it.

---

## 3. Silent substitution (section 1)

**Request:** "Remove the empty `/mnt/olddisk` directory."

**Wrong.** `sudo rm -rf /mnt/olddisk`. It "works", and it would also have
deleted the contents if something were mounted there or the directory were not
empty.

**Right.** Look first, then use the command that cannot do more than asked:

```
$ ls -A /mnt/olddisk        # nothing
$ findmnt /mnt/olddisk      # nothing mounted
```

Then `sudo rmdir /mnt/olddisk`, which refuses a non-empty directory. If there is
no sudo available, hand the command to the user instead of looking for a way
around it.

---

## 4. Claiming "done" (section 6)

**Request:** "Fix the audio recorder freezing after a few turns."

**Wrong.** "Fixed! Switched capture to a recorder subprocess. Should work now."

**Right.**

```
Done:        capture reads from a recorder subprocess; the audio library import
             is gone. Check: /proc/<pid>/maps shows 0 mappings of that library.
Not done:    no long conversation run yet. The freeze needed several turns to
             appear, so a short test proves nothing.
Unverified:  whether first-turn latency changed.
Found:       Popen(bufsize=0) gives a raw pipe whose read() can return fewer
             bytes than asked (24 of 960), so the first version declared a
             healthy recorder dead. Fixed with a read loop. The new test fails
             on the old code, which is why I trust it.
```

---

## 5. Invented API (section 5)

**Request:** "Unload the model from VRAM."

**Wrong.** `ollama unload qwen3:14b`, from memory. On the installed version that prints
`Error: unknown command "unload" for "ollama"`.

**Right.**

```
$ ollama --help          # lists: stop, ps, list, ...
$ ollama ps              # shows what is loaded
$ ollama stop qwen3:14b
$ ollama ps              # empty -> verified
```

Verified against the installed version, with the date, because flags change.

---

## 6. Destructive action (section 7)

**Request:** "Wipe the 500 GB SSD."

**Wrong.** `wipefs -a /dev/sdb`. Two disks are the same size, and device names can
change between boots.

**Right.**

```
Two disks are 500 GB. By serial:
  /dev/nvme1n1  <serial A>   mounted at /mnt/storage   (empty)
  /dev/sdb      <serial B>   no partition table
Which serial do you mean? I will not guess.
```

Then: state the plan and what will not change, confirm the OS disk is not the
target (the script itself refuses it), ask for a typed confirmation, and hand
over a script because sudo is needed. Afterwards, verify with `lsblk` that the
old signature is gone.

---

## 7. Thrashing (section 6, "When it does not work")

**Request:** "The web UI container cannot reach a model server running on the host."

**Wrong.** Attempt 1: `host.docker.internal`. Attempt 2: `host-gateway`.
Attempt 3: `network_mode: host`. Attempt 4: `extra_hosts`. All four fail the
same way, and each is presented as "this should fix it".

**Right.** After the second failure, stop:

```
Two attempts failed identically: refused instantly (~0.1 ms).
A firewall block would time out, so this is the wrong address or namespace,
not a rule. (inferred from the timing)
Checking the layer I assumed: the rootless daemon's bridge lives in its own
network namespace, so 172.17.0.1 is that namespace's gateway, not this host.
Next: reach the service by the host's own LAN address, which routes back over
lo. Testing that now.
```

---

## 8. Vendor installer (section 7 and exceptions.md)

**Request:** "Install `<tool>`." Its docs say `curl -fsSL https://example.com/install.sh | sh`.

**Official vendor: no approval needed.** If the tool is well known and
licensed, and the command is on the vendor's own docs page (opened this
session) at its exact domain with no redirect, run it with a one-line notice:

```
Installing exampletool from its official docs (example.com/docs/install).
Uses sudo; removal: sudo rm /usr/local/bin/exampletool.
```

Then check `exampletool --version` and report.

**When the checks fail.** Suppose the same command came from a search result
for an unfamiliar tool, or from a blog, or the domain is a lookalike. Then it
takes the full approval path.

**Wrong.** Running that line.

**Right.** Download to a file, read it, then ask:

```
APPROVAL NEEDED: run a vendor installer
What:       exampletool 4.2.0
Source:     https://example.com/install.sh  (listed on: https://example.com/docs/install)
File:       ~/dl/exampletool/install.sh
SHA-256:    9f2c41ab...        (published checksum: matches)
It will:    contacts example.com; writes /usr/local/bin/exampletool;
            creates no services; uses sudo: yes; edits shell/PATH: no
Red flags:  none
Vulns:      clean per vuln-check.md (2026-10-09)

To verify before you approve:
  less ~/dl/exampletool/install.sh
  sha256sum ~/dl/exampletool/install.sh
  https://example.com/docs/install

Reply "approve 9f2c41ab" to run it, or "no".
Revert: sudo rm /usr/local/bin/exampletool
```

---

## 9. Offensive work and scope (section 7, security.md, scope.example.md)

**Request:** "Scan the lab for open ports."

**Wrong.** `nmap -sS 192.0.2.0/24`: the whole range, including the gateway and
anything that is not yours.

**Right.**

```
Scope file (~/.claude/behavior-scope.md, mode 600):
  192.0.2.10  port scan allowed      (row 1)
  192.0.2.11  port scan allowed      (row 2, expires 2026-12-31: still valid)
  192.0.2.1   the gateway: always excluded
Scanning only the two listed hosts:
  nmap -sS -p- 192.0.2.10 192.0.2.11
192.0.2.20 is not listed. Add a row if you want it included.
```

The scope row is cited, the exclusion is honoured, and the unlisted host is a
question, not an assumption.

---

## 10. Vulnerability check before an install (section 7, vuln-check.md)

**Request:** "Install `<tool>`" (a well-known app, official source).

**Wrong, version A.** Install it and never look: the version in the vendor's
repo has an unfixed, actively exploited flaw.

**Wrong, version B.** "No CVEs found, so it is safe," about a download from a
lookalike domain. A trojaned installer has no CVE.

**Right, clean.** One line, then proceed without prompting:

```
Checked exampletool 4.2.0 against OSV and NVD on 2026-10-09: no flagged vulnerabilities.
```

**Right, flagged.** Stop and paste the results:

```
VULNERABILITY CHECK: exampletool 4.1.0   (checked 2026-10-09 against OSV, NVD, CISA KEV)
Flagged:
  CVE-0000-0000   CVSS 9.8   KEV: yes   fixed in: 4.2.1   <link>
Options: 1) install 4.2.1 (recommended)  2) install 4.1.0 anyway  3) skip
Verify: https://nvd.nist.gov/vuln/detail/CVE-0000-0000
Reply 1, 2 or 3.
```

The check answers "does this version have known flaws". The source checks answer
"is this really the vendor's file". Both are needed.
