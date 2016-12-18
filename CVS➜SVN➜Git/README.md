## Converting [CVS][cvs] & [Subversion (SVN)][svn] repositories to [Git][git]


<a name="contents"></a>
---
### Contents

+ [Requirements](#requirements)
+ [Subversion to Git](#subversion-to-git)
+ [CVS to Subversion](#cvs-to-subversion)
+ [References](#references)


<a name="requirements"></a>
---
### [Requirements](#contents)

The following utilities are required:
+ [cvs2svn][] ([Ubuntu][deb.cvs2svn])
+ [git][] ([Ubuntu][deb.git])
+ [git-svn][] ([Ubuntu][deb.git-svn])
+ [rsync][] ([Ubuntu][deb.rsync])
+ [svnadmin][svn] ([Ubuntu][deb.svn])
+ [svnrdump][svn] ([Ubuntu][deb.svn])


---
### [Subversion to Git](#contents)

For this section you will need <span style="color: blue;">*rsync*</span>, <span style="color: blue;">*svnadmin*</span>, <span style="color: blue;">*svnrdump*</span>, <span style="color: blue;">*git-svn*</span>, & <span style="color: blue;">*git*</span>.

#### Step 1: Option 1: Making a complete copy of a Subversion repository

???

#### Step 1: Option 2: Creating a dump of a Subversion repository

##### Remote repository:

```
$ svnrdump dump [path-to-repo] > [dump-file]
```

Here is an example for the [Debreate][debreate] project hosted at [SourceForge][sourceforge]:

```
$ svnrdump dump svn://svn.code.sf.net/p/debreate/svnroot debreate-svn.dump
```

##### Local repository:

???

##### Converting dump to local repository (optional)

With the dump file you can use <span style="color: blue;">*svnadmin*</span> to create the actual local repository:

```
$ svnadmin create [path-to-local-repo]
$ svnadmin load [path-to-local-repo] < [dump-file]
```

#### Step 2: Option 1: Create Git repository from Subversion repository

???

<a name="git-svn-dump"></a>
#### Step 2: Option 2: Create Git repository from Subversion dump file

???


---
### [CVS to Subversion](#contents)

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

[man.rsync]: https://linux.die.net/man/1/rsync

[debreate]: https://sourceforge.net/projects/debreate
[sourceforge]: https://sourceforge.net/
[wxsvg]: https://sourceforge.net/projects/wxsvg

[wiki.cvs]: https://en.wikipedia.org/wiki/Concurrent_Versions_System
[wiki.git]: https://en.wikipedia.org/wiki/Git
[wiki.rsyn]: https://en.wikipedia.org/wiki/Rsync
[wiki.svn]: https://en.wikipedia.org/wiki/Apache_Subversion
