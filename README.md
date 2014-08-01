THIS IS A REUPLOAD OF NET/2

ORGINAL README:


This is the src tree for the second Berkeley networking distribution.
This file is intended to be a simple, preliminary guide to finding your
way around and compiling the programs.  We apologize that this
distribution has so little in the way of compatibility with previous
systems.  We had hoped that we could provide compatibility with at least
4.3BSD, but we simply did not have sufficient time to accomplished the
task.  In general, this release is similar to the 4.3BSD-Reno, although
it has a new virtual memory system and other changes.

First, there has been a major reorganization of the file system.  (You
may have seen similar reorganizations on 4.3BSD-Reno and systems shipped
by Sun Microsytems and Digital Equipment Corporation, among others.)
In general, the reorganization is as follows.  (Directories not listed are
pretty much unchanged, i.e.  /dev is the same as always.)

	/etc		configuration files (NO BINARIES)
	/bin		binaries needed when running single-user
	/sbin		binaries for the root user when running single-user

	/var		per machine variable directories
	/var/mail	the old /usr/spool/mail
	/var/spool	the old /usr/spool directories
	/var/tmp	the old /usr/tmp
	/var/acct	the old /usr/adm
	/var/log	log files
	/var/crash	crash dumps

	/usr/bin	the rest of the binaries
	/usr/lib	system libraries (NO BINARIES)
	/usr/libdata	system datafiles
	/usr/libexec	programs executed by other programs
	/usr/sbin	the rest of the binaries for the root user
	/usr/share	architecture independent files

The directories containing the source tree parallel the directories where
the binaries live, i.e. the binaries for /usr/bin are in /usr/src/usr.bin,
the files that are installed in /usr/share/misc live in
/usr/src/share/misc, and so on.  It is our intent that the entire system
be installed from /usr/src -- the files normally found in /etc are
prototyped (and installed) from /usr/src/etc.  Include files are installed
from /usr/src/include.  One exception to this is the software not
maintained at Berkeley.  For example, the Kerberos software can be found
in /usr/src/kerberosIV, and the ISODE software is in
/usr/src/contrib/isode.  Manual pages are in the directories of the
programs that they document; if they aren't directly related to a program,
they are in /usr/src/share/man.

Make has changed a lot.  It's pretty well documented, so you should read
the man page.  All of the makefiles in /usr/src have been modified to use
the new make features.  Your make will almost certainly not work on these
makefiles.  However, the new make will work on your old makefiles.  If
you only wish to install one or two programs, you may want to just create
makefiles for them and build them by hand.  If you want to build the entire
system, you probably want to get our make running on your system.

This is done by by going to usr.bin/src/make and entering "make -f
Makefile.dist".  This is a minimal makefile which just compiles the make
program.  It will create a binary named pmake.  Compiling pmake on your
system may fail.  If it does, there's probably a difference in your
/usr/include files that make is unhappy about.  You probably want to
figure out what the real problem is in this case.  Loading make on your
system may also fail.  If it does, you are probably missing one or more
routines in your C library that make needs.  Finding the correct routine
in the lib/libc/* directories, and creating a .o for the make directory
will probably get you around this problem.  Once pmake compile and loads,
it can be installed as "make".

Once you have a "new" make working, you have to install the template files
that it uses.  These files are in the directory src/share/mk.  Normally, they
are installed in the directory /usr/share/mk.  If you wish to install them
somewhere else, change the file pathnames.h in src/usr.bin/make to reflect
where you plan to install them.  There's a file named bsd.README in the
src/share/mk directory that briefly discusses how the BSD make templates
work.  It's not necessary reading, but it might be useful.

Once you have a make compiled and its template files installed, you can use
the standard makefiles.  One other comment, most of the standard makefiles
will attempt to build manual pages as well as the program.  This will be a
problem, because the manual pages require roff macro packages which will not
have been installed.  You can install these macros (see src/share/tmac),
or use the command "make NOMAN=noman" or add NOMAN=man as part of your
"MAKE" environmental variable when you make the BSD source to solve this
problem.

In each of the source directories you will find a symbolic link named "obj".
This symbolic link points to somewhere in the file hierarchy /usr/obj.  For
example, the "obj" symlink in bin/ls points to /usr/obj/bin/ls.  This is the
way that we build multiple architectures from a single source tree.  We
create a /usr/obj that is local to each machine which is building for an
architecture.  Then, we remote mount the source tree (often read-only) and
start the compile.  Make changes directory into the "obj" subdirectory, and
builds the object files there.  (There is one real nastiness in this scheme.
Any makefile wishing to reference a file relative to the source directory
must use the ${.CURDIR} macro before the path name, because when make runs
it cd's into the "obj" directory.  This *will* be corrected by 4.4BSD, but
we haven't done it yet.)  A simple work-around is to remove the symbolic
link obj, or make it a real sub-directory of the source directory.

Now you're ready to try and build the system.  First, we haven't really done
this (as I said before, we just ran out of time).  So don't take the following
as a real solution, it's simply the way that we had planned to approach the
problem.

There are really two problems that you're likely to encounter.  The first
are include files that aren't what the BSD source expects, and the second
are C library routines that are either missing or different.  The include
files are probably best handled by creating a directory, called, for the
sake of discussion, bsdinclude, in the top level of the distribution
source tree.  Add a -I include path to the CFLAGS macro in the source
makefiles that you are trying to compile so that the compiler looks for
its include files in bsdinclude first.  (Another way to do this, to avoid
modifying the makefiles, is to put the -I include path into the
environmental variable "COPTS".  This environmental variable is used by
make.)  Then, as you encounter problems in compiling, create include files
that fix the problem.

For example, one of the changes that we've made in our release is that
we've extracted all full path names from the source code and placed them
either in an include file in the source directory or an include file in
/usr/include.  Therefore, you will find a number of programs that include
<paths.h> (the path include file for the entire system).  Since your
system will probably not have a paths.h include file, you can install the
BSD one in bsdinclude (modifying it as necessary) and the problem should
go away.  However, our <utmp.h> include file has had paths added to it,
as well, and now includes the <lastlog.h> include file as well.  To make
this work, I'd suggest creating a utmp.h file in bsdinclude which #defines
the paths that the BSD utmp.h include file does, but which then includes
your standard utmp.h and lastlog.h include files.  So, the bsdinclude
version of utmp.h might look like:

	#define _PATH_UTMP      "/var/run/utmp"
	#define _PATH_WTMP      "/var/log/wtmp"
	#define _PATH_LASTLOG   "/var/log/lastlog"

	#include "/usr/include/lastlog.h"
	#include "/usr/include/utmp.h"

I believe that this approach will make it possible to build the C library.
Once the C library is built, install it somewhere.  As you compile
programs you will probably find unresolved references that need to be
satisfied using the BSD library.  I'd suggest adding the BSD library 
*after* the standard C library.  You can do this by changing the makefiles,
or adding the string "LDADD=-lc the/bsd/library/path" to your environment.
Note, programs that require other libraries will probably require  additional
information in the LDADD environmental variable.
