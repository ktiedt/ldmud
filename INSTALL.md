Topics covered:

  - Prerequisites: what do you need to compile and run LDMud-3.5.x
  - Unix or Unix-like systems (like Linux, FreeBSD, MacOS X, ...).
  - 64 bit systems (LP64 model)
  - Compiling for 32-bit environments on 64-bit systems
  - Compiling sources directly from our git repository
  - IPv6
  - mySQL
  - Windows 95/98/NT/XP


Prerequisites
-------------
  - a POSIX.1-2001 conformant platform
  - a C99 compatible build environment
  - bison (yacc won't work! bison 2.7.x is known to work)
  - supported architectures: i386, x86_64 (work in progress, see section about
      64 bit systems). Other platforms are not actively supported and tested
      by us, but many will most likely work as well. Please try and tell us
      about your experiences.
  - autoconf and automake for compiling sources directly from our git
      repository.

  Optional libraries:
  - libpcre for the support of PCRE compatible regular expressions
  - mysql for the mysql package
  - postgresql for the postgresql package
  - libsqlite3 for the sqlite package
  - libgcrypt for various optional algorithms for hashing and crypting
  - OpenSSL or GnuTLS for TLS support
  - libxml2 or libiksemel for XML support
  - json-c (libjson0) for JSON support


Unix or Unix-like system
------------------------

  The driver uses a standard autoconfiguration system which on most
  systems does all the work for you (for exceptions see below).
  To prepare the compilation, execute the 'configure' script from
  within the src/ directory.
  [Note: If you got LDMud from our git repository please execute the
  autogen.sh script in src/ to generate the configure script and other
  auto-generated files.]

  If you have git or compile for other people (e.g. to distribute a binary),
  it is seriously recommended to compile from a cloned git repository.
  Otherwise the version information of the binary will not be precise.

  configure checks for a number of site specific settings and uses this
  information to create the files machine.h (from machine.h.in), Makefile
  (from Makefile.in) and config.h (from config.h.in). We'll come back to
  config.h below.

  configure takes a lot of arguments (--help will tell you everything),
  but the most important are these:

    --prefix=PREFIX:  the base directory for the mud installation,
                        defaults to /usr/local/mud .
    --bindir=DIR:     the directory to install the executables in,
                        defaults to ${PREFIX}/bin .
    --libdir=DIR:     the directory where the mudlib is found,
                        defaults to ${PREFIX}/lib .
    --includedir=DIR: the directory where driver's LPC include files
                        are supposed to live.
                        defaults to ${PREFIX}/include (which is usually wrong).
    --libexecdir=DIR: the directory where the programs for the ERQ are found,
                        defaults to ${PREFIX}/libexec .

  These settings are written into the Makefile and compiled into the driver,
  just the mudlib directory setting can be changed with a commandline
  argument.

  A lot of the drivers parameters can be tweaked for better performance; these
  parameters are defined in config.h . This file too is created by configure,
  which provides sensible defaults for all parameters for which no explicite
  setting is provided. To tweak a setting yourself, pass the argument
  '--enable-<option>=yes|no' resp. '--with-<option>=<value>' to configure on
  the commandline.

  Alternatively, the indivial specifications can be collected in a settings
  file, which is stored in the directory src/settings/. To use the
  setting file <osb>, give '--with-<osb>' as argument to configure. The
  file src/settings/default documents the available settings. The setting
  files are self-executing: './settings/<foo> [<extra-configure-args>]' will
  start configure with the proper commandline arguments.


  The following environment variables can be used to tweak the behaviour
  of the configure script:
    CC:           the name of the C compiler
    CFLAGS:       compiler flags to be used during the configure script
    EXTRA_CFLAGS: compiler flags to be used when compiling the game driver
    LDFLAGS:      linker flags to be used by configure and for linking the
                  game driver.


  After configuration is finished, you may want to modify the Makefile
  to fine tune those parameters which are not covered by the configuration.

  The compilation is done using make. Following targets are implemented:

    <none>:          compile the driver, named 'ldmud'.
    install:         compile the driver and install it in ${bindir}
    utils:           compile the utilities, especially the ERQ demon
    install-utils:   compile and install the utilities in ${bindir}
    install-headers: install the driver header files in ${includedir}.
    install-all:     compile and install everything.

  To actually run a mud, you need a mudlib. The driver comes with the
  source of the old 2.4.5 mudlib in mud/lp-245/, support for other mudlibs
  is also found in mud/. Copy all mudlib files into your 'lib' directory
  and make sure that the files from mudlib/sys/ all go into your lib's
  include directory (usually 'sys/').

  After compilation, you may run our testsuite (work in progress!) in the
  test subdirectory (launch test/run.sh).

  To test the driver, start it with 'ldmud 4242'. If you see the message
  'LDMud ready for users', the driver is up and accepting connections. Test
  it with 'telnet localhost 4242'.


64 bit systems (using the LP64 model)
-------------------------------------
The driver can be compiled on systems which use the LP64 data model (longs
and pointers are 64 bit wide). This is the case on many unixoids (but not
Windows).
There are some muds, which use the driver on LP64 architectures.
However, it still has some issues on these architectures. For details
please have a look at our bug tracker (http://mantis.ldmud.eu/mantis/)
in the category 'portability' (and there probably are some unknown ones
as well).
Full support for LP64 machines is planned in the current development
series 3.5.x. If you are willing to test and report issues, please do
so. ;-)

WARNING: LPC savefiles from 64 bit systems with integers exceeding the
         (signed) 32 bit integers can't be restored on 32 bit systems!

If you want to compile the driver for a 64 bit platforms and your compiler
does not produce LP64 code by default, we suggest to set CFLAGS, LDFLAGS
and EXTRA_CFLAGS (and maybe CC) prior to configure to suitable values to
cause your compiler to compile for the desired platform, e.g.
# export CFLAGS="-m64"
# export LDFLAGS="-m64"
# export EXTRA_CFLAGS=$CFLAGS
# ./configure

Setting EXTRA_FLAGS is important, otherwise the driver will be compiled using
different compiler options that the ones used during the configure run.
It is crucial that you DON'T CHANGE the compiler options controlling the
platform in the generated Makefile, unless you know very well what you
do. (If the machine.h generated by configure is for a different platform, the
driver may assume wrong sizes of data types, resulting in undefined behaviour
and crashes.)


Compiling for 32-bit environments on 64-bit systems
---------------------------------------------------
If you want to compile the driver for a 32 bit platforms and your compiler
does produce LP64 code by default, we suggest to set CFLAGS, LDFLAGS
and EXTRA_CFLAGS (and maybe CC) prior to configure to suitable values to
cause your compiler to compile for the desired platform, e.g.
# export CFLAGS="-m32"
# export LDFLAGS="-m32"
# export EXTRA_CFLAGS=$CFLAGS
# ./configure


Compiling sources directly from our git repository
--------------------------------------------------
If you want to be on the bleeding edge of development, you may check out the
LDMud sources from our git repository at github:
  https://github.com/ldmud/ldmud

To clone the repo:
  git clone https://github.com/ldmud/ldmud.git

LDMud-3.5.x (current development branch) is in the branch master.
LDMud-3.3.x in master-3.3 and LDMud-3.2.x in master-3.2.

There are also various tags for various releases.

You will need the autoconf package to generate and update some files 
Prior to configuring the driver, please run:
# ./autogen.sh
in the src/ directory. Then configure the driver as usual.


IPv6
----
  If your machine supports IPv6, the driver can be configured to use it: give
  '--enable-use-ipv6=yes' as argument to the configure script.


mySQL
-----
  If your machine has mySQL installed, the driver can be configured to use
  it: give '--enable-use-mysql=yes' as argument to the configure script.

  Alternatively, if your mySQL uses an unusual include/library path,
  the option can be given as '--enable-use-mysql=/unusual/path', which
  will use the given path as search path for both include and library
  files in addition to the normal system search paths. The include files will
  be searched in <path>/include and <path>, the library files will be search
  in <path>/lib/mysql, <path>/lib, and <path>, in this order.

  The username and password for the mySQL database are specified by
  the mudlib as arguments to the efun db_connect().

  Use mysqladmin to create any databases you want to provide - the
  names are later used in the efun db_connect() to connect to
  the databases.


Windows 95/98/NT/XP/7
---------------------

  To compile the gamedriver for Windows, you need the 'Cygwin' package,
  which is a port of gcc, bash, and other GNU/Unix programs. Once it
  is installed and running, the procedure is the same as under Unix.

  CygWin is available from http://cygwin.com/.
  (When installing make sure that your installation includes gcc, bash, make,
   sed, awk, and bison and stuff listed above...)

  There are some pre-compiled binaries for Windows linked from our homepage.
  Please have a look at:
  http://www.ldmud.eu/resources.html

  However, they are compiled for specific mudlibs, so your mileage may vary.

