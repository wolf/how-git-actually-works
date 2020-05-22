# How Git Actually Works (presentation)

* Git is written in layers
    * GUI (for instance, looking at a diff in IntelliJ) is the outermost layer, closest to the user, supposedly more friendly than the command-line
    * 'Porcelain' (`git diff`) is the command-line stuff meant for humans
    * 'Plumbing' (`git diff-files`, `git diff-tree`, `git diff-index`, etc.) is all the machinery used to build Git from pieces, typically not for the end-user
* Where does Git store all its magic?
    * What's in a brand-new `.git` directory?
* Storage in the object database
    * How a file is stored ('blob')
        * Since addressed by hash, objects are unique: the same file can't be stored twice
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
    * `git gc` to clean up loose objects (as a porcelain user, you _never_ need to call this by hand)
