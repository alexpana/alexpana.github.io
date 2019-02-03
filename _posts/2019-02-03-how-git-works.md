---
layout: post
title:  "How git works under the hood"
---

At its core, `git` is a very simple content tracker designed by Linus Torvalds from the perspective of a [file system developer](https://marc.info/?l=linux-kernel&m=111314792424707). Git is also notoriously very hard to learn. I blame this on tutorials that teach how to use git using recipes instead of explaining how git is designed and building on top of that knowledge. If you've used git before but still find some concepts confusing then hopefully this article will clear some things up.

## Git objects

Git stores all the content of the repository (commits and files) as objects under the `.git/objects` folder. These objects are compressed text files that can be inspected anytime by anyone. No diffs, no magic. There are four types of objects: commits, blobs, trees, and tags. All of them are stored in the same way, but they contain different content and serve different purposes.

Object files are named using a hash (more on this later):

    $ find .git/objects -type f
    .git/objects/e2/44aa3ab918c69e456a81950af95c1acd559c75
    .git/objects/1a/44f1e2007856360206bd9e5870242faa35c598
    .git/objects/38/fabd36fc6cbf02446b8034fe053f6276b16a8e

Notice they are split into subdirectories named using the first two characters of the hash: `e2`, `1a`, `38`. The object's hash includes the directory name, like `e244aa3ab918c69e456a81950af95c1acd559c75`. This is done for multiple reasons, including:

1. Some file systems impose a limit on the number of children a folder can have, and some projects (e.g. the Linux kernel) get close to that limit.
2. Most file systems store folder children using an array or linked list. Accessing a file usually requires iterating this list, meaning performance will degrade when the number of children increases.

You can inspect the type and content of any object using:

    $ git cat-file -p <SHA1>

And you can find the type of an object by using:

    $ git cat-file -t <SHA1>

### Blobs

When you commit a file to git, a SHA1 hash is generated from its content. This hash uniquely [1] identifies this file not by name or path inside the repo, but by content. The file is then compressed using `zlib` and saved under `.git/objects/<SHA1>`. The result is called a blob (Binary Large OBject) object. It's now binary because it was compressed. The name of the file isn't stored here.

![Git file object](/assets/git_file_object.png)

Two files with identical content will have identical hashes. As a result, git will only store the file once. This makes sense because storing two identical files would be wasteful. This holds true regardless of what commit the file is part of.

To generate the SHA1 hash for a file, you can use:

    $ git hash-object -t blob <filename>

When needed, git can generate diffs on the fly between any two blobs using `git diff`.

### Directories

Since files in your repository are structured in folders, git has to keep track of that. All directories, starting with the root are stored in objects of type `tree`. If you were to read the contents of a tree object using `git cat-file` you would find something like:

    100644 blob 557db03de...    README.md
    040000 tree f67bc4132...    src

These are effectively the contents of the folder whose hash we just inspected. For each child file and folder git stores the type, mode, hash, and name much like a real file system would. This solves the problem of remembering what name a file has, and where it's located within the folder structure.

Any time something changes within a folder that would affect the content of its tree object: permissions, names or the content of children, git generates another tree object for that folder. Since the hash of this new tree object is changed, its parent needs to be updated with the new hash as well. This means that git will generate new tree objects for all folders up to the root. Any unaffected tree objects are reused.

![Git tree object](/assets/git_tree_object.png)

Notice how a modified file generates a new blob which in turn requires new trees to be generated recursively. Unmodified files (and trees) are reused.

It's trivial to see that reading the tree objects and blob objects recursively, one can recreate the original files and file structure.

Just like with files, if the change happens to be a revert, and git already has the same files in the same folders already in the objects folder, nothing new will be created and the objects are reused.

Git doesn't explicitly track file movement or renames. Moving a file without changing its contents only affects the directory objects since the content and the hash remain the same. When git sees a remove and an add with the same file hash, it can easily and accurately determine it's a moved file. Moving and changing a file isn't trivially recognized by git and only works heuristically if the change is below a certain ratio.

### Commits

This is the first object type that's actually visible by the user. Creating a new commit with `git commit` creates a new commit object. It's easy to find the hash of the commit in the `git log`. Reading the contents of this object reveals something like:

    tree e244aa3ab918c69e456a81950af95c1acd559c75
    parent 38fabd36fc6cbf02446b8034fe053f6276b16a8e
    author Alexandru Pana <alex@test.com> 1549120626 +0200
    committer Alexandru Pana <alex@test.com> 1549120626 +0200
    
    Initial commit

The commit object contains: 

1. A reference to the root tree object.
2. A reference to the parent commit. Merge commits will contain multiple parent lines.
3. The author (the person who originally created the commit).
4. The committer (occasionally a different person that merged the commit) and the commit message.

When you checkout a branch git finds the commit that branch points to, finds the tree object the branch references and then expands that tree to update the working directory.

There are a few interesting and very important concepts related to commits:

1. **Commits are immutable**. Once you committed something, it's there forever (more on this later). Some git commands allow you to 'change the history'. This is technically inaccurate. Commands like rebasing or amending create new commits leaving the old ones in place. You may still access your previous commits if you wish. What actually changes is the branch.
2. **Branches are just references to commits**. Branches are plaintext files stored in files under `.git/refs` that contain the SHA1 of the commit they point to.
3. The currently active branch is stored in a special ref called `HEAD`. Checking out branches moves the HEAD from one branch to another. The history of where the HEAD has been is stored in something called the `reflog`. You can, in rare situations, checkout a commit instead of a branch. When you do this git will issue a warning that you are in a "detached HEAD state". You can see the reflog using:

        $ git reflog

4. Commits that cannot be reached from any branch, tag, the reflog and some other sources are considered inaccessible. Occasionally you can run `git gc` to remove unreachable commits, just like any other garbage collector. Any unreachable object will also be removed.
5. Stash entries are also stored as commit objects.

### Tags

Tags are used to save a reference to a commit using a name (like a branch) by storing an immutable object (unlike a branch). Tags can have a description and can be PGP signed. You can find tags under `.git/refs/tags` (like a branch). Reading the contents of a tag reveals something like:

    object 7637915b4aa542a86a3b2bc2f56b04f10f73df43
    type commit
    tag release_0.1
    tagger Alexandru Pana <alex@test.com> 1549138132 +0200
    
    We're going to production!

The `object` and `type` lines usually reference a commit object to which the tag is pointing.

### Final thoughts

Git is surprisingly simple at its core. A few simple concepts are used to create all the complexity of committing, branching, merging and sharing code. Remember that everything you do with git eventually boils down to creating and referencing objects. Also, remember that every commit is immutable and guaranteed to still be around even after you've apparently lost it. The next time you encounter something confusing, try to figure out what git does behind the scenes.

### Further reading

* [Learn git branching](https://learngitbranching.js.org/) interactive git sandbox by GitHub that teaches essential git skills using tutorials and visualization.
* [Git Internals](https://github.com/pluralsight/git-internals-pdf) by Scott Chacon, it's short, pretty and filled with descriptive graphics.
* [Git Pro Book](https://git-scm.com/book/en/v2) by Scott Chacon and Ben Straub, longer and more comprehensive book on git internals.