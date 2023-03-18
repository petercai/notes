# Git fundamentals, a complete guide
If you already work with [Git](https://git-scm.com/book/en/v2) daily but want to have a good comprehension of Git **fundamentals**, then this post is for you.

Here, you'll have the chance to truly understand the **Git architecture** and how commands such as **add, checkout, reset, commit, merge, rebase, cherry-pick, pull, push** and **tag** work internally.

_Don't let Git master you_, learn the Git fundamentals and **master Git instead**.

Brace yourselves, a **complete guide about Git** is about to start.

* * *

[](#first-things-first)💡 First things first
--------------------------------------------

You must practice _while you read_ this post.

Following along, let's first create a new project called `git-101` and then initialize a git repository with the command `git init`:  

```
$ mkdir git-101
$ cd git-101 
```

Enter fullscreen mode Exit fullscreen mode

The Git CLI provides two types of commands:

*   **plumbing**, which consists of _low-level commands_ used by Git internally behind the scenes when users type _high-level_ commands
    
*   **porcelain**, which are the _high-level_ commands commonly used by Git users
    

In this guide, we'll see how the **plumbing commands relate to the porcelain commands** that we use day-to-day.

* * *

[](#the-git-architecture)⚙️ The Git architecture
------------------------------------------------

Inside the project _which contains a Git repository_, we can check the Git components:  

```
$ ls -F1 .git/

HEAD
config
description
hooks/
info/
objects/
refs/ 
```

Enter fullscreen mode Exit fullscreen mode

We're going to focus on the main ones:

*   .git/objects/
    
*   .git/refs
    
*   HEAD
    

Let's analyse each component **in detail**.

* * *

[](#the-object-database)💾 The Object Database
----------------------------------------------

Using the UNIX tool `find` we can see the structure of the `.git/objects` folder:  

```
$ find .git/objects

.git/objects
.git/objects/pack
.git/objects/info 
```

Enter fullscreen mode Exit fullscreen mode

In Git, everything is persisted in the `.git/objects` structure, which is the **Git Object Database**.

What kind of content can we persist in Git? _Everything_.

> 🤔 **Wait!**
> 
> How is that possible?

Through the use of [hash functions](https://en.wikipedia.org/wiki/Hash_function).

### [](#hashing-for-the-rescue)🔵 Hashing for the rescue

A **hash function** _maps data of arbitrary, dynamic size into fixed-size values_. By doing this, we can store/persist anything because the final value will have always the _same size_.

Bad implementations of hash functions can easily lead to **collisions**, where two different dynamic-size data could map to the same final hash of fixed-size.

[SHA-1](https://en.wikipedia.org/wiki/SHA-1) is a well-known implementation of the hash function that is in general safe and hardly has collisions.

Let's take, for instance, the hashing of the string `my precious`:  

```
$ echo -e "my precious" | openssl sha1
fa628c8eeaa9527cfb5ac39f43c3760fe4bf8bed 
```

Enter fullscreen mode Exit fullscreen mode

_Note: If you're using Linux, you can use the command_ `sha1sum` instead of `OpenSSL`.

### [](#comparing-differences-in-the-content)🔵 Comparing differences in the content

A _good hashing_ is a safe practice where **we can't know the raw value**, i.e doing the reverse engineering.

In case we want to know _if the value has changed_, we just wrap the value into the hashing function and _voilà_, we can compare the difference:  

```
$ echo -e "my precious" | openssl sha1
fa628c8eeaa9527cfb5ac39f43c3760fe4bf8bed

$ echo -e "no longer my precious" | openssl sha1
2e71c9ae2ef57194955feeaa99f8543ea4cd9f9f 
```

Enter fullscreen mode Exit fullscreen mode

If the _hashes are different_, then we can assume that the **value has changed**.

Can you spot an opportunity here? What about using **SHA-1** to store data and _just keep track of everything_ by comparing hashes?

That's exactly what Git does internally 🤯.

### [](#git-and-sha1)🔵 Git and SHA-1

**Git uses SHA-1 to generate hashing** of everything and stores it in the `.git/objects` folder. _Simple like that!_

The **plumbing** command `hash-object` does the job:  

```
$ echo "my precious" | git hash-object --stdin
8b73d29acc6ae79354c2b87ab791aecccf51701f 
```

Enter fullscreen mode Exit fullscreen mode

Let's compare with the `OpenSSL` version:  

```
$ echo -e "my precious" | openssl sha1
fa628c8eeaa9527cfb5ac39f43c3760fe4bf8bed 
```

Enter fullscreen mode Exit fullscreen mode

**Oooops**...it's quite different. That's because Git _prepends a specific word_ followed by the _content size_ and the delimiter `\0`. Such a word is what Git calls **the object type**.

Yes, _Git objects have types_. The first one we'll look into is **the blob object**.

### [](#the-blob-object)🔵 The blob object

When we send for instance the string "my precious" to the `hash-object` command, Git prepends the pattern `{object_type} {content_size}\0` to the SHA-1 function, so that:  

```
blob 12\0myprecious 
```

Enter fullscreen mode Exit fullscreen mode

Then:  

```
$ echo -e "blob 12\0my precious" | openssl sha1
8b73d29acc6ae79354c2b87ab791aecccf51701f

$ echo "my precious" | git hash-object --stdin
8b73d29acc6ae79354c2b87ab791aecccf51701f 
```

Enter fullscreen mode Exit fullscreen mode

**Yay!** 🎉

### [](#storing-blobs-in-the-database)🔵 Storing blobs in the database

But the command `hash-object` itself does not persist into the `.git/objects` folder. We should append the option `-w` and the object will be persisted:  

```
$ echo "my precious" | git hash-object --stdin -w
8b73d29acc6ae79354c2b87ab791aecccf51701f

$ find .git/objects
...
.git/objects/8b
.git/objects/8b/73d29acc6ae79354c2b87ab791aecccf51701f

### Or, simply
$ find .git/objects -type f
.git/objects/8b/73d29acc6ae79354c2b87ab791aecccf51701f 
```

Enter fullscreen mode Exit fullscreen mode

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--ywn--PTz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9m8l1485y5a0d8cwog1o.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--ywn--PTz--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9m8l1485y5a0d8cwog1o.png)

### [](#reading-the-raw-content-of-a-blob)🔵 Reading the raw content of a blob

We already know that for cryptographic reasons it's not possible to read the raw content from its hashing version.

> 🤔 Ok, but wait.
> 
> How does Git _get to know the original value_?

It uses the hash as a **key pointing to a value**, which is the _original content itself_ using a compression algorithm called [Zlib](https://en.wikipedia.org/wiki/Zlib), that **compacts the content** and stores it in the object database, hence **saving storage space**.

The **plumbing** command `cat-file` does the job so that, _given a key_, inflates the compressed data thus getting the original content:  

```
$ git cat-file -p 8b73d29acc6ae79354c2b87ab791aecccf51701f
my precious 
```

Enter fullscreen mode Exit fullscreen mode

In case you are guessing, that's right, **Git is a key-value database**!

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--RFwCeLdE--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jhjl5r81esbokdfo6gqs.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--RFwCeLdE--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jhjl5r81esbokdfo6gqs.png)

### [](#promoting-blobs)🔵 Promoting blobs

When using Git, we want to work on the content and **share them with other people**.

Commonly, after working on various files/blobs, we are ready to _share them and sign our names for the final work_.

In other words, we need to **group, promote and add metadata to our blobs.** This process works as follows:

1.  Add the blob to a _staging area_
    
2.  Group **all blobs** in the stage area into a _tree structure_
    
3.  Add **metadata** to the _tree structure_ (author name, date, a semantic message)
    

Let's see the _above steps in detail_.

### [](#stage-area-the-index)🔵 Stage area, the index

The **plumbing** command `update-index` allows to add a blob to the stage area and give a name to it:  

```
$ git update-index \
    --add \
    --cacheinfo 100644 \
    8b73d29acc6ae79354c2b87ab791aecccf51701f \
    index.txt 
```

Enter fullscreen mode Exit fullscreen mode

*   `--add`: adds the blob to the stage, also called **the index**
    
*   `--cacheinfo`: used to register a file that is not in the working directory yet
    
*   the blob hash
    
*   `index.txt`: a name for the blob in the index
    

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--1X5S86kt--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dhcrrhbui33ao66muizk.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--1X5S86kt--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dhcrrhbui33ao66muizk.png)

Where does Git _store the index_?  

```
$ cat .git/index

DIRCsҚjT¸zQp    index.txtÆ
                          7CJVVÙ 
```

Enter fullscreen mode Exit fullscreen mode

It's not human-readable though, it's compressed using Zlib.

We can add as many blobs to the index as we want, for example:  

```
$ git update-index {sha-1} f1.txt
$ git update-index {sha-1} f2.txt 
```

Enter fullscreen mode Exit fullscreen mode

After adding blobs to the index, we can group them into a **tree structure** which is ready to be promoted.

### [](#the-tree-object)🔵 The tree object

When using the **plumbing** command `write-tree`, **Git groups all blobs that were added to the index** and create another object in the `.git/objects` folder:  

```
$ git write-tree
3725c9e313e5ae764b2451a8f3b1415bf67cf471 
```

Enter fullscreen mode Exit fullscreen mode

Checking the `.git/objects` folder, note that a new object was created:  

```
$ find .git/objects

### The new object
.git/objects/37
.git/objects/37/25c9e313e5ae764b2451a8f3b1415bf67cf471

### The blob previously created
.git/objects/8b
.git/objects/8b/73d29acc6ae79354c2b87ab791aecccf51701f 
```

Enter fullscreen mode Exit fullscreen mode

Let's retrieve the original value using `cat-file` to understand better:  

```
### Using the option -t, we get the object type
$ git cat-file -t 3725c9e313e5ae764b2451a8f3b1415bf67cf471
tree

$ git cat-file -p 3725c9e313e5ae764b2451a8f3b1415bf67cf471
100644 blob 8b73d29acc6ae79354c2b87ab791aecccf51701f index.txt 
```

Enter fullscreen mode Exit fullscreen mode

That's an interesting output, it's quite different from the blob which _returned the original content_.

_In the tree object_, Git returns **all the objects that were added to the index**.  

```
100644 blob 8b73d29acc6ae79354c2b87ab791aecccf51701f index.txt 
```

Enter fullscreen mode Exit fullscreen mode

*   `100644`: the cacheinfo
    
*   `blob`: the object type
    
*   the blob hash
    
*   the blob name
    

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--AcDvEW36--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sf62qc9mlvcgde8hzyb2.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--AcDvEW36--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sf62qc9mlvcgde8hzyb2.png)

Once the promotion is done, **time to add some metadata to the tree**, so we can declare the author's name, date and so on.

### [](#the-commit-object)🔵 The commit object

The **plumbing** command `commit-tree` receives a tree, a commit message and creates another object in the `.git/objects` folder:  

```
$ git commit-tree 3725c -m 'my precious commit'
505555f4f07d90ae14a0f2e67cba7f7b9af539ee 
```

Enter fullscreen mode Exit fullscreen mode

What kind of object is it?  

```
$ find .git/objects
...
.git/objects/50
.git/objects/50/5555f4f07d90ae14a0f2e67cba7f7b9af539ee

### cat-file
$ git cat-file -t 505555f4f07d90ae14a0f2e67cba7f7b9af539ee
commit 
```

Enter fullscreen mode Exit fullscreen mode

What about its value?  

```
$ git cat-file -p 505555f4f07d90ae14a0f2e67cba7f7b9af539ee

tree 3725c9e313e5ae764b2451a8f3b1415bf67cf471
author leandronsp <leandronsp@example.com> 1678768514 -0300
committer leandronsp <leandronsp@example.com> 1678768514 -0300

my precious commit 
```

Enter fullscreen mode Exit fullscreen mode

*   `tree 3725c`: **the referencing tree object**
    
*   author/committer
    
*   the commit message **my precious commit**
    

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--dCQIKcq3--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rx73s2uu3uyi8367a47y.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--dCQIKcq3--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rx73s2uu3uyi8367a47y.png)

> 🤯 OMG! Am I seeing a pattern here?

Furthermore, commits can reference other commits:  

```
$ git commit-tree 3725c -p 50555 -m 'second commit'
5ea578a41333bae71527db537072534a199a0b67 
```

Enter fullscreen mode Exit fullscreen mode

Where the option `-p` allows referencing a **parent commit**:  

```
$ git cat-file -p 5ea578a41333bae71527db537072534a199a0b67

tree 3725c9e313e5ae764b2451a8f3b1415bf67cf471
parent 505555f4f07d90ae14a0f2e67cba7f7b9af539ee
author leandronsp <leandronsp@gmail.com> 1678768968 -0300
committer leandronsp <leandronsp@gmail.com> 1678768968 -0300

second commit 
```

Enter fullscreen mode Exit fullscreen mode

We can see that, _given a commit with a parent_, we can traverse **all commits recursively**, _through all their trees_ until we **get to the final blobs**.

A potential solution:  

```
$ git cat-file -p <first-commit-sha1>
$ git cat-file -p <first-commit-tree-sha1>
$ git cat-file -p <first-commit-parent-sha1>
$ git cat-file -p <parent-commit-sha1>
... 
```

Enter fullscreen mode Exit fullscreen mode

And so on. Well, _you got to the point._

### [](#log-for-the-rescue)🔵 Log for the rescue

The **porcelain** `git log` command solves that problem, by _traversing all commits, their parents and trees_, giving us a perspective of a **timeline of our work**.  

```
$ git log 5ea57

commit 5ea578a41333bae71527db537072534a199a0b67
Author: leandronsp <leandronsp@gmail.com>
Date:   Mon Mar 13 22:42:48 2023 -0300

    second commit

commit 505555f4f07d90ae14a0f2e67cba7f7b9af539ee
Author: leandronsp <leandronsp@gmail.com>
Date:   Mon Mar 13 22:35:14 2023 -0300

    my precious commit 
```

Enter fullscreen mode Exit fullscreen mode

> 🤯 OMG!
> 
> Git is a giant yet lightweight key-value graph database!

### [](#the-git-graph)🔵 The Git Graph

Within Git, we can _manipulate objects like pointers in graphs_.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--Mz9rlw1t--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xwj38c1zvimxfxpfzcq1.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--Mz9rlw1t--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xwj38c1zvimxfxpfzcq1.png)

*   **Blobs** are data/files snapshots
    
*   **Trees** are set of blobs or another tree
    
*   **Commits** reference trees and/or other commits, adding metadata
    

That's super nice and all. But using `sha1` in the `git log` command can be cumbersome.

What about **giving names to hashes**? Enter **References**.

* * *

[](#git-references)Git References
---------------------------------

References are located in the `.git/refs` folder:  

```
$ find .git/refs

.git/refs/
.git/refs/heads
.git/refs/tags 
```

Enter fullscreen mode Exit fullscreen mode

### [](#giving-names-to-commits)🔵 Giving names to commits

We can associate any commit hash with an arbitrary name located in the `.git/refs/heads`, for instance:  

```
echo 5ea578a41333bae71527db537072534a199a0b67 > .git/refs/heads/test 
```

Enter fullscreen mode Exit fullscreen mode

Now, let's issue `git log` using the new reference:  

```
$ git log test commit 5ea578a41333bae71527db537072534a199a0b67
Author: leandronsp <leandronsp@gmail.com>
Date:   Mon Mar 13 22:42:48 2023 -0300

    second commit

commit 505555f4f07d90ae14a0f2e67cba7f7b9af539ee
Author: leandronsp <leandronsp@gmail.com>
Date:   Mon Mar 13 22:35:14 2023 -0300

    my precious commit 
```

Enter fullscreen mode Exit fullscreen mode

Even better, Git provides the plumbing command `update-ref` so we can use it to update the association of a commit to a reference:  

```
$ git update-ref refs/heads/test 5ea578a41333bae71527db537072534a199a0b67 
```

Enter fullscreen mode Exit fullscreen mode

_Sounds familiar, uh?_ Yes, we are talking about **branches**.

### [](#branches)🔵 Branches

Branches are references that point to a specific commit.

As branches represent the `update-ref` command, the _commit hash can change_ at any time, that is, **a branch reference is mutable**.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--8MZwFZXi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a00yup5tew9hrmnr96xg.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--8MZwFZXi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a00yup5tew9hrmnr96xg.png)

For a moment, let's think about how a `git log` without arguments work:  

```
$ git log

fatal: your current branch 'main' does not have any commits yet 
```

Enter fullscreen mode Exit fullscreen mode

> 🤔 Hmmm...
> 
> How does Git _get to know that my current branch is the "main"_?

### [](#head)🔵 HEAD

The HEAD reference is located in `.git/HEAD`. It's a single file that points to a _head reference_ (branch):  

```
$ cat .git/HEAD

ref: refs/heads/main 
```

Enter fullscreen mode Exit fullscreen mode

Similarly, using a **porcelain** command:  

```
$ git branch
* main 
```

Enter fullscreen mode Exit fullscreen mode

Using the **plumbing** command `symbolic-ref`, we can manipulate to _which branch the HEAD points_:  

```
$ git symbolic-ref HEAD refs/heads/test

### Check the current branch
$ git branch
* test 
```

Enter fullscreen mode Exit fullscreen mode

Like `update-ref` on branches, we can update the HEAD using `symbolic-ref` at any time.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--QKHTuL_J--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/toezd1b9r5zlrezmwb27.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--QKHTuL_J--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/toezd1b9r5zlrezmwb27.png)

