---
description: Replace a string across the repo, then commit and push
argument-hint: "<old text>" "<new text>"
allowed-tools: Bash(grep:*), Bash(sed:*), Bash(git:*), Read, Edit
---

Replace one string with another across this repository, then commit and push.

Arguments: $ARGUMENTS

Parse the arguments into OLD and NEW. They are normally two quoted strings —
`/replace "My Dashboard" "Roman's Empire"`. If you cannot confidently tell where
OLD ends and NEW begins, stop and ask rather than guessing.

Follow these steps in order. Stop and report if any step fails.

## 1. Find

```
grep -rn "OLD" . --include="*.html" --include="*.js" --include="*.css" --include="*.md" | grep -v node_modules | grep -v "\.git/"
```

Report the file:line hits. If there are **zero** matches, stop — say so and do
not commit. If OLD appears in files you did not expect (docs, specs, vendored
code), list them and ask which are in scope before editing anything.

## 2. Replace

Edit each in-scope file. With `sed`, pick a delimiter that appears in neither
string (`|` or `#`, not `/` if the text has slashes) and escape any regex
metacharacters in OLD:

```
sed -i '' "s|OLD|NEW|g" <file>
```

Single quotes inside the text (`Roman's`) are fine inside a double-quoted sed
expression. Prefer the Edit tool when a string is awkward to escape.

## 3. Verify

Re-run the grep from step 1. It must return no matches. Print the changed lines
so the new text is visible in the transcript.

## 4. Commit and push

```
git add <changed files>
git commit -m "<imperative one-line summary of the rename>"
git push
```

Then confirm the push actually reached the remote — `git push` printing nothing
useful, or "Everything up-to-date", is not proof on its own:

```
git ls-remote origin refs/heads/$(git branch --show-current)
git rev-parse HEAD
```

Those two hashes must match. Report the pushed range (`old..new`) and the final
hash.

If the push fails on authentication, do not retry it repeatedly — report that no
GitHub credential is configured and that `gh auth login` (or a PAT, or an SSH
key) is needed. The commit stays safely on the branch until then.

## Note on this repo

`row/` is a second clone of this same repository. It is untracked from the outer
repo, so commits here do not cover it. If it contains OLD too, apply the same
replacement to its working tree so the copies do not drift, leave it
uncommitted, and say so — committing in both clones against one remote creates
divergence.
