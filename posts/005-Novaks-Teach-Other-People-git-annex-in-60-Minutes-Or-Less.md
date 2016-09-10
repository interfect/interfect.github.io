# Novak's Teach Other People `git annex` in 60 Minutes Or Less

I've been using a piece of software called [Git Annex](https://git-annex.branchable.com) for a while now, to manage my many large files and to schlep them around for me. Despite following along with the [official tutorial](https://git-annex.branchable.com/walkthrough/), reading a bunch of other stuff on the [official Git Annex Branchable site](https://git-annex.branchable.com/), and even using the software for a couple of weeks, it took me a long time to really understand WTF the thing was doing for me. Now I do, and I'm going to explain it to you, in the 60 or so minutes I have to write it out.

Git Annex is a tool for managing a distributed collection of files, using git to manage the directory layout and location tracking information, and its own wizardry to actually schlep the contents around. You are presented with a Git repository full of symlinks to your files; if you have a local copy of the file, the symlink points to it, while if the content of the file isn't present locally, the symlink is broken. The actual file data is stored inside the `.git` directory, in files named by the hash of the file contents (which handles deduplication).

Generally, you will have a collection of Git repositories, each of which has remotes pointing to one or more of the others. Each repository is identified by a GUID, and any one repository knows about the existence of all the others, even those that it doesn't have a remote configured for and can't push to or pull from. These repositories conceptually form a single "network" or "annex"--the Git Annex docs don't really have a name for it, but basically all the repository clones have the same list of files and the same information about which repositories have which files' data in them.

Once you get yourself a Git repository (with a `git init` or `git clone`), and assign it a GUID (with a `git annex init`), you can start doing Annex-y things. To add a file to Git Annex, just drop it somewhhere inside the repository and then use `git annex add` on it:

```
# Make your Git repo
mkdir annex
cd annex
git init

# Make it a Git Annex repo
git annex init

echo "Here is a file" > file.txt
git annex add file.txt
```