In the picture below, we'll change our HEAD from the **main** branch to the **fix** branch:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--DMIHMPgi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/01nz6085jlze4nqlza9d.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--DMIHMPgi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/01nz6085jlze4nqlza9d.png)

Without arguments, the `git log` command traverses the root commit that is referenced by the current branch (HEAD):  

```
$ git log

commit 5ea578a41333bae71527db537072534a199a0b67 (HEAD -> test)
Author: leandronsp <leandronsp@gmail.com>
Date:   Tue Mar 14 01:42:48 2023 -0300

    second commit

commit 505555f4f07d90ae14a0f2e67cba7f7b9af539ee
Author: leandronsp <leandronsp@gmail.com>
Date:   Tue Mar 14 01:35:14 2023 -0300

    my precious commit 
```

Enter fullscreen mode Exit fullscreen mode

Until now, we learned architecture _and main components_ in Git, along with the **plumbing commands**, which are more _low-level_ commands.

Time to associate all this knowledge with the **porcelain** commands we use daily.

* * *

[](#the-porcelain-commands)🍽️ The porcelain commands
-----------------------------------------------------

Git brings more _high-level_ commands that we can use with no need to manipulate objects and references directly.

Those commands are called **porcelain commands**.

### [](#git-add)🔵 git add

The `git add` command takes files in the **working directory** as arguments, _saves them as blobs_ into the database and adds _them to the index_.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--P4i-RMw1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tmfcfn23v1yvesjk9mzz.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--P4i-RMw1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tmfcfn23v1yvesjk9mzz.png)

