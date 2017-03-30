## Concepts

1. What are the differences between git remote prune, git prune, git fetch --prune, etc ?
> There are potentially three versions of every remote branch:
>
> 1. The actual branch on the remote repository
> 2. Your snapshot of that branch locally (stored under refs/remotes/...)
> 3. And a local branch that might be tracking the remote branch

Let's start with `git prune`. This removes objects that are no longer being `referenced`, not references. Suppose you have a local branch, that means there is a ref named `random_branch_i_want_deleted` that refers to some objects that represent the history of that branch.

So, by definition, `git prune` will not remove `random_branch_I_want_deleted`. It's really a way to delete data that has accumulated in Git but is not being referenced by anything useful. In general, it doesn't affect your view of any branches.

`git remote prune origin` and `git fetch --prune` both operate on references under `refs/remotes/...` (I'll refer to these as remote references). 

It doesn't affect local branches. The `git remote` version is useful if you only want to remove remote references under a particular remote. Otherwise, the two do exactly the same thing. So, in short, `git remote prune` and `git fetch --prune` operate on number 2 above.

To remove a `local branch`, you should use `git branch -d` (or -D if it's not merged anywhere). FWIW, there is `no git command` to `automatically` remove the local tracking branches if a remote branch disappears.