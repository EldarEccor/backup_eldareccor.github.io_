---
#layout: post
title: (Git) Bash to the rescue - moving misplaced test classes with 1 line
date: 2018-07-31
categories: ant
author: EldarEccor
---

After today I appreciate the Git bash on Windows even more:

Some test cases were located under *src/main/* for years. This goes against java conventions that dictate *src/test/* for all test classes and resources, but it **never** seemed to be a problem. The CI build was green, Eclipse could run the tests and everybody was happy...

The problem with conventions is, that other people rely on them - which resulted in all of those tests

* being packaged and delivered together with production code
* simply ignored by the build environments as well as the CI server ;-)
 
Long story short - there were a lot of files under *src/main/* with names ending in 'TestCase.java' that had to be moved to *src/test/* while keeping the directory structure (the java packages) intact. That's were the Git Bash came to the rescue, because sadly i am horrible at windows scripting and the documentation and examples are way better for bash:

We needed to
1. Locate all test cases files
2. Create the new folder structure under *src/test/java*
3. Move the files over
 
Locating files is done using 'find'. We have to specify where to look (the current directory) and what kind of matches we are looking for. We want to match by name and are looking for paths ending in *TestCase.java*. 'find' will output all matching paths line by line. 

```bash
find . -name *TestCase.java
./src/main/java/foo/bar/FooTestCase.java
./src/main/java/foo/bar/BarTestCase.java
...
```

For each of these files we want to create the same directory structure, but instead of *main* we want to use *test*. This is achieved via 'dirname' and basic String replacement because of our package structure (no other occurence of *main*). For now we ignore the necessary loop and just assume $FILE is our current file.

```bash
 mkdir -p $(dirname "${FILE/main/test}")
```

Finally we want to move the current file.

```bash
mv $FILE "${FILE/main/test}
```

Putting everything together we tell bash to execute our find command and loop other the created output.

```bash
for FILE in `find . -name *TestCase.java`;do mkdir -p $(dirname "${FILE/main/test}"); mv $FILE "${FILE/main/test}"; done
```

The for loop syntax is quite self explanatory if you try to format it a bit:

```bash
for FILE in `find . -name *TestCase.java`;
do 
	mkdir -p $(dirname "${FILE/main/test}"); 
	mv $FILE "${FILE/main/test}"; 
done
```