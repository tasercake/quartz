---
---

#git
From [this StackOverflow thread](https://stackoverflow.com/questions/54408829/granular-access-to-directories-within-monorepo)

With `git subtree` you can:

- create a monorepo composed of subtrees, each of which can be linked to separate remote repos
- have a single aggregate/unified history (the point of a monorepo)
- pull changes from subtree remotes into the monorepo
- push changes made in any subtree of the monorepo to its separate remote
- keep your simple/easy workfows.

  > `git subtree` does not require users of your repository to learn anything new. They can ignore the fact that you are using `git subtree` to manage dependencies."