Git Annex will checksum the file, move the actual contents over to somewhere inside `.git`, and replace it with a symlink. (If you're on Windows, or working on an NTFS filesystem on Linux, the file will instead sytay right where it is; Git Annex will operate in "Direct Mode", due to its low opinion of your filesystem.)

After you add your files, you can use the all-purpose `git annex sync` command, which will commit your new file and also synchronize any changes in repository layout and file content location tracking with all of the remotes it knows about.

```
git annex sync
```

Now you can make another repository:

```
# Make another Git repo
cd ..
git clone annex annex2
cd annex2

# Set up Git Annex in it
git annex init "other repository"
```

It helps to assign the repository a description (in this case, "other repository"). If you don't have a Git remote in your local repository pointing to a particular repository, Git annex will identify that other repository by its description, or, if the description isn't set, will just throw an ugly GUID at you.

At this point, you should have a broken `file.txt` symlink in your `annex2` directory. (If you're in Direct Mode, you instead have a normal file with the path that the symlink would point to as its contents.) That's because the contents for the file haven't been downloaded to your local repo.

You can ask Git Annex where the file actually is:

```
git annex whereis file.txt
```

You will get a report like this:

```
whereis file.txt (1 copy)
        350396c3-d9c0-4c68-b58e-4f4331cf84a3 -- user@host:~/annex [origin]
ok
```

You get the GUID and description (in this case auto-generated), as well as the remote name you can use to reach the repo from here, if any, in brackets.

You can download the file contents with `git annex get`:

```
git annex get file.txt
```

Git Annex will download the file, and then your symlink will be fixed and you will be able to see its contents.

After getting files, don't forget to use `git annex sync` to synchronize your file location tracking data back to the other repository. Since `annex2` has `annex` as its `origin` remote, but `annex` doesn't have a remote pointing to `annex2`, you have to sync in the `annex2` repo:

```
git annex sync
```

You can then go back to the `annex` repo, and see what it knows about where your file is:

```
cd ../annex
git annex whereis file.txt
```

It will know that there are two copies, and that it has one of them, but the other one is in "other repository". The "other repository" repo isn't pointed to by any remote, so there's no way to call that repo up and upload or download files.

```
whereis file.txt (2 copies)
        350396c3-d9c0-4c68-b58e-4f4331cf84a3 -- user@host:~/annex [here]
        44481925-b630-489f-ab11-c25922310dcc -- other repository
ok
```

If you try to get rid of your local copy of the file's data (with `git annex drop`), Git Annex will complain. By default it's configured to check with at least one other repo that has a copy of file data before deleting its local copy, to make sure it's still there. This is to handle the case where you deleted the file data in the other repo as well, but haven't synchronized location tracking information yet.

```
git annex drop file.txt
```

```
drop file.txt (unsafe)
  Could only verify the existence of 0 out of 1 necessary copies

  Try making some of these repositories available:
        44481925-b630-489f-ab11-c25922310dcc -- other repository

  (Use --force to override this check, or adjust numcopies.)
failed
git-annex: drop: 1 failed
```

You can add a remote with standard Git commands:

```
git remote add other-repo ../annex2
```

Then your drop will succeed, because Git Annex will dial up the remote and verify that it actually has a copy of the file's contents:

```
git annex drop file.txt
```

```
drop file.txt ok
(recording state in git...)
```

At this point I'm going to assume that you get the basic idea, and I'm rapidly running out of my 60 minutes. So here's a cheat sheet of all the tasks you might want to do:

* **Add a file**: `git annex add <filename>`
* **Delete file contents from local repo**: `git annex drop <filename>`
* **Download file contents**: `git annex get <filename>`
* **Upload file contents**: `git annex copy --to=<remote> <filename>`
* **Move your copy of a file's contents elsewhere**: `git annex move --to=<remote> <filename>`
* **Find where a file is**: `git annex whereis <filename>`
* **Synchronize file list and location tracking**: `git annex sync`, or `git annex sync <remote>` to sync with a particular remote

All of these commands will operate recursively when given directories instead of files. The commands that move content arround will take `--to` and `--from` options to specify what remote you want the content to be fetched from or sent to (though you can't use both at once). All of these commands (except for `git annex add`) also accept [matching options](https://git-annex.branchable.com/git-annex-matching-options/) in addition to or even instead of a filename, to specify what files to work on. You can say things like "download all files we don't have that are smaller than 3 megabytes":

```
git annex get --not --in=here --and --smallerthan=3mb
```

Or "copy all the files that are here and that aren't on my work computer to my flash drive" (assuming you have remotes configured for both):

```
git annex copy --to=flashdrive --in=here --and --not --in=work

# Don't forget to save location tracking info. Especially if flashdrive is a "special" remote and not a real Git repo.
git annex sync
```

Presumably you would then bring the drive to work and unload the files there:

```
git annex move --from=flashdrive
```

There's also a `git annex find` that just prints out the names of whatever matches its matching options.

To me, these matching options are what really make Git Annex useful, and it's a shame the official tutorial doesn't really cover them. You can tell the computer exactly what content you want to send where, and it will do it for you.

There are more useful things in Git Annex, like [special remotes](https://git-annex.branchable.com/special_remotes/) that let you store file contents in places where you don't want a full Git checkout (including encrypted in the cloud), and [preferred content expressions](https://git-annex.branchable.com/preferred_content/) and [repository groups](https://git-annex.branchable.com/preferred_content/standard_groups/) which together let you automate the schleping about of different kinds of content. But all that stuff starts to get complicated fast, and the basic copying and matching system is really what you need to know to get yourself out of whatever jams you get into with the more complicated systems. Also, I'm starting to run out of my 60 minutes, so I'll have to end it here.

Stay tuned for more on how I use Git Annex to back stuff up in the cloud in a future post.

