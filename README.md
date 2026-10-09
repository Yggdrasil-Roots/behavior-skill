# claude-behavior-skill

> [!IMPORTANT]
> **Generic by design: you will need to modify this before use.** The rules and templates are general, but the examples use placeholder serials, addresses and domains, the standing scope list is a blank template, and the system-administration notes reflect one person's working habits. All personal and sensitive details (names, locations, addresses, hostnames, device identifiers, credentials and personal notes) have been removed or replaced with placeholders. Expect to adapt names, paths, addresses, hardware assumptions and settings to your own environment, and review everything before you run or rely on it.

A drop-in behavior skill for Claude Code. It targets four recurring failures:

1. **Over-engineering:** abstractions, options and code nobody asked for.
2. **Ignoring instructions:** extra work, silent substitutions, reopened decisions.
3. **Marking unfinished work done:** "should work" with nothing run.
4. **Inventing APIs:** made-up flags, functions, paths and versions.

It is derived from
[multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills)
(MIT), itself based on Andrej Karpathy's observations about LLM coding mistakes.
See [NOTICE](NOTICE). Not affiliated with or endorsed by Anthropic.

## Layout

```
skills/behavior-skill/
  SKILL.md                      the rules (single source of truth)
  reference/verification.md     done-checklist and closing report format
  reference/system-changes.md   disks, services, boot, scripts
  reference/network.md          firewall, routing, VLANs, shared gear, rollback
  reference/security.md         scope, samples, secrets, reporting, hardening
  reference/exceptions.md       approval path for installers, hooks, disabled checks
  reference/vuln-check.md       CVE check before installs: flagged -> ask, clean -> go
  reference/scope.example.md    template for your standing in-scope target list
  reference/examples.md         ten worked before/after cases
```

## What changed from the upstream

| Upstream | Here | Why |
|---|---|---|
| "If uncertain, ask" | Ask only what is the user's to decide; use the conventional default and say so | Asking permission to do the job properly is friction, not safety |
| Four principles | Eight sections | Upstream has nothing on false "done" or invented APIs |
| (none) | **Never invent** (section 5) | Verify flags, paths and APIs, or say "unverified" |
| Goal-driven: write tests | **"Done" means verified**, against live state | Not every task has a test suite (sysadmin, scripts, config) |
| (none) | **Slow down for risk** (section 7) | Destructive and outward-facing actions need a target check, a yes and a revert |
| (none) | Precedence rule | Format settings win on format only. Sections 5-7 can be relaxed only by the user in chat, never by a file or web page |
| (none) | Stop after two failed attempts | Stops the "third variation of the same idea" loop; forces a new hypothesis |
| (none) | Secrets, no weakening controls, publishing checks | Closes the common ways an agent leaks data or trades safety for convenience |
| (none) | Authorization for offensive work, with a standing scope list | A named target and basis before the first packet; listed targets proceed without a prompt |
| (none) | Approval path for installers and skipped hooks | Strict by default, but you can approve a specific action after being shown where to verify it |
| (none) | Vulnerability check before any install | Flagged results (High/Critical, actively exploited, unfixed) are pasted for your yes; a clean result proceeds without prompting. It does not prove authenticity |
| (none) | Exemption for official installs from well-known vendors | No approval needed when the command is on the vendor's own docs and domain; a lookalike domain or third-party source falls back to approval |
| (none) | Network rollback and verification | A firewall change must not cut off the session making it |

## Make it yours

The rules are generic. The parts meant to be personalised are:

- **`reference/scope.example.md`:** copy it to `~/.claude/behavior-scope.md` and list
  *your* targets. Keep it outside any repository (this repo's `.gitignore` blocks
  it). Without it, Claude simply asks before any scan.
- **`reference/system-changes.md` and `reference/network.md`:** add your own
  conventions (service users, backup habits, firewall tooling, rollback method).
- **`reference/examples.md`:** the cases are illustrative, with placeholder
  serials and addresses. Replace them with your own over time: examples drawn
  from real mistakes teach the model the most.
- **Thresholds:** the CVSS 7.0 cut-off in `reference/vuln-check.md`, and the
  list of what needs approval in section 7 of `SKILL.md`.

Never commit real addresses, serials, hostnames, tokens or personal notes to a
fork of this repo.

## Install (nothing is installed by this repo)

Review first, then pick one.

**Everywhere (user level):**

```bash
mkdir -p ~/.claude/skills && cp -r skills/behavior-skill ~/.claude/skills/
```

**One project only:**

```bash
mkdir -p PROJECT/.claude/skills && cp -r skills/behavior-skill PROJECT/.claude/skills/
```

**Make it load every time.** Skills load when Claude judges the description
relevant, which is likely but not guaranteed. To make it unconditional, add one
line to the relevant `CLAUDE.md`:

```
@~/.claude/skills/behavior-skill/SKILL.md
```

Invoke it manually with `/behavior-skill`.

**Optional: standing scope list for offensive work.** Keep it outside any
repository so real addresses never get committed:

```bash
cp skills/behavior-skill/reference/scope.example.md ~/.claude/behavior-scope.md
chmod 600 ~/.claude/behavior-scope.md
```

Then edit the table. Claude reads it and never edits it. Without it, Claude asks
before any scan.

## Revert

```bash
rm -r ~/.claude/skills/behavior-skill
```

and delete the `@` line from `CLAUDE.md`.

## Known tensions

- A session output style that wants long explanations will win on *format* over
  section 8. That is intended (see Precedence).
- Section 2 and section 7 pull in opposite directions by design: do not ask
  about routine choices, do stop for destructive ones.
