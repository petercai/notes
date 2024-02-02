# get-sizer

https://github.com/github/git-sizer


# # BFG Repo-Cleaner

## Usage

First clone a fresh copy of your repo, using the [`--mirror`](https://stackoverflow.com/q/3959924/438886) flag:

```
$ git clone --mirror git://example.com/some-big-repo.git
```

This is a [bare](https://git-scm.com/docs/gitglossary.html#def_bare_repository) repo, which means your normal files won't be visible, but it is a _full_ copy of the Git database of your repository, and at this point you should **make a backup of it** to ensure you don't lose anything.

Now you can run the BFG to clean your repository up:

```
$ java -jar bfg.jar --strip-blobs-bigger-than 100M some-big-repo.git
```

The BFG will update your commits and all branches and tags so they are clean, but it doesn't physically delete the unwanted stuff. Examine the repo to make sure your history has been updated, and then use the standard [`git gc`](https://git-scm.com/docs/git-gc) command to strip out the unwanted dirty data, which Git will now recognise as surplus to requirements:

```
$ cd some-big-repo.git
$ git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

Finally, once you're happy with the updated state of your repo, push it back up _(note that because your clone command used the `--mirror` flag, this push will update **all** refs on your remote server)_:

```
$ git push
```

At this point, you're ready for everyone to ditch their old copies of the repo and do fresh clones of the nice, new pristine data. It's best to delete all old clones, as they'll have dirty history that you _don't_ want to risk pushing back into your newly cleaned repo.


## Examples


Delete all files named 'id_rsa' or 'id_dsa' :

```
$ bfg --delete-files id_{dsa,rsa}  my-repo.git
```

Remove all blobs bigger than 50 megabytes :

```
$ bfg --strip-blobs-bigger-than 50M  my-repo.git
```