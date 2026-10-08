# Repository instructions

## Publication and Codex Changes

The user requires published work to stop appearing as pending Changes in Codex.
After every authorized commit and push, and after switching the working branch:

1. Confirm the intended local branch tracks its matching published branch on origin.
2. Verify the remote branch SHA directly and confirm it equals local HEAD.
3. Check staged, unstaged, and untracked files. Do not leave task source or documentation
   changes only locally; publish them when authorized. Preserve unrelated user changes
   and report them instead of deleting or committing them without permission.
4. Verify the Codex comparison base is the matching origin branch, not an obsolete
   branch or upstream release. The post-publication comparison must be empty.
5. Check the local origin/HEAD comparison reference. If it is missing or points to
   an obsolete base for this workflow, align it with the current published origin
   branch using git remote set-head origin <branch>. This is local metadata, shared
   by worktrees of the same repository; it does not change GitHub's default branch.
6. Open the Codex Branch review against that explicit origin branch. Selecting
   Unstaged alone may not clear the sidebar counter. Verify the displayed counter
   when UI access is available; otherwise state that UI confirmation is pending.
7. Never reset, delete code, rewrite history, force-push, merge unrelated branches,
   or change GitHub's default branch just to hide the counter.

Remote-tracking HEAD and Codex view settings are not transported by commits. On
another PC, repeat these checks after checkout. Do not claim the Changes counter
has disappeared based solely on a clean git status or a successful push.

