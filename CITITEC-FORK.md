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


## After a release: update the MCP host's control.json

The Cititec MCP host does not pick up a new release on its own. `/etc/mcp/control.json` on the
host pins each server's image by digest **and lists every tool the server may expose**. A tool
that is in the image but not in that list does not appear for anyone.

**Every time a release adds, renames or removes a tool, update `control.json` in the same
change window**, or the new tool stays invisible (example: Bitdefender's incident and
quarantine tools, and Datto RMM's `datto_list_device_jobs`, were in the image but missing until
they were added to the list). On the host (`ssh mcp-admin`):

1. Back up the file: `sudo cp /etc/mcp/control.json /etc/mcp/control.json.bak-$(date +%Y%m%d-%H%M%S)`
2. Point the service's `image` at the new release's digest (`ghcr.io/cititec-pty-ltd/<server>@sha256:...`,
   from the GitHub package page or `docker pull` output).
3. Add new **read-only** tools to the service's `read_tools` list, and tools that change
   anything to `write_tools` (writes are approval-gated proposals). Remove renamed or deleted
   tools.
4. Check the file before anything uses it - a JSON error stops **every** MCP server:
   `sudo python3 -m json.tool /etc/mcp/control.json > /dev/null && echo "control.json is valid"`
5. Test the server: `ssh -T mcp-server mcp-<server>`, and check `tools/list` shows the new
   tools. Existing Claude sessions get "Policy changed; reconnect"; users start a new Claude
   session to see the new tools.

Also update the `cititec-mcp-servers` skill (Cititec-Pty-Ltd/Cititec, `claude/skills/`) when a
tool is added or changes behaviour, and upload the new skill zip.

## Protection

- Ruleset **Protect main**: `main` cannot be deleted or force-pushed. Normal pushes and PR merges
  still work.
- Only organization owners can delete repositories in Cititec-Pty-Ltd.
- If Wyre deletes or privatises the upstream repository, this fork stays (only Sync fork stops).