In short, `git add`:

1.  runs `hash-object` for every file argument
    
2.  runs `update-index` for every file argument
    

### [](#git-commit)🔵 git commit

`git commit` takes a message as the argument, groups all the files previously added to the index and creates a **commit object**.

First, it runs `write-tree`:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--Qnbri7wE--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/imc0yzk6cn98hd3d09gf.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--Qnbri7wE--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/imc0yzk6cn98hd3d09gf.png)

Then, it runs `commit-tree`:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--pc32baZe--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tix2m469gnkqp2hes7k0.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--pc32baZe--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tix2m469gnkqp2hes7k0.png)  

```
$ git commit -m 'another commit'

[test b77b454] another commit
 1 file changed, 1 deletion(-)
 delete mode 100644 index.txt 
```

Enter fullscreen mode Exit fullscreen mode

* * *

[](#manipulating-pointers-in-git)🕸️ Manipulating pointers in Git
-----------------------------------------------------------------

The following **porcelain** commands are widely used, which manipulate the **Git references** under the hood.

Assuming we just cloned a project where the **HEAD** is pointing to the **main branch**, which points to the commit **C1**:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--37cQd96L--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v78l7749er21w7qzbjll.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--37cQd96L--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v78l7749er21w7qzbjll.png)

How can we **create another new branch** from the current HEAD and _move the HEAD to this new branch_?

