# How Git Actually Works (presentation)

* Git is written in layers
    * 'Plumbing' is all the machinery used to build Git from pieces, typically not for the end-user.  Examples:
        * `git hash-object`
        * `git cat-file`
        * et al
    * 'Porcelain' is the command-line stuff meant for humans
        * `git diff`
        * `git log`
    * A GUI is yet another layer of porcelain
* Where does Git store all its magic?
    * What's in a brand new `.git` directory
        * `git init my-new-repo`
            * Pay particular attention to the `objects` directory: this is the important part---the "object database"
* Storage in the object database
    * Let's put an object in the database
        * `echo "Hello, LAZ!" > file1`
        * `wc -c file1` -- the string is 12 characters long, counting the newline echo automatically adds.  This will be important in a minute
        * `git hash-object -w file1`.  `-w` means write it to the object database.
        * Git spits out a 40-character (20-byte) object-id.
            * Where did it come from?
            * Where did the data actually go?
                * `tree .git/objects`
    * How a file is stored ('blob')
        * `pushd .git/objects/27`
        * in IPython
            * `compressed = open("c775e4d0742779111bcef256e0d9fe384103f2", "rb").read()`
            * `decompressed = zlib.decompress(compressed)`
            * `object_id = hashlib.sha1(decompressed).hexdigest()`
        * `popd`
        * Talk a bit about hashes.
            * Since addressed by hash, objects are unique: the same file can't be stored twice
        * `cp file1 file2 ; git hash-object -w file2` -- is there a new object?  No.
        * `git cat-file -t 27c7` --- `cat-file` can show the type of an object in the database
        * `git cat-file -p 27c7` --- `cat-file` can show the contents of the object as well
        * `tree .git/objects` --- note: only one object in the database
    * How a directory is stored ('tree')
        * Let's make a commit so we can see how Git snapshots our top-level directory
        * `git commit -am "Initial commit"`
        * `tree .git/objects` --- three objects now, the blob, the commit, and something for the directory
        * `git cat-file -t 21ad` --- ah!  so a directory is a 'tree' object
        * `git cat-file -p 21ad` --- that just assigns names and modes to other objects
        * note the tree _doesn't_ contain a name for itself
    * With just these two kinds of objects, Git is like a bare-bones file-system.  In fact it's an instance of a Content-Addressable Storage (CAS) system
        * What's good about them
            * They're fast --- you go straight to the object, no paths to worry about
            * They're efficient --- no object is stored more than once
    * How a commit is stored ('commit')
    * The only other thing that goes in the object database: annotated tags ('tag')
* How commits connect everything together
    * tree (exactly one per commit)
    * parent(s) (zero or more depending on the kind of commit.)
    * what is the difference between two commits, how is that stored?
    * how many objects are created when you change one file in your commit?
    * how to know when a file is renamed
* what is the index/staging-area
    * Surprise!  It's another database!
    * `git ls-files --stage` (look familiar?  compare to a tree.)
    * the index is a commit ready to be "stamped", all it needs are its trees to be created and a commit object
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
