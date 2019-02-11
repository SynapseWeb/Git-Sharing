# WORK IN PROGRESS ... DOESN'T FULLY WORK YET!!

# Git-Sharing
Help on sharing GIT repositories between various machines (Windows, Linux, Mac, non-networked, etc)

## Bundle Sharing
Assume that you want to share a project stored in GIT between two collaborators without a shared repository. This can be done by using a "bundle" file instead of a shared repository. A "bundle" file can be created by any developer and then distributed to the other developers who can use it to update their own local repositories. All of the history and merging is retained just as in a shared repository. This section describes how to accomplish this using a simple example which can be performed on a single computer (for practice) or between multiple computers as needed.

### Practice on a single computer

Because each Git repository is self contained, bundle sharing can be practiced by creating two separate repositories on the same computer. Since we'll be pretending that they're on different machines, create two folders somewhere named something like:

* .../Machine_A
* .../Machine_B

Inside each machine's folder, you can create an arbitrarily long subdirectory tree to get to the actual project files. That might looks something like this:

* .../Machine_A/myfiles/coolproj
* .../Machine_B/mystuff/test/coolproj

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
Machine_A$ git init
Machine_A$ git add hello.java
Machine_A$ git add makefile
Machine_A$ git add .gitattributes
Machine_A$ git commit -m "Initial versions of hello.java and makefile in cool project." 
```
Now we're ready to create the initial bundle for the master branch of this repository. The resulting bundle file can be created anywhere, but that location should remain fixed because it will be referenced from git as if it were a real repository. For this example, we will create it directly above our "coolproj" directory with a name of "coolproj.bundle". We do this with:

```
Machine_A$ git bundle create ../coolproj.bundle master
```
Now the entire repository has been "bundled up" into a file at: Machine_A/myfiles/coolproj.bundle. This file could be shared with other developers by any method available (email, sneakernet, carrier pigeon, ...). In our example, we're just going to copy that file to:

.../Machine_B/mystuff/test/coolproj.bundle

Then change directory to that location (cd ../../../Machine_B/mystuff/test) and issue the "git clone" command:

```
Machine_B$ git clone -b master coolproj.bundle coolproj
```
This will create a subdirectory named "coolproj" which is a duplicate of the original. It will have all of the source files and all of the history of the original.

This is where things get a little tricky. When git clones from a repository, it will generally create a "remote" from the location of the "upstream" source. Since this clone was done from a bundle file, the "remote" will be that file. Look inside the file:

```Machine_B/mystuff/test/coolproj/.git/config```

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
The "url" line will generally have the full path (shown here as "...") to your bundled file. As far as Git is concerned, that's the "upstream" location to pull new commits from. Yet, it's just a file in your file system.

Now if you look at the "config" file in your original "Machine_A" repository, you'll only find this:

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
```
It has no remote option. But we can add it manually (and possibly other ways) so that the bundle is the same as any remote repository that we can pull from. Change the file "/Machine_A/myfiles/coolproj/.git/config" to be:

```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = /.../Machine_A/myfiles/coolproj.bundle
	fetch = +refs/heads/*:refs/remotes/origin/*
```
Remember to replace the "..." with your actual path.

Now both "machines" are set up to pull their changes from their respective bundles. Let's test it.

In the "Machine_B" area, add a line to the Java program:

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
Machine_B$ git add -u
Machine_B$ git commit -m "Added a last changed on Machine_B line."
```

Now create a new bundle to send back to Machine_A:

```
Machine_B$ git bundle create ../coolproj.bundle master
```
Just as before, that bundled file (Machine_B/mystuff/test/coolproj.bundle) can be transmitted by any means back to Machine_A. In this case, just copy the file to replace the version in Machine_A's location. Then in the Machine_A/myfiles/coolproj directory, execute a ```"git pull"``` and observe the changes:

```
From / ... /Machine_A/myfiles/coolproj.bundle
 * branch            master     -> FETCH_HEAD
Updating #######..#######
Fast-forward
 hello.java | 1 +
 1 file changed, 1 insertion(+)
```
That shows the update. Now the project can be updated on Machine_A again:

```
public class hello {

  public static void main ( String args[] ) {
    System.out.println ( "Hello World!!" );
    System.out.println ( "Last changed on Machine_B." );
    System.out.println ( "Last changed on Machine_A." );
  }

}
```
Commit and bundle as was done on Machine_B:
```
Machine_A$ git add -u
Machine_A$ git commit -m "Added a line written on Machine_A."
Machine_A$ git bundle create ../coolproj.bundle master
```
Then send the "coolproj.bundle" file back to Machine_B where it can be pulled from and tested:

```
Machine_B$ git pull origin master
Machine_B$ make run
```

That's the same process used to share a git repository between machines that are actually different.