### [](#git-checkout)🔵 git checkout

By using the `git checkout` with the `-b` option, Git will create a new branch from the current one (HEAD) and move the HEAD to this new branch.  

```
### HEAD
$ git branch
* main

### Creates a new branch "fix" using the same reference SHA-1
#### of the current HEAD
$ git checkout -b fix
Switched to a new branch 'fix'

### HEAD
$ git branch
* fix
main 
```

Enter fullscreen mode Exit fullscreen mode

Which **plumbing** command is responsible for moving the HEAD? Exactly, _symbolic-ref_.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--__odEO8O--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3yhvz46cb9mdj720wxlu.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--__odEO8O--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3yhvz46cb9mdj720wxlu.png)

Afterwards, we do some new work on the **fix branch** and then perform a `git commit`, which will add a new commit called **C3**:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--CnRSPeqo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6qmbh90fxn2o63wr7gpt.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--CnRSPeqo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6qmbh90fxn2o63wr7gpt.png)

By running `git checkout`, we can keep switching the HEAD across different branches:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--YJRBDhOK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b5i2b8mnp7zkrs9uguha.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--YJRBDhOK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b5i2b8mnp7zkrs9uguha.png)

Sometimes, we may want to **move the commit that a branch points to**.

We already know that the plumbing command `update-ref` does that:  

