# Git Tips

A bunch of useful commands for git (at least I find it usefull)

## use different account
It happened that we have different git accounts which belong to different private keys (one account = one private key) and, on a same computer, we want to push to a repo from a specific account. 
Here one way to achieve that.

### Change the SSH key used for push

- Run this one-off command in the repository that needs a different key. It stores the override inside `.git/config`.

```bash
git config core.sshCommand "ssh -i <path_to_private_key> -o IdentitiesOnly=yes -F /dev/null"
```

- Replace `<path_to_private_key>` with the absolute path to the key you want to use (for example: `/Users/bob/.ssh/id_rsa_alt`).
- Git now reaches the remote using that key only for this repo, leaving other repositories untouched.

## Safety nets when things break

- `git reflog` lists every commit your HEAD touched, even the ones no longer in the branch history. Use it whenever you lose track of a commit.
- Found the commit you need? Create a rescue branch before doing anything risky:

```bash
git checkout -b rescue/<topic> <commit_hash_from_reflog>
```

- Not sure what you are about to do will work? Drop a quick backup tag so you can return instantly:

```bash
git tag backup/$(date +%Y%m%d-%H%M) HEAD
```

- After recovering, delete temporary rescue tags/branches once you are confident the history is safe.

## Reorganize commits

- Need to tweak the last commit? Amend it without creating a new one:

```bash
git commit --amend
```

- Combine or reorder several recent commits with an interactive rebase. Pick the range you want to edit (here, the last three commits):

```bash
git rebase -i HEAD~3
```

	- Inside the editor, change `pick` to `reword`, `squash`, or `fixup` to adjust the history.
	- If you use `fixup`/`squash`, finish the rebase and Git will prompt you to edit the merged commit message.

- Polishing commits stacked on top of new work? Mark quick fixes with `--fixup` so autosquash reorders them automatically:

```bash
git commit --fixup <commit_hash>
git rebase -i --autosquash <base_branch>
```

- After rewriting history, force-push with care if the branch is already published:

```bash
git push --force-with-lease
```