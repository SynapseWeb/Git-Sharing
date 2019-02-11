# WORK IN PROGRESS ... DOESN'T FULLY WORK YET!!

# Git-Sharing
Help on sharing GIT repositories between various machines (Windows, Linux, Mac, non-networked, etc)

## Bundle Sharing
Suppose you want to share a project stored in GIT between two collaborators without a shared repository. This can be done using a "bundle" file instead of a shared repository. A "bundle" file can be created by any developer and then distributed to the other developers who can use it to update their own local repositories. All of the history and merging is retained just as in a shared repository. This section describes how to accomplish this using a simple example which can be performed on a single computer (for practice) or between multiple computers as needed.

### Practice on a single computer

Because each Git repository is self contained, bundle sharing can be practiced by creating two separate repositories on the same computer. Since we'll be pretending that they're on different machines, create two folders somewhere named something like:

* .../shared_path/Machine_A
* .../shared_path/Machine_B

Inside each machine's folder, you can create an arbitrarily long subdirectory tree to get to the actual project files. That might looks something like this:

* .../shared_path/Machine_A/myfiles/coolproj
* .../shared_path/Machine_B/mystuff/test/coolproj

Let's assume that the project is started on Machine_A (in the "coolproj" directory) and has a Java source file (hello.java) and make file (makefile):

```
public class hello {

  public static void main ( String args[] ) {
    System.out.println ( "Hello World!!" );
  }

}
```

```
all: hello.jar

hello.jar: hello.java makefile
	rm -f *.class
	javac hello.java
	jar -cfe hello.jar hello *.class hello.java
	rm -f *.class

run: hello.jar
	java -jar hello.jar

clean:
	rm -f *.jar
	rm -f *.class
```
To keep line endings straight right from the beginning, create a ".gitattributes" file:

```
# Set default behavior where git decides what to do:

* text=auto

# Define special cases (shows various examples):

*.txt    text
*.class  binary
*.jar    binary
*.vcproj text eol=crlf
*.sh     text eol=lf
```

