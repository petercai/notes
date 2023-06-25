# Git Internals - How Git works 
### Motivation

Before I joined LinkedIn in 2013, I had been using a Centralized Version Control System (VCS) like SVN for more than 3 years. Moving to Git wasn’t easy initially but [this](https://www.youtube.com/watch?v=ZDR433b0HJY) video from Scott Chacon helped me understand the Git fundamentals. Since then, I’ve been a firm believer of using Git over any other VCS and I thought I had a good understanding of Git. Last week, I had a discussion on Git internals with a couple of my friends and all of us realized that we were not thorough with this topic and hence this is my attempt in getting better at it.

This post talks about the internals of Git and assumes a fair understanding of a VCS. To read about various git commands - consider [this](http://byte.kde.org/~zrusin/git/git-cheat-sheet-medium.png) cheat-sheet or [this](http://git-scm.com/book/en/v2) book.

### Introduction

Before we dive into git internals, a couple of notes on git.

1.  Git is open-sourced — Feel free to explore it’s code [here](https://github.com/git/git).
2.  Git is a decentralized VCS — In a centralized VCS, the database resides on a central server and you **checkout** a copy from the server. Most of the commands require you to contact the central database and hence require network access. In a decentralized or distributed VCS, each and every node has a copy of the database and hence you **clone** a copy from a remote server. Note that the remote server has no special permissions except for the fact that all the nodes have access to the remote server. As a result of this, most of the commands on git (except _git push_ or _git pull_) can be performed without network access.

When you clone a git repository, it creates a .git folder at the root of the repository. This is where git stores all the data. This is a snapshot of the folder — 

![](_assets/1520215883499.jpg)

1.  The **branches** directory isn’t used by newer Git versions.
2.  The **description** file is used to provide a name to the repository with the description and is only used by the Web UI.
3.  The **config** file contains your project-specific configuration options
4.  The **info** directory keeps a global exclude file for ignored patterns that you don’t want to track in a .gitignore file.
5.  The **hooks** directory contains your client- or server-side hook scripts.
6.  The **index** file is where Git stores your staging area information.

This blog post would go in detail about the 3 main components in this .git folder — 

1.  Objects directory — The objects directory stores all the content of the git database.
2.  Refs directory — The refs directory stores pointers into commit objects in that data.
3.  Head file — The HEAD file points to the branch you currently have checked out.

#### Git Objects

At the core, git is nothing but a **key-value data store**. Git uses [SHA-1](http://en.wikipedia.org/wiki/SHA-1) hash of the content (content could either refer a file or commit or directory structure — we’ll get to it soon) as the key and the content itself (compressed) acts as the value. In order to understand more about git objects, let’s walk-through an example.

Consider a simple repository with just 1 commit with the contents of a file called first.txt — “This is the first line.”

![](_assets/1520175074860.jpg)

Git repository with 1 commit

Let us take a look at the objects directory in .git folder -

![](_assets/1520214741924.jpg)

Git Objects

As you can see, it has created 3 directories (**59**, **c5** and **dd**) besides the info and the pack (which we’ll cover later in this blog post).

**Content Objects** — The first folder is the content of the file itself. You can view the contents of by running the _git cat-file_ command.

![](_assets/1520214254087.jpg)

Content Object

**Tree Objects —**The second folder is a tree object. Git stores the file system structure in these tree objects. The first column shows the unix permissions, the second column is either blob or tree depending on whether it is a pointer to a file or another directory, the third is the hash of the object pointed to, and the fourth is the filename. In this case there is only one file tracked by git, first.txt, and you can see that this tree node reflects that by listing one file, and pointing to the blob holding its contents.

![](_assets/1520233414243.jpg)

Tree Object

**Commit Objects**  — The third folder is the commit, with a header containing author, committer details and time-stamp, followed by the commit message itself. If you type git log you will recognize that the commit number is just the hash of this commit object.

![](_assets/1520212263339.jpg)

Commit Object

When you make a change to first.txt and commit it, this will create 3 more folders — the first one will be a **snapshot** of the latest file, the second one will be for the folder structure pointing to the latest commit and the third one is for the commit. Here’s the screenshot showing the latest snapshot — 

![](_assets/1520174963167.jpg)

Seeing the number of loose files in objects directory grow.

All those diffs that we’re used to seeing on Git’s UI are calculated on the fly. _When you commit, git stores snapshots of the entire file, it does not store diffs from the previous commit._ As a repository grows, the object count grows exponentially and clearly it becomes inefficient to store the data as loose object files. Hence, git packs them and stores them as a .pack file.

#### Git Packs

As the repository grows bigger, it becomes inefficient for Git to store snapshots of every version of every file. Consider this example — 

1\. I add a huge file (~20 MB) to git and commit it. This is what the git objects directory looks like — 

![](_assets/1520215677452.jpg)

Screenshot of the objects directory (760 KB) after committing a large file to git.

2\. I make a small change to the file and then commit it again. This is what the git objects directory looks like —

![](_assets/1520041546135.jpg)

Screenshots of the objects directory (1.5MB) after a small change to the large file.

What has happened is, git has duplicated the copy of the file and the .git directory size has doubled. You can see why this wouldn’t scale. Wouldn’t it be nice if Git could store one of them in full but then the second object only as the delta between it and the first? Git can do this and this is what pack objects are for. Git packs up several of these objects into a single binary file called a “packfile” in order to save space and be more efficient. This can be initiated in 2ways — 

1.  Manually by running the ‘git gc’ command.
2.  Automatically by pushing to the git server.

![](_assets/1520211444814.jpg)

Screenshot of the git gc command and compressed git objects size to 456K.

You can view the contents of the pack by running the git verify-pack command.

![](_assets/1520214591982.jpg)

Viewing the contents of a packfile in git.

Note that the 9127 blob (the content of the large text file in the 2nd commit) references the 50ee blob (the content from the 1st commit). What is also interesting is that the second version of the file is the one that is stored intact, whereas the original version is stored as a delta — this is because you’re most likely to need faster access to the most recent version of the file.

#### Git Refs

Let’s take a look at what’s there in the refs directory — 

![](https://:0)

The heads directory has a file for every branch in your repository containing the SHA-1 hash of the latest commit in that repository.

If you add a remote and push to it, Git stores the SHA-1 value you last pushed to that remote for each branch in the refs/remotes directory.

The tag folder has references to a tag object. A tag object is very much like a commit object — it contains a tagger, a date, a message, and a pointer. The main difference is that a tag object generally points to a commit rather than a tree. It’s like a branch reference, but it never moves — it always points to the same commit but gives it a friendlier name.

#### Git Head

The HEAD file is a symbolic reference to the branch you’re currently on. By symbolic reference, we mean that unlike a normal reference, it doesn’t generally contain a SHA-1 value but rather a pointer to another reference. If you look at the file, you’ll normally see something like this:

![](https://:0)

A look at the .git/HEAD file.

#### References

This is just an introduction to the internals of Git and there's lots more to this. You can read more from the references below -

1.  [http://git-scm.com/book/en/v2/](http://git-scm.com/book/en/v2/)
2.  [http://slidetocode.com/2013/08/25/how-git-works/](http://slidetocode.com/2013/08/25/how-git-works/)