```
$ git update-ref refs/heads/fix 356c2 
```

Enter fullscreen mode Exit fullscreen mode

In _porcelain_ language, let me introduce you to the **git reset**.

### [](#git-reset)🔵 git reset

The `git reset` **porcelain** command runs **update-ref** internally, so we just need to perform:  

```
$ git reset 356c2 
```

Enter fullscreen mode Exit fullscreen mode

But how does Git know the branch to move? Well, _git reset_ **moves the branch that HEAD is pointing to**.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--JwrBx8a2--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ki3dzr73q7xb7kpz7704.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--JwrBx8a2--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ki3dzr73q7xb7kpz7704.png)

What about when there are _differences_ between revisions? By using `reset`, Git moves the pointer but **leaves all the differences in the stage area** (index).  

```
$ git reset b77b 
```

Enter fullscreen mode Exit fullscreen mode

Checking with `git status`:  

```
$ git status

On branch fix
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        another.html
        bye.html
        hello.html

nothing added to commit but untracked files present (use "git add" to track) 
```

Enter fullscreen mode Exit fullscreen mode

_The revision commit was changed in the_ **_fix branch_** _and all the differences were_ **_moved to the index_**.

Still, what should we do in case we want to **reset AND discard** all the differences? Just passing on the option `--hard`:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--WuasFyS3--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zhlcot7lpmuo57jmo5a1.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--WuasFyS3--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zhlcot7lpmuo57jmo5a1.png)

By using `git reset --hard`, any difference between revisions **will be discarded** and they _won't appear in the index_.

### [](#golden-tip-about-moving-a-branch)💡 Golden tip about moving a branch

In case we want to perform the _plumbing_ `update-ref` on another branch, there's no need to checkout the branch like needed in **git reset**.

We can perform the **porcelain** `git branch -f source target` instead:  

```
$ git branch -f main b77b 
```

Enter fullscreen mode Exit fullscreen mode

Under the hood, it performs a `git reset --hard` in the source branch. Let's check to which commit the **main branch** is pointing:  

```
$ git log main --pretty=oneline -n1
b77b454a9a507f839880879a895ac4f241177a28 (main) another commit 
```

Enter fullscreen mode Exit fullscreen mode

Also, we confirm that the **fix branch** is still pointing to the `369cd` commit:  

```
$ git log fix --pretty=oneline -n1
369cd96b1f1ef6fa7de1ff2ed12e15be979dcffa (HEAD -> fix, test) add files 
```

Enter fullscreen mode Exit fullscreen mode

We did a "git reset" _without moving the HEAD_!

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--70WY3M-1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7v7faopa858go3zkrygq.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--70WY3M-1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7v7faopa858go3zkrygq.png)

Not rare, _instead of moving a branch pointer_, we want to **apply a specific commit to the current branch**.

Meet **cherry-pick**.

### [](#git-cherrypick)🔵 git cherry-pick

With the **porcelain** `git cherry-pick`, we can apply an arbitrary commit to the current branch.

Take the following scenario:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--AkwMg0Yo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i0n7mldmwq8p3be6bzlf.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--AkwMg0Yo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i0n7mldmwq8p3be6bzlf.png)

*   **main** points to C3 - C2 - C1
    
*   **fix** points to C5 - C4 - C2 - C1
    
*   **HEAD** points to fix
    

In the _fix branch_, we are **missing the C3 commit**, which is being referenced by the _main branch_.

We can apply it by running `git cherry-pick C3`:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--7gaRMQMk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uvw4m2fom52ggxvn0o13.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--7gaRMQMk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uvw4m2fom52ggxvn0o13.png)

Note that:

*   the **C3** commit will be cloned into a new commit called **C3'**
    
*   this new commit will reference the C5 commit
    
*   **fix will move the pointer to C3'**
    
*   _HEAD keeps pointing to fix_
    

After applying changes, the graph will be represented as follows:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--pRZHxUdG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2gqsdauqv0alwrb4w4ol.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--pRZHxUdG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2gqsdauqv0alwrb4w4ol.png)

There's another way to move the pointer of a branch though. It consists of _applying an arbitrary commit of another branch_ but **merging the differences** if needed.

You're not wrong, we're talking about **git merge** here.

### [](#git-merge)🔵 git merge

Let's describe the following scenario:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s---Fl18baJ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/61lv3lp1bsi3o7ingaxq.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s---Fl18baJ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/61lv3lp1bsi3o7ingaxq.png)

*   main points to C3 - C2 - C1
    
*   fix points to C4 - C3 - C2 - C1
    
*   HEAD points to the main
    

We want to apply the fix branch into the current (main) branch, _a.k.a_ perform a **git merge fix**.

Please note that the **fix branch contains all commits belonging to the main branch** (C3 - C2 - C1), having only _one commit ahead_ of the main (C4).

In this case, the main branch will be "forwarded", pointing to the same commit as the fix branch.

This kind of merge is called **fast-forward**, as described in the image below:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--UIjnM-MC--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m6zh34jsvg46wly76qcq.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--UIjnM-MC--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m6zh34jsvg46wly76qcq.png)

### [](#when-fastforward-is-not-possible)When fast-forward is not possible

Sometimes, our tree structure current's state does not allow fast-forward. Take the scenario below:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--vMSkxLdM--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a4tsezqc59orz8a2o2nb.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--vMSkxLdM--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a4tsezqc59orz8a2o2nb.png)

That's when the merge branch - **fix branch** in the above example -, is _missing one or more commits_ from the current branch (main): **the C3 commit**.

As such, _fast-forward is not possible_.

However, for the merge to succeed, Git performs a technique called **Snapshotting**, composed of the following steps.