Create the initial git repository (if one doesn't already exist):
```
Machine_A$ cd Machine_A/myfiles/coolproj
.../Machine_A/myfiles/coolproj$ git init
.../Machine_A/myfiles/coolproj$ git add hello.java
.../Machine_A/myfiles/coolproj$ git add makefile
.../Machine_A/myfiles/coolproj$ git add .gitattributes
.../Machine_A/myfiles/coolproj$ git commit -m "Initial versions of hello.java and makefile in cool project." 
```
You can check the log (history) with ```git log```. That should show that there's one version (one commit) in the repository.

Now we're ready to create the initial bundle of the master branch of this repository. The resulting bundle file can be created anywhere, but that location should remain fixed because it will be referenced from git as if it were a real repository. For this example, we will create it directly above our "coolproj" directory with a name of "coolproj.bundle". We do this with:

```
.../Machine_A/myfiles/coolproj$ git bundle create ../coolproj.bundle master
```
Now the master branch of the repository has been "bundled up" into a file at: Machine_A/myfiles/coolproj.bundle. This file could be shared with other developers by any method available (email, sneakernet, carrier pigeon, ...). In our example, we're just going to copy that file to .../Machine_B/mystuff/test/coolproj.bundle as shown here:

```
.../Machine_A/myfiles/coolproj$ cp ../coolproj.bundle ../../../Machine_B/mystuff/test/
```

At this point, you could switch to that new directory (via cd ../../../Machine_B/mystuff/test), but since we're simulating two machines, it's advisable to open another command shell window for commands on the "B" machine. That will save a lot of directory changing. In either case, you will need to have the current directory (on the "B" machine) set to ".../Machine_B/mystuff/test". Then issue the "git clone" command:
```
.../Machine_B/mystuff/test$ git clone -b master coolproj.bundle coolproj
```
This will create a subdirectory (if it doesn't already exist) named "coolproj" which is a duplicate of the original "master" branch. It will have all of the source files and all of the repository history of the original branch. You can work in that new directory to edit and build the code:
```
.../Machine_B/mystuff/test$ cd coolproj
.../Machine_B/mystuff/test/coolproj$ make run
```
This is where things get a little tricky. When git clones from a repository, it will generally create a "remote" from the location of the "upstream" source. Since the "clone" on "Machine_B" was done from a bundle file, the "remote" will be that file. Look inside the file:

```.../Machine_B/mystuff/test/coolproj/.git/config```

You should find something that looks like this:

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = / ... /Machine_B/mystuff/test/coolproj.bundle
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```
The "url" line will generally have the full path (shown here as "...") to your bundled file. As far as Git is concerned, that's the "upstream" location to use for pulling updates. Yet, it's just a file in your file system. You could change that long path to be just "../coolproj.bundle" to make it relative to the project. For now, we can leave the long path.

Now if you look at the ```".../Machine_A/myfiles/coolproj/.git/config"``` file in your original "Machine_A" repository, you'll only find this:

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
```
It has no "remote" option because it was not cloned from an existing repository. Remember that this (Machine A) version was created with ```"git init"``` when the project was started (without any cloning). But we can add a "remote" with the "git remote" command:

```
.../Machine_A/myfiles/coolproj$ git remote add origin ../coolproj.bundle
```

Now the file (".../Machine_A/myfiles/coolproj/.git/config") should look like this:

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = ../coolproj.bundle
	fetch = +refs/heads/*:refs/remotes/origin/*
```
Notice that it now has a "remote" section that tells it where to get updates. Now both "machines" are set up to pull their changes from their respective bundles. Let's test it.

In the "Machine_B" coolproj directory, add another "print" line to the Java program:

```
public class hello {

  public static void main ( String args[] ) {
    System.out.println ( "Hello World!!" );
    System.out.println ( "Last changed on Machine_B." );
  }

}
```
Commit the changes:

```
.../Machine_B/mystuff/test/coolproj$ git add -u
.../Machine_B/mystuff/test/coolproj$ git commit -m "Added a last changed on Machine_B line."
```
View the log to see the history with ```"git log"```. You should see both of the commit messages (the original done on "Machine_A" and the one you just added on "Machine_B"). By default, the most recent commit messages are shown at the top.

Now create a new bundle with that history to send back to Machine_A:

```
.../Machine_B/mystuff/test/coolproj$ git bundle create ../coolproj.bundle master
```
Just as before, that bundled file (```.../Machine_B/mystuff/test/coolproj.bundle```) can be transmitted by any means back to "Machine_A". In this case, just copy the file to replace the version in Machine_A's location:
```
.../Machine_B/mystuff/test/coolproj$ cp ../coolproj.bundle ../../../../Machine_A/myfiles/coolproj.bundle
```
Then in the ```.../Machine_A/myfiles/coolproj directory```, execute a ```"git pull origin master"``` and observe the changes:
```
.../Machine_A/myfiles/coolproj$ git pull origin master
```
```
From ../coolproj.bundle
 * branch            master     -> FETCH_HEAD
Updating #######..#######
Fast-forward
 hello.java | 1 +
 1 file changed, 1 insertion(+)
```
That shows the update that had been created in the Machine_B repository.  You can verify that with another ```"git log"``` command on Machine A. You should see the original commit along with the commit done on Machine B.

Now the project can be updated on Machine_A again. Add a third print line to the source code file:

```
public class hello {

  public static void main ( String args[] ) {
    System.out.println ( "Hello World!!" );
    System.out.println ( "Last changed on Machine_B." );
    System.out.println ( "Last changed on Machine_A." );
  }

}
```
Commit and bundle the same way that it had been done on Machine_B:
```
.../Machine_A/myfiles/coolproj$ git add -u
.../Machine_A/myfiles/coolproj$ git commit -m "Added a line written on Machine_A."
.../Machine_A/myfiles/coolproj$ git bundle create ../coolproj.bundle master
```
Check the log again (with ```"git log"```) to verify that there are now three commits in the repository on Machine A.

Then "send" the "coolproj.bundle" file back to Machine_B (by copying coolproj.bundle) where it can be pulled from and tested:

```
.../Machine_B/mystuff/test/coolproj$ cp ../../../../Machine_A/myfiles/coolproj.bundle ../coolproj.bundle
.../Machine_B/mystuff/test/coolproj$ git pull origin master
.../Machine_B/mystuff/test/coolproj$ git log
.../Machine_B/mystuff/test/coolproj$ make run
```

Both machines will have the same history and the same files. You can use these two repositories to experiment with changes that cause conflicts and require merging.

While this tutorial was conducted on the same machine, it could have been done on two different machines running different operating systems (as long as they both have "git"). The initial ".gitattributes" file sets the parameters needed to handle line ending differences between Windows and non-Windows computers.
