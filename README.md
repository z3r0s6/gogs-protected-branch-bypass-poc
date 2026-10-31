# Gogs Protected Branch Bypass PoC

Proof of concept for a protected branch bypass in Gogs through the pull request
merge path.

Branch protection is enforced by the pre-receive hook, and the hook only runs
its checks when `SSH_ORIGINAL_COMMAND` is set. The normal write paths set it. The
PR merge push (`pr.Merge` to `git.Push`) does not, and git-module's `Push` cannot
carry environment variables anyway, so the hook exits at its first line and the
merge lands on a protected branch with no checks. A Write collaborator who is not
in the push whitelist can merge into a protected branch by opening a PR and
clicking Merge.

Tested on Gogs 0.14.3 (latest stable release).

## Requirements

- Python 3 and `git`
- Run this on the Gogs server, since it works against the repository's installed
  hooks and needs filesystem access to the bare repo
- A repository with a protected branch and a non-whitelisted Write collaborator

## Usage

```
python3 exploit.py <bare_repo_path> --branch <branch>
```

Options:

- `--branch` the protected branch to target (default `main`)
- `--author` identity to commit as, a non-whitelisted collaborator
  (default `regularuser <regular@test.local>`)
- `--user-id` `GOGS_AUTH_USER_ID` used for the control push
- `--repo-id` `GOGS_REPO_ID` used for the control push

Example:

```
python3 exploit.py /data/git/repositories/testadmin/test-repo.git --branch main
```

## What it does

1. Reads the current tip of the protected branch.
2. Pushes a commit the way the merge does it, with `SSH_ORIGINAL_COMMAND` unset,
   and shows it land on the protected branch (the hook never ran).
3. Pushes the same change again with the hook environment set, and shows it get
   rejected (the hook works fine when it is fed).

## Safety

Run this only against a Gogs instance you own or have explicit permission to
test. The script restores the branch tip when it finishes, so it leaves the
repository as it found it.
