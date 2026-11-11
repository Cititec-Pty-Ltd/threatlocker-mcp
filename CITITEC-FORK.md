# Cititec fork

This is Cititec's fork of [WYRE-AI/threatlocker-mcp](https://github.com/WYRE-AI/threatlocker-mcp). The upstream
project is licensed under Apache-2.0; that licence and its notices still apply to this fork.

Read this before changing anything here: it says which changes go back to Wyre and which stay
in this fork.

## Two kinds of change

| Kind of change | Branch from | Goes to |
|----------------|-------------|---------|
| Fix or improvement that is useful to everyone | Wyre's current `main` (`upstream/main`) | A PR to `WYRE-AI/threatlocker-mcp`, **and** the same branch merged into our `main` |
| Cititec-only: package name (`@cititec-pty-ltd/...`), `.npmrc`, release workflow, this file | Our `main` | Our `main` only - never in a PR to Wyre |

Never open a PR to Wyre from our `main`: it carries the Cititec-only changes, and Wyre would have
to reject it.

## Set up once

```bash
git clone https://github.com/Cititec-Pty-Ltd/threatlocker-mcp.git
cd threatlocker-mcp
git remote add upstream https://github.com/WYRE-AI/threatlocker-mcp.git
```

## Send a fix to Wyre

```bash
git fetch upstream
git switch -c fix/<short-name> upstream/main
# make the change, add tests, update CHANGELOG.md
git push -u origin fix/<short-name>

# PR to Wyre
gh pr create --repo WYRE-AI/threatlocker-mcp --base main --head Cititec-Pty-Ltd:fix/<short-name>
# The same branch into our main, so we have the fix straight away
gh pr create --repo Cititec-Pty-Ltd/threatlocker-mcp --base main --head fix/<short-name>
```

Both this fork and Wyre's repository are **public**. Keep commits, tests and PR text free of
client names, hostnames, user names, organization or computer IDs and other internal details -
use made-up example values (`WS-01`, `C:\Temp\setup.msi`, a dummy GUID).

## Bring Wyre's updates in

```bash
git fetch upstream
git switch main
git pull
git merge upstream/main     # merge - never rebase
# if there are conflicts, they are usually package names: keep @cititec-pty-ltd
git push
```

Do not rebase our `main` or force-push it. The **Protect main** ruleset blocks force-pushes and
deleting `main` on purpose, so the fork's history cannot be lost. GitHub's **Sync fork** button
only works while there are no conflicts; otherwise merge as above.


## After a release

Adding, renaming or removing a tool also needs a change on Cititec's MCP host, or the new tool
does not appear for anyone. Follow the internal runbook in the private Cititec-Pty-Ltd/Cititec
repository (`claude/mcp-host/README.md`), and update the `cititec-mcp-servers` skill there.

## Protection

- Ruleset **Protect main**: `main` cannot be deleted or force-pushed. Normal pushes and PR merges
  still work.
- Only organization owners can delete repositories in Cititec-Pty-Ltd.
- If Wyre deletes or privatises the upstream repository, this fork stays (only Sync fork stops).