First, Git looks to the next **common parent** of the two branches, in this example, the **C2** commit.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--f3P_JmNK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1s3sfasy9216hhq8lzo5.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--f3P_JmNK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1s3sfasy9216hhq8lzo5.png)

Secondly, Git takes a **snapshot of the target** C3 commit branch:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--nazvycHV--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r5b784w4wejqx7xawynu.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--nazvycHV--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r5b784w4wejqx7xawynu.png)

Third, Git takes a **snapshot of the source** C5 commit branch:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--ZS-W9_nG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ouc15w3h73xwub9vuorn.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--ZS-W9_nG--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ouc15w3h73xwub9vuorn.png)

Lastly, Git automatically creates a commit merge (C6) and points it to two parents respectively: C3 (target) and C5 (source):

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--QE8XsYIN--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q030zd9jjl0nb17i871j.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--QE8XsYIN--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q030zd9jjl0nb17i871j.png)

Have you ever wondered why your Git tree displays some commits that were created **automatically**?

_Make no mistake_, this merge process is called the **three-way merge**!

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--AeZ6ACw3--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/05j8ntrxzax8ruydehl1.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--AeZ6ACw3--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/05j8ntrxzax8ruydehl1.png)

Next, let's explore _another merge technique_ where fast-**forward is not possible**, but instead of snapshotting and automatic commit merge, Git applies the differences **on top of the source branch**.

Yes, that's the **git rebase**.

### [](#git-rebase)🔵 git rebase

Consider the following image:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--0KyMPq_l--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n6u9hkcopd56anjjt6s7.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--0KyMPq_l--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n6u9hkcopd56anjjt6s7.png)

*   main points to C3 - C2 - C1
    
*   fix points to C5 - C4 - C2 - C1
    
*   HEAD points to fix
    

We want to **rebase** the main branch into the fix branch, by issuing `git rebase main`. But how does _git rebase_ work?

👉 **git reset**

First, Git performs a **git reset main**, where the fix branch will point to the same main branch pointer: _C3 - C2 - C1_.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--NBOuFkUM--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qq56zo5l7v6urjib67n0.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--NBOuFkUM--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qq56zo5l7v6urjib67n0.png)

At this moment, the C5 - C4 commits _have no references_.

👉 **git cherry-pick**

Second, Git performs a **git cherry-pick C5** into the current branch:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--WpXiilh---/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ucbxeoflzk8izrb2w621.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--WpXiilh---/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ucbxeoflzk8izrb2w621.png)

Note that, during a _cherry-pick process_, the cherry-picked commits are cloned, thus the final hash will change: **C5 - C4 becomes C5' - C4'**.

After cherry-pick, we may have the following scenario:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--fJ5ptBZO--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q65tu7b5h3zj1ma2b0gj.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--fJ5ptBZO--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q65tu7b5h3zj1ma2b0gj.png)

👉 **git reset again**

Lastly, Git will perform a **git reset C5'**, so the fix branch pointer will move _from C3 to C5'_.

The rebase process is finished.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--1JKATFwo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jjs73j725nbmjiau9nwc.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--1JKATFwo--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jjs73j725nbmjiau9nwc.png)

So far, we've been working with _local branches_, i.e on our machine. Time to learn how to work with **remote branches**, which are synced with _remote repositories on the internet_.

* * *

[](#remote-branches)🌐 Remote Branches
--------------------------------------

To work with remote branches, we have to _add a remote_ to our local repository, using the **porcelain** command `git remote`.  

```
$ git remote add origin git@github.com/myaccount/myrepo.git 
```

Enter fullscreen mode Exit fullscreen mode

Remotes are located in the `.git/refs/remotes` folder:  

```
$ find .git/refs
...
.git/refs/remotes/origin
.git/refs/remotes/origin/main 
```

Enter fullscreen mode Exit fullscreen mode

### [](#download-from-remote)🔵 Download from remote

How do we **synchronize** the remote branch with our local branch?

Git provides **two steps**:

👉 **git fetch**

By using the **porcelain** `git fetch origin main`, Git will download the remote branch and synchronize it with a new local branch called **origin/main**, also known as the **upstream branch**.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--ih7ruYBJ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9hii3cz4cjttk4pqnwzx.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--ih7ruYBJ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9hii3cz4cjttk4pqnwzx.png)

👉 **git merge**

After fetching and syncing the upstream branch, we can perform a `git merge origin/main` and because the upstream is ahead of our local branch, Git will safely apply a **fast-forward merge**.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--TN2Nqva4--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/86x06eev8316smdlz8i2.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--TN2Nqva4--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/86x06eev8316smdlz8i2.png)

