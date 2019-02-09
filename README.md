# Git-Sharing
Help on sharing GIT repositories between various machines (Windows, Linux, Mac, non-networked, etc)

## Bundle Sharing
Assume that you want to share a project stored in GIT between two collaborators without a shared repository. This can be done by using a "bundle" file instead of a shared repository. A "bundle" file can be created by any developer and then distributed to the other developers who can use it to update their own local repositories. All of the history and merging is retained just as in a shared repository. This section describes how to accomplish this using a simple example which can be performed on a single computer (for practice) or between multiple computers as needed.

### Practice on a single computer

Because each Git repository is self contained, bundle sharing can be practiced by creating two separate repositories on the same computer. Since we'll be pretending that they're on different machines, create the top level folder names as something like:

Machine_A
Machine_B

Inside each machine's folder, you can create an arbitrarily long subdirectory tree to get to the actual project files. That might looks something like this:

Machine_A/myfiles/coolproj
Machine_B/mystuff/test/coolproj

Let's assume that the project is started on Machine_A and has a Java source file (hello.java) and make file (makefile):

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
cd Machine_A/myfiles/coolproj
git init
git add hello.java
git add makefile
git add .gitattributes
git commit -m "Initial versions of hello.java and makefile in cool project." 
```
Now we're ready to create the initial bundle for the master branch of this repository. The bundle file can be created anywhere, but that location should remain fixed because it will be referenced from git as if it were a real repository. For this example, we will create it directly above our "coolproj" directory with a name of "coolproj.bundle". We do this with:

```
git bundle create ../coolproj.bundle master
```
Now the entire repository has been "bundled up" into a file at: Machine_A/myfiles/coolproj.bundle. This file could be shared with other developers by any method available (email, sneakernet, carrier pigeon, ...). In our example, we're just going to copy that file to:

Machine_B/mystuff/test/coolproj.bundle

Then change directory to that location (cd ../../../Machine_B/mystuff/test) and issue the "git clone" command:

```
git clone -b master coolproj.bundle coolproj
```
This will create a subdirectory named "coolproj" which is a duplicate of the original.
