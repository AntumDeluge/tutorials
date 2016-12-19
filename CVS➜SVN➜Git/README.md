## Migrating [CVS][cvs] ➜ [Subversion][svn] (SVN) ➜ [Git][git]


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

The following utilities are used in these tutorials:
+ [cvs2svn][] ([Ubuntu][deb.cvs2svn])
+ [git][] ([Ubuntu][deb.git])
+ [git-svn][] ([Ubuntu][deb.git-svn])
+ [perl][] ([Ubuntu][deb.perl])
+ [rsync][] ([Ubuntu][deb.rsync])
+ [svnadmin][svn] ([Ubuntu][deb.svn])
+ [svnrdump][svn] ([Ubuntu][deb.svn])


<a name="svntogit"></a>
---
## 3: [Subversion to Git](#contents)


<a name="git-svn"></a>
### Clone & convert a remote Subversion repository with git-svn

***Requirements: git-svn***


The [*git-svn*][man.git-svn] command, generally invoked as '<span style="color: blue; font-style: italic;">git svn</span>', is a tool for managing Subversion repositories with Git. It "<span style="font-style: italic;">can track a standard Subversion repository</span>". As I understand it, it basically allows you to manage a Subversion repository as though it were a [*distributed version control system*][wiki.dvcs]. A repository can be followed if it is layed out with the common *trunk*, *branches*, & *tags* directories by using the <span style="color: blue; font-style: italic;">--stdlayout</span> option. Otherwise, <span style="color: blue; font-style: italic;">-T</span> (trunk), <span style="color: blue; font-style: italic;">-b</span> (branch), and/or <span style="color: blue; font-style: italic;">-t</span> (tags) must be used.

Of the methods laid out here, this is probably the simplest. The only step is to use <span style="color: blue;">*git-svn*</span> to clone the remote repository:

```
$ git svn clone --stdlayout remote-svn-repo local-repo
```

The *local-repo* is automatically converted. If you don't want Subversion revision information appended to the Git commit messages, add "<span style="color: blue;">*--no-metadata*</span>" to the command:

```
$ git svn clone --stdlayout --no-metadata remote-svn-repo local-repo
```

Here is an example for the [Debreate][debreate] project hosted at [SourceForge][sourceforge]:

```
$ git svn clone --stdlayout svn://svn.code.sf.net/p/debreate/svnroot debreate-git
```


<a name="usernames"></a>
### Transfer usernames & emails to new Git repository

***Requirements: Perl, Subversion, git-svn***


If you want to transfer usernames/emails to the Git repository, there is another step that must be taken beforehand. Checkout a working copy of the main branch of the remote repository:

```
$ svn co remote-repo-url/branch local
```

Change to the local working directory then run the following command to generate a "*users.txt*" file:

```
$ cd local
$ svn log --xml | grep author | sort -u | perl -pe 's/.*>(.*?)<.*/$1 = /' | tee ../users.txt
```

The file will be a list of contributer usernames, each followed by a "*=*", found in the Subversion repository. For each username, add the person's name, or username, followed by an email address in triangle brackets "*<>*". It doesn't matter what the name portion is. It can be the same as the Subversion username or the person's real name. The email is more important to Git & required to attribute contributions. The file should end up looking something like this:

```
user1 = SomeGuy <someguy@someplace.com>
user2 = Gamer Girl <ggirl123@gamesrcool.net>
...
...
```

Now when you invoke the ***svn git*** command, use the ***--authors-file*** option:

```
$ git svn clone --stdlayout --authors-file=users.txt remote-svn-repo local-repo
```

If you get an error "*Author: (no author) not defined in authors file*", add the following, or something similar, to the list of usernames:

```
(no author) = no_author <no_author@no_author>
```


<a name="3.2"></a>
### Use a local copy of a full repository (not just a working copy, e.g. svn checkout)

*The following is given only for informational purposes, since you can do the same thing in less steps with the previous method.*

To make a local copy of a Subversion repository, if you don't already have one, you can use one of the following.


<a name="svnsync"></a>
#### Create a local copy using svnsync & svnadmin

*Requirements: Subversion*


Create an empty repository with ***svnadmin create***:

```
$ svnadmin create local-repo
```

An executable script named "*pre-revprop-change*" must exist in the "*hooks*" directory:

```
$ echo '#!/bin/sh' > local-repo/hooks/pre-revprop-change
$ chmod 0755 local-repo/hooks/pre-revprop-change
```

Initialize the repository with ***svnsync init***. The path to the local repository must use the '*file://*' protocol & must be an absolute path:

```
$ svnsync init file:///path/to/local-repo remote-repo-url
```

Finally, use ***svnsync sync*** to bring the repository up to date:

```
$ svnsync sync file:///path/to/local-repo
```

You can now checkout a working copy of the repository:

```
$ svn co file:///path/to/local-repo/branch working-copy
```