However, **_fetch + merge_** _could be repetitive_, as we would synchronize local/remote branches multiple times a day.

But today is _our lucky day_, and Git provides the **git pull** porcelain command, that performs fetch + merge on our behalf.

👉 **git pull**

With `git pull`, Git will perform fetch (synchronize remote with the upstream branch), and then merge the upstream branch into the local branch.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--pQnPMLg1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ce7lb2f2bxpeoigfqbyr.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--pQnPMLg1--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ce7lb2f2bxpeoigfqbyr.png)

Okay, we've seen how to pull/download changes from the remote. On the other hand, how about sending local changes to remote?

### [](#upload-to-remote)🔵 Upload to remote

Git provides a **porcelain** command called `git push`:

👉 **git push**

Performing `git push origin main` will first upload the changes to remote:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--Q9BnAiNK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t6peyj4lcpdisu5310qx.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--Q9BnAiNK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t6peyj4lcpdisu5310qx.png)

Then, Git will merge the upstream `origin/main` with the local `main` branch:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--3UgB2wiI--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nafbkbjsqioiljaeaqkw.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--3UgB2wiI--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nafbkbjsqioiljaeaqkw.png)

At the end of the **push process**, we have the following image:

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--B-X8dK-F--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jcq5u4t60i15qm8127jd.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--B-X8dK-F--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jcq5u4t60i15qm8127jd.png)

Where:

*   The remote was updated (local changes pushed to the remote)
    
*   main points to C4
    
*   origin/main points to C4
    
*   HEAD points to the main
    

### [](#giving-immutable-names-to-commits)🔵 Giving immutable names to commits

Until now, we learned that **branches are simply mutable references to commits**, that's why we can move a branch pointer at any time.

However, Git also provides a way to _give immutable references_, which cannot have their pointers changed (unless you delete them and create them again).

Immutable references are helpful when we want to _label/mark commits_ that are ready for some production release, for example.

Yes, we are talking about **tags**.

👉 **git tag**

Using the porcelain `git tag` command, we can give names to commits but we cannot perform reset or any other command which would change the pointer.

[![](https://res.cloudinary.com/practicaldev/image/fetch/s--Ic_PNbdY--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pm62925zwulfu2vijjie.png)
](https://res.cloudinary.com/practicaldev/image/fetch/s--Ic_PNbdY--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pm62925zwulfu2vijjie.png)

It's quite useful for release versioning. Tags are located in the `.git/refs/tags` folder:  

```
$ find .git/refs

...
.git/refs/tags
.git/refs/tags/v1.0 
```

Enter fullscreen mode Exit fullscreen mode

_If we want to change the tag pointer, we must delete it and create another one with the same name._

* * *

[](#git-reflog)💡 Git reflog
----------------------------

Last but not least, there's a command called `git reflog` which keeps all the changes we've made in our local repository.  

```
$ git reflog

369cd96 (HEAD -> fix, test) HEAD@{0}: reset: moving to main
b77b454 (main) HEAD@{1}: reset: moving to b77b
369cd96 (HEAD -> fix, test) HEAD@{2}: checkout: moving from main to fix
369cd96 (HEAD -> fix, test) HEAD@{3}: checkout: moving from fix to main
369cd96 (HEAD -> fix, test) HEAD@{4}: checkout: moving from main to fix
369cd96 (HEAD -> fix, test) HEAD@{5}: checkout: moving from fix to main
369cd96 (HEAD -> fix, test) HEAD@{6}: checkout: moving from main to fix
369cd96 (HEAD -> fix, test) HEAD@{7}: checkout: moving from test to main
369cd96 (HEAD -> fix, test) HEAD@{8}: checkout: moving from main to test 369cd96 (HEAD -> fix, test) HEAD@{9}: checkout: moving from test to main
369cd96 (HEAD -> fix, test) HEAD@{10}: commit: add files
b77b454 (main) HEAD@{11}: commit: another commit
5ea578a HEAD@{12}: 
```

Enter fullscreen mode Exit fullscreen mode

**It's quite useful** if we want to **go back and forth** on the Git timeline. Along with _reset, cherry-pick and similar_, it's a powerful tool if we want to master Git.

* * *

[](#wrapping-up)Wrapping Up
---------------------------

What a long journey!

This article was a bit too long, but I could express the main topics I think are important to understand about Git.

I hope that, after reading this article, you should be more confident while using Git, resolving daily conflicts and painful situations during a merge/rebase process.

Follow me on [twitter](https://twitter.com/leandronsp) and check out my website blog [leandronsp.com](http://leandronsp.com/), where I also write some tech articles.

Cheers!