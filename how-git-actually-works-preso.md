# How Git Actually Works (presentation)

* Git is written in layers
    * 'Plumbing' (e.g., `git diff-files`, `git diff-tree`, `git diff-index`, `git hash-object`, `git cat-file`, et al) is all the machinery used to build Git from pieces, typically not for the end-user
    * 'Porcelain' (e.g., `git diff`, `git log`) is the command-line stuff meant for humans
    * GUI (for instance, looking at a diff in IntelliJ) is another layer of porcelain, closest to the user, supposedly more friendly than the command-line
* Where does Git store all its magic?
    * What's in a brand new `.git` directory
        * `git init my-new-repo`
        * `cd my-new-repo ; tree .git`
            * HEAD and refs/ I'll explain later
            * config you already know, but let me demonstrate with `git config user.email Wolf@learninga-z.com`
            * description is mostly vestigal
            * hooks contain scripts that execute at various points in git actions, like just before a commit for instance
            * info/excludes contains global gitignore rules; we don't use it currently
            * Pay particular attention to the `objects` directory: this is the important part---the "object database"
* Storage in the object database
    * Let's put an object in the database
        * `echo "Hello, LAZ!" | wc -c` the string is 12 characters long, counting the newline echo automatically adds.  This will be important in a minute
        * `echo "Hello, LAZ!" | git has-object --stdin -w`.  `--stdin` means take the input from stdin instead of looking for a filename.  `-w` means write it to the object database.
        * Git spits out a 40-character (20-byte) object-id.
            * Where did it come from?
            * Where did the data actually go?
                * `tree .git/objects`
    * How a file is stored ('blob')
        * in IPython
            * `compressed = open("c775e4d0742779111bcef256e0d9fe384103f2", "rb").read()`
            * `decompressed = zlib.decompress(compressed)`
            * `object_id = hashlib.sha1(decompressed).hexdigest()`
        * (Talk a bit about hashes.) Since addressed by hash, objects are unique: the same file can't be stored twice
    * How a directory is stored ('tree')
    * How a commit is stored ('commit')
    * The only other thing that goes in the object database: annotated tags ('tag')
* How commits connect everything together
    * tree (exactly one per commit)
    * parent(s) (zero or more depending on the kind of commit.)
    * what is the difference between two commits, how is that stored?
    * how many objects are created when you change one file in your commit?
    * how to know when a file is renamed
* what is the index/staging-area
    * `git ls-files --stage` (look familiar?  compare to a tree.)
* rescuing changes/commits
    * `git fsck --lost-found` to rescue lost files/commits
* so, do i have like a zillion loose objects then?
    * what is a pack-file?
    * `git gc` to clean up loose objects (as a porcelain user, you **never** need to call this by hand)
* What are "refs"?
    * a ref is just an object-id, typically of a commit
    * now what about that HEAD file in .git
        * almost always the name of the branch you currently have checked out
        * is a commit-id, instead, when you have a detached head
    * what lives in .git/refs
        * every local branch (in heads)
        * every remote branch (in remotes)
        * every tag (in tags)
