## Converting [CVS][cvs] & [Subversion (SVN)][svn] repositories to [Git][git]


<a name="1"></a>
---
## 1: Contents

+ [Requirements](#requirements)
+ [Subversion to Git](#subversion-to-git)
+ [CVS to Subversion](#cvs-to-subversion)
+ [References](#references)


<a name="2"></a>
---
## 2: [Requirements](#contents)

The following utilities are required:
+ [cvs2svn][] ([Ubuntu][deb.cvs2svn])
+ [git][] ([Ubuntu][deb.git])
+ [git-svn][] ([Ubuntu][deb.git-svn])
+ [rsync][] ([Ubuntu][deb.rsync])
+ [svnadmin][svn] ([Ubuntu][deb.svn])
+ [svnrdump][svn] ([Ubuntu][deb.svn])


<a name="3"></a>
---
## 3: [Subversion to Git](#contents)

<span style="font-size: 14px; font-style: italic;">
For this section you will need <span style="color: blue;">rsync</span>, <span style="color: blue;">svnadmin</span>, <span style="color: blue;">svnrdump</span>, <span style="color: blue;">git-svn</span>, & <span style="color: blue;">git</span>.
</span>


<a name="3.1"></a>
### 3.1: Method 1 : Clone a remote repository with git-svn

The [*git-svn*][man.git-svn] command, generally invoked as '<span style="color: blue; font-style: italic;">git svn</span>', is a tool for managing Subversion repositories with Git. It "<span style="font-style: italic;">can track a standard Subversion repository</span>". As I understand it, it basically allows you to manage a Subversion repository as though it were a [distributed version control system][wiki.dvcs]. A repository can be followed if is layed out with the common *trunk*, *branches*, & *tags* directories by using the <span style="color: blue; font-style: italic;">--stdlayout</span> option. Otherwise, <span style="color: blue; font-style: italic;">-T</span> (trunk), <span style="color: blue; font-style: italic;">-b</span> (branch), and/or <span style="color: blue; font-style: italic;">-t</span> (tags) must be used.

Of the options layed out here, this is probably the simplest. Use <span style="color: blue; font-style: italic;">git svn</span> to clone the remote repository:

```
$ git svn clone --stdlayout remote-svn-repo local-repo
```

The local '*local-repo*' is automatically converted.

Here is an example for the [Debreate][debreate] project hosted at [SourceForge][sourceforge]:

```
$ git svn clone --stdlayout svn://svn.code.sf.net/p/debreate/svnroot debreate-git
```

If you want to transfer usernames/emails to the Git repository, you must first checkout a working copy of the Subversion repository:

```
$ svn co remote-repo-url local
```

Change to the local working copy & run the following command to generate a "*users.txt*" file:

```
$ cd local
$ svn log --xml | grep author | sort -u | perl -pe 's/.*>(.*?)<.*/$1 = /' | tee ../users.txt
```

A list of usernames found in the Subversion tree will be added to the file. You need to manually fill out the new Name/Username & email for the Git repository:

```
user1 = SomeGuy <someguy@someplace.com>
user2 = Gamer Girl <ggirl123@gamesrcool.net>
...
...
```


<a name="3.2"></a>
### 3.2: Method 2 : Get a local copy of a full repository (not just a working copy, e.g. svn checkout)


<a name="3.2.1"></a>
#### 3.2.1: Option 1 : Using svnadmin & svnsync

Create an empty repository with <span style="color: blue; font-style: italic;">svnadmin create</span>:
```
$ svnadmin create local-repo
```

An executable script named "*pre-revprop-change*" must exist in the "*hooks*" directory:

```
$ echo '#!/bin/sh' > local-repo/hooks/pre-revprop-change
$ chmod 0755 local-repo/hooks/pre-revprop-change
```

Initialize the repository with <span style="color: blue; font-style: italic;">svnsync init</span>. The path to the local repository must use the '*file://*' protocol & must be an absolute path:

```
$ svnsync init file:///path/to/local-repo remote-repo-url
```

And finally, use <span style="color: blue; font-style: italic;">svnsync sync</span> to bring the repository up to date:

```
$ svnsync sync file:///path/to/local-repo
```

You can now checkout a working copy of the repository:

```
$ svn co file:///path/to/local-repo/branch working-copy
```

<a name="3.2.2"></a>
##### 3.2.2: Option 2 : Using svnrdump to create a dump file

With the <span style="color: blue; font-style: italic;">svnrdump</span> command you can dump the contents of a remote repository to a local file: 

```
$ svnrdump dump remote-repo-url > dump-file
```

Debreate project example:

```
$ svnrdump dump svn://svn.code.sf.net/p/debreate/svnroot debreate-svn.dump
```


**Converting dump to local repository**

Using <span style="color: blue; font-style: italic;">svnadmin</span>, a local repository can be created from the dump:

```
$ svnadmin create local-repo
$ svnadmin load local-repo < dump-file
```

Once again, you can now check out a working copy.


<a name="3.2.3"></a>
#### 3.2.3: Convert to Git

```
$ git svn clone --stdlayout file:///path/to/local-repo git-repo
```


<a name="4"></a>
---
## 4: [CVS to Subversion](#contents)

This tutorial will show you how to convert a CVS repository to Git. But the method shown here requires that it first be converted to a Subversion repository (See: [Subversion to Git](#subversion-to-git)).

For this section you will need <span style="color: blue;">*rsync*</span>, <span style="color: blue;">*cvs2svn*</span>, & <span style="color: blue;">*svnadmin*</span>.

#### Step 1: Making a complete copy of a CVS repository

```
$ rsync -av --delete-delay rsync://[path-to-repo]/* [path-to-local-repo]
```

Explanation of arguments([rsync manpage][man.rsync]):
+ <span style="color: blue;">**a**</span> : Archive mode
+ <span style="color: blue;">**v**</span> : Verbose (optional)
+ <span style="color: blue;">**delete-delay**</span> : Find deletions during, delete after

Here is an example for the [wxSVG][wxsvg] project hosted an SourceForge:

```
$ rsync -av --delete-delay rsync://wxsvg.cvs.sourceforge.net/cvsroot/wxsvg/* wxsvg-cvs-copy
```

#### Step 2: Convert the local repository to a Subversion dump file

```
$ cvs2svn --dumpfile=[dump-file] [path-to-local-repo]
```

Here is an example for the wxSVG backup above:

```
$ cvs2svn --dumpfile=wxsvg.svn-dump wxsvg-cvs-copy
```

#### Step 3: Convert the Subversion dump file to a local Git repository

From this point, you can follow the instructions under [**Subversion to Git**](#git-svn-dump).


---
### [References](#contents)

+ [Audacity wiki: CVS To SVN Migration](http://wiki.audacityteam.org/wiki/CVS_To_SVN_Migration)
+ [Backing Up & Restoring a Remote SVN Repository](http://www.crowbarsolutions.com/backing-up-restoring-a-remote-svn-repository/)
+ [converting sourceforge.net repository from CVS to subversion](http://uucode.com/blog/2010/03/09/converting-sourceforgenet-repository-from-cvs-to-subversion/)
+ [How to migrate SVN repository with history to a new Git repository?](http://stackoverflow.com/questions/79165/how-to-migrate-svn-repository-with-history-to-a-new-git-repository)
+ [Mirror a Subversion repository](http://www.microhowto.info/howto/mirror_a_subversion_repository.html)
+ [Moving a Subversion Repository to Another Server](https://www.petefreitag.com/item/665.cfm)


[cvs]: http://savannah.nongnu.org/projects/cvs
[cvs2svn]: http://cvs2svn.tigris.org/
[git]: http://git-scm.com/
[git-svn]: https://git-scm.com/docs/git-svn
[rsync]: https://rsync.samba.org/
[svn]: http://subversion.apache.org/

[deb.cvs2svn]: http://packages.ubuntu.com/cvs2svn
[deb.git]: http://packages.ubuntu.com/git
[deb.git-svn]: http://packages.ubuntu.com/git-svn
[deb.rsync]: http://packages.ubuntu.com/rsync
[deb.svn]: http://packages.ubuntu.com/subversion

[man.git-svn]: https://linux.die.net/man/1/git-svn
[man.rsync]: https://linux.die.net/man/1/rsync

[debreate]: https://sourceforge.net/projects/debreate
[sourceforge]: https://sourceforge.net/
[wxsvg]: https://sourceforge.net/projects/wxsvg

[wiki.cvs]: https://en.wikipedia.org/wiki/Concurrent_Versions_System
[wiki.dvcs]: https://en.wikipedia.org/wiki/Distributed_version_control
[wiki.git]: https://en.wikipedia.org/wiki/Git
[wiki.rsyn]: https://en.wikipedia.org/wiki/Rsync
[wiki.svn]: https://en.wikipedia.org/wiki/Apache_Subversion