<a name="svnrdump"></a>
#### Create a local copy using svnrdump

*Requirements: Subversion*


With the ***svnrdump*** command you can dump the contents of a remote repository to a local file: 

```
$ svnrdump dump remote-repo-url > dump-file
```

Debreate project example:

```
$ svnrdump dump svn://svn.code.sf.net/p/debreate/svnroot debreate-svn.dump
```

<a name="svndump-to-git"></a>
Create a bare local repository:

```
$ svnadmin create local-repo
```

Load the dump file into the repository:

```
$ svnadmin load local-repo < dump-file
```

Once again, you can now check out a working copy.


<a name="localsvn"></a>
#### Convert local Subversion repo to Git

*Requirements: git-svn*


Converting a local repository with ***git-svn*** is the same, just use the "*file://*" protocol:

```
$ git svn clone --stdlayout --authors-file=users.txt file:///path/to/local-repo git-repo
```


<a name="cvstosvn"></a>
---
## 4: [CVS to Subversion or Git](#contents)

This section will show you how to convert a CVS repository to Git. But the method shown here requires that it first be converted to a Subversion repository.

*Requirements: rsync, cvs2svn, & svnadmin*


+ ***TODO: Look up options for [cvs2svn][man.cvs2svn]***
+ ***TODO: Look up options for [rsync][man.rsync]***
+ ***TODO: Create section for [cvs2git][man.cvs2git] (included with cvs2svn)***
+ ***TODO: Create section for [git-cvsimport][man.git-cvsimport] (in [git-cvs][git-cvsimport] package)***

Make a local copy of the CVS repository with ***rsync***:

```
$ rsync -av --delete-delay rsync://remote-repo-path/* local-repo
```

Explanation of arguments([rsync manpage][man.rsync]):
+ <span style="color: blue;">**a**</span> : Archive mode
+ <span style="color: blue;">**v**</span> : Verbose (optional)
+ <span style="color: blue;">**delete-delay**</span> : Find deletions during, delete after

Here is an example for the [wxSVG][wxsvg] project hosted at SourceForge:

```
$ rsync -av --delete-delay rsync://wxsvg.cvs.sourceforge.net/cvsroot/wxsvg/* wxsvg-cvs-copy
```

Convert the local repository to a Subversion dump file with ***cvs2svn***:

```
$ cvs2svn --dumpfile=dump-file local-repo
```

Here is an example for the wxSVG backup above:

```
$ cvs2svn --dumpfile=wxsvg.svndump wxsvg-cvs-copy
```

See above for converting the dump file to a local Subversion repository, & in turn, using ***git-svn*** to convert that to a Git repository.


---
### [References](#contents)

+ [Audacity wiki: CVS To SVN Migration](http://wiki.audacityteam.org/wiki/CVS_To_SVN_Migration)
+ [Author: (no author) not defined in authors file](https://www.guyrutenberg.com/2011/11/09/author-no-author-not-defined-in-authors-file/)
+ [Backing Up & Restoring a Remote SVN Repository](http://www.crowbarsolutions.com/backing-up-restoring-a-remote-svn-repository/)
+ [converting sourceforge.net repository from CVS to subversion](http://uucode.com/blog/2010/03/09/converting-sourceforgenet-repository-from-cvs-to-subversion/)
+ [How to migrate SVN repository with history to a new Git repository?](http://stackoverflow.com/questions/79165/how-to-migrate-svn-repository-with-history-to-a-new-git-repository)
+ [Mirror a Subversion repository](http://www.microhowto.info/howto/mirror_a_subversion_repository.html)
+ [Moving a Subversion Repository to Another Server](https://www.petefreitag.com/item/665.cfm)


[cvs]: http://savannah.nongnu.org/projects/cvs
[cvs2svn]: http://cvs2svn.tigris.org/
[git]: http://git-scm.com/
[git-cvsimport]: https://git-scm.com/docs/git-cvsimport
[git-svn]: https://git-scm.com/docs/git-svn
[perl]: https://www.perl.org/
[rsync]: https://rsync.samba.org/
[svn]: http://subversion.apache.org/

[deb.cvs2svn]: http://packages.ubuntu.com/cvs2svn
[deb.git]: http://packages.ubuntu.com/git
[deb.git-cvs]: http://packages.ubuntu.com/search?keywords=git-cvs
[deb.git-svn]: http://packages.ubuntu.com/git-svn
[deb.perl]: http://packages.ubuntu.com/perl
[deb.rsync]: http://packages.ubuntu.com/rsync
[deb.svn]: http://packages.ubuntu.com/subversion

[man.cvs2git]: https://linux.die.net/man/1/cvs2git
[man.cvs2svn]: https://linux.die.net/man/1/cvs2svn
[man.git-cvsimport]: https://linux.die.net/man/1/git-cvsimport
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
