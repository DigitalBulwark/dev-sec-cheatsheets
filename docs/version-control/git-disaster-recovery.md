# Advanced Git Disaster Recovery

A compromised repository history or a hard-reset disaster is common in fast-paced DevSecOps. Use these snippets to recover lost commits, scrub secrets from history, and cherry-pick specific changes.

## 1. Recovering a Deleted Branch or Hard Reset

If you ran `git reset --hard` by accident and lost unsynced local commits, they are not gone entirely. Git keeps an internal diary of all pointer movements called the Reflog.

> [!TIP]
> Use Reflog immediately before garbage collection runs (default is ~30 days).

```bash
git reflog
# Output looks like:
# e1b2c3a HEAD@{0}: reset: moving to HEAD~1
# f4f5f6b HEAD@{1}: commit: add payment gateway logic

git checkout -b <new-recovery-branch> f4f5f6b
```

## 2. Scrubbing Secrets from History (Leaked Keys)

If an API key or a secret was committed, deleting it in the next commit does NOT remove it from the historical `.git` object database. You must rewrite history to obliterate it.

> [!CAUTION]
> Rewriting history alters commit hashes. All your peers must pull aggressively and rebase after you force push.

Using `git filter-repo` (Preferred over `filter-branch`):
```bash
pip install git-filter-repo
git filter-repo --path-match <file_with_secret.json> --invert-paths
```

If the secret is embedded inside source code across multiple commits, you can excise the exact string:
```bash
git filter-repo --replace-text <(echo "AABBCCDD_12345==>REDACTED")
```

Force push to origin explicitly after the rewrite:
```bash
git push origin --force --all
```

## 3. Surgical Cherry-Picking

If a junior developer merged unreviewed code into Main, and you only want to extract *one specific* security patch they wrote without bringing the rest of the branch:

```bash
git cherry-pick -x <commit_hash>
```
The `-x` flag automatically appends the original commit hash to the commit message for traceability.
