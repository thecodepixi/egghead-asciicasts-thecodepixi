We've created a directory called `cool-project`, and we've moved ourselves into the directory. If we take a look inside, there's nothing here yet. What we want to do is we want to pull down a remote Git repo and place it inside of this directory.

The remote repository that we want on our machine is located on GitHub, [https://github.com/trevordmiller/utility-functions](https://github.com/trevordmiller/utility-functions), and after clicking the `Clone or download` button, we can see that there is this URL of where the remote is located, `https://github.com/trevordmiller/example-utility-functions.git`. We're going to copy this. Now back in our Command Line, we can type `git clone`, and then paste in the URL that we copied from GitHub, `git clone https://github.com/trevordmiller/example-utility-functions.git`.

Now if we look at what's inside this folder, we can see that we have another folder called `utility-functions`, which is our cloned repo. If we move into that folder, and we output what's inside of it, we can see that it has the `.git` directory.

![.git file in directory](../images/misc-practical-git-copy-remote-repos-to-local-machines-with-git-clone-git-in-file.png)

We'd know that this is our Git repo from the remote. We also see that this folder contains a couple of files that contain our code. If we look back at our GitHub page, we can see that it's the same files that are located in the remote repository.