# How Git Actually Works


## Preamble

* Git is written in layers
    * 'Plumbing' is all the machinery used to build Git from pieces, typically not for the end-user.  Examples:
        * `git help hash-object`
        * `git help cat-file`
        * `git help ls-files`
        * `git help count-objects`
        * et al
    * 'Porcelain' is the command-line stuff meant for humans
        * `git help diff`
        * `git help log`
    * A GUI is just another layer of porcelain
* Talk a bit about hashes.
    * What is a hash?
        * [What is SHA1?](https://en.wikipedia.org/wiki/SHA-1)
    * What does Git use hashes for?
        * File integrity: the hashes are stored, and the saved object must re-hash to that same value or there is a problem
        * [Why does git use a cryptographic hash function?](https://stackoverflow.com/a/28792805/908269)
* Talk a bit about object databases
    * At its core, Git is a tool for taking "snapshots" of your working directory
    * Git stores these snapshots in an object database
    * `git init my-new-repo ; cd my-new-repo`
    * `tree .git` --- examine the .git/objects directory
* Summary
    * Git is written in layers
        * Plumbing is the parts that make it work
        * Porcelain is the user interface
    * Git uses SHA1 for file integrity, but it's moving to SHA256
    * Git stores all its snapshots in an object database (strictly key-value, no SQL) that lives in .git/objects


## How the object database actually works

* Let's put a snapshot of a file in the object database
    * `echo "Hello, LAZ!" > file1`
    * `wc -c file1` -- the string is 12 characters long, counting the newline echo automatically adds.  This will be important in a minute
    * `git hash-object -w file1`.  `-w` means write it to the object database.
    * Git spits out a 40-character (20-byte) object-id.
        * Where did it come from?
        * Where did the data actually go?
            * `tree .git/objects`
* Let's look inside the snapshot ('blob')
    * `pushd .git/objects/27`
    * In IPython
        * `import hashlib, zlib`
        * `compressed = open("c775e4d0742779111bcef256e0d9fe384103f2", "rb").read()`
        * `decompressed = zlib.decompress(compressed)`
        * `object_id = hashlib.sha1(decompressed).hexdigest()`
    * `popd`
    * Since addressed by hash, objects are unique: the same file can't be stored twice
    * `cp file1 file2 ; git hash-object -w file2` --- is there a new object?  No.
    * `echo "Hello, Laz!" | git hash-object --stdin -w` --- note the hash is totally different and there _is_ a new object
    * `tree .git/objects` --- note: only one object in the database
    * `git cat-file -t 27c7` --- `cat-file` can show the type of an object in the database
    * `git cat-file -p 27c7` --- `cat-file` can show the contents of the object as well
* Let's put a snapshot of a directory into the object database ('tree')
    * Let's make a commit so we can see how Git snapshots our top-level directory
    * `git add .`
    * `git commit -m "Initial commit"`
    * `tree .git/objects` --- three objects now, the blob, the commit, and something for the directory
    * `git cat-file -t 21ad` --- ah!  so a directory is a 'tree' object
    * `git cat-file -p 21ad` --- that just assigns names and modes to other objects
    * Note the tree _doesn't_ contain a name for itself, or a time-stamp
* With just these two kinds of objects, Git is like a bare-bones file-system.  In fact, it's an instance of a Content-Addressable Storage (CAS) system (more about this)
    * What's good about CAS's
        * They're fast --- you go straight to the object, no paths to worry about
        * They're efficient --- no object is stored more than once
    * [Content-addressable storage](https://en.wikipedia.org/wiki/Content-addressable_storage)
* How a commit is stored ('commit')
    * `git log --oneline` --- to get the commit-id
    * `git cat-file -t <the-commit-id>`
    * `git cat-file -p <the-commit-id>`
* The only other thing that goes in the object database: annotated tags ('tag')
* Summary
    * Git stores files (as 'blob's) and directories (as 'tree's) in the object database
        * They get a short header with 'blob', or 'tree' as their start and then the length of the data (as text) and a null byte
        * Within the object database, the hash of this object is the key, the value is this object compressed
    * That makes Git a bare-bones file-system, also known as a Content-addressable storage system
    * Commits ('commit') and tags ('tag') are also stored in the object database


## How commits connect everything together

* Let's make a second commit
    * `mkdir sub-directory ; git mv file2 sub-directory `
    * `git ls-files --stage`
    * `git commit` (make a big commit message, multi-line)
    * `git log --oneline` --- to get the commit-id
    * `git cat-file -p <the-new-commit-id>`
* Tree (exactly one goes in the commit object)
* Parent(s) (zero or more depending on the kind of commit.)
* How many objects are created when you change one file in your commit? (assuming it doesn't recreate an earlier state)
* What is the difference between two commits, how is that stored?
    * Well then, how does Git find all these differences and do blame etc.?
    * That's right, Git does like 1000x more work than you thought it did
* How to know when a file is renamed
* Summary
    * Commits are connected because each contains the commit-id(s) of its parent(s)
    * Commits contain the object id of the root tree
    * New objects are created when you change a leaf file: one for the file, and one for each directory up to and including the root.
    * No diffs or change-lists are stored.  Everything is calculated dynamically.
    * Copied/moved files are determined at the time you look.  That's not stored either.


## what is the index/staging-area

* Surprise!  It's another kind of database!
* `git ls-files --stage` (look familiar?  compare to a tree.)
    * It's _all_ blobs, with full paths (Note: this is why Git can't store empty directories) (Note the paths are in alphabetical order)
* The index is a commit ready to be "stamped", all it needs are its trees to be created and a commit object
* Summary
    * The index (or staging area) is another kind of database, listing all the full paths files currently in the entire project
    * It is a commit ready to be stamped into existence
    * It starts filled in with the current state of a commit.  `git add` fills in different object-ids


## rescuing changes/commits

* `git fsck --lost-found` to rescue lost files/commits
* `git reflog` will also find lost commits, but that's outside the scope of this talk
* Summary
    * You can check your repo for consistency and find dangling objects
    * One way is with `git fsck`; another way to find lost commits is with `git reflog`


## What are "refs"?

* A ref is just an object-id, typically of a commit
* Now what about that HEAD file in .git
    * Almost always the name of the branch you currently have checked out
    * Is a commit-id, instead, when you have a detached head
* What lives in .git/refs
    * Every local branch (in heads) --- because the tip of a branch is simply a ref
    * Every remote branch (in remotes)
    * Every tag (in tags)
* Summary
    * A "ref" is just an object-id, usually to a commit somewhere
    * Git manages branches and checkouts by keeping track of refs
    * A branch is just a file named for the branch, containing a ref to the current tip
    * HEAD is just a file containing the name of the current branch


## So, do i have like a zillion loose objects then?  Git sounds pretty inefficient

* `cd /c/dev/code/git_repo`
* `git count-objects` --- that's not enough!  where are the rest?
* `git count-objects -v` --- note the in-pack objects count
* What is a pack-file?
* `git gc` to clean up loose objects (as a porcelain user, you **never** need to call this by hand)
* Summary
    * Git does not store a million loose objects
    * Instead, they're bundled into efficient "packs" and deltified for even better space savings
    * These packs are built by the garbage collector which also eventually discards old unreferenced objects
