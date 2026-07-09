---
inclusion: always
---

# Git Commits

If you are Kiro IDE, be aware that you have a bug that happens when you try to commit to git passing the commit message via the `-m` parameter. When you do that, for some reason all the newlines are stripped off the commit message. So you should write your commit message to a temporary text file inside the `.git/` directory and use it when performing the git commit.
