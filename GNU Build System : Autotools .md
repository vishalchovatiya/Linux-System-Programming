### Brief

- Autotools are suite of programming tools used to make source code packages portable to many Unix-like systems.
- Autotools mainly consists of the GNU utility programs Autoconf, Automake & Libtool.

### Why We Need Autotools

- `Autoconf` : automatically generates `configure` script by scans of existing tree to find its dependencies, that are supposed to work on all kinds of platforms. Configure generates a `config.h` file (from a template) which programs can include to work around portability issues. For example, if `HAVE_LIBPTHREAD` is not defined, use forks instead.
- `Automake` : No need to write lengthy & complex makefiles. just define target, dependencies, flags, etc.
- `Libtool` : simplifying the building and installation of shared libraries on any Unix-like system. 
- Autotools can handle cross plateform development smoothly.

Above statements may seems like alien right now. But just go through below example & then read again. You will get what i just said

### Compiling Hello World With Autotools

- Creating following files in empty directory

#### configure.ac
```
AC_INIT([PackageName], [1.0], [bug-report@address])     # Initialize `Autoconf`. Specify package's name, version & bug-report address.
AM_INIT_AUTOMAKE                                        # Initialize Automake
AC_PROG_CC                                              # Check for a C compiler
AC_CONFIG_HEADERS([config.h])                           # Declare config.h as output header
AC_CONFIG_FILES([Makefile src/Makefile])                # Declare Makefile & src/Makefile as output files 
AC_OUTPUT                                               # Output all declared files
```
#### Makefile.am
```
SUBDIRS = src                                           # Build recursively in src directory 
```
#### src/Makefile.am
```
bin_PROGRAMS = hello                                    # "hello" is target & will be installed in bindir (as specify "bin_")
hello_SOURCES = main.c                                  # Dependencies of target hello is main.c
```
#### src/main.c
```
#include<stdio.h>

int main(){
        puts("Hello World !");
        return 0;
}
```
#### Directory Structre
```
$ ls -R
.:
configure.ac  Makefile.am  src

./src:
main.c  Makefile.am
```
#### Build Package

```
$ autoreconf --install
$ ./configure --prefix=$(pwd)
$ make
$ bin/hello
Hello World !
$
```
At this stage, there are lot of other files are generated as follows : 

- `Makefile.in`, `config.h.in`, `config*` : expected configuration templates.
- `aclocal.m4` : definitions for third-party macros used in `configure.ac`.
- `depcomp`, `install-sh`, `missing` : auxiliary tools used during the build .
- `autom4te.cache/` : Autotools cache files.

### How Autotool Works

#### Understand Build Packege Procedure

##### Step 1 : `autoreconf --install`

- `autoreconf` is a helper that knows how to call `autoconf`, `autoheader`, `aclocal`, `automake`, `libtoolize`, `autopoint`, etc tools in the right order.

**Behind `autoreconf`**

- `aclocal` : Scan `configure.ac` for uses of third-party macros, and gather definitions in `aclocal.m4`.
- `autoconf` : Create `configure` from `configure.ac` & `aclocal.m4`.
- `autoheader` : Create `config.h.in` from `configure.ac`.
- `automake --add-missing` : Create `Makefile.in`s from `Makefile.am`s, `configure.ac` & `aclocal.m4`. `--add-missing` option will add required file(like `config.guess`, `config.sub`, `missing`, `depcomp`, `install-sh`, etc) to carry out build process.

__Note__ : Run `libtoolize`, If you use `LT_INIT` to create shared library with `libtool` in `configure.ac`, otherwise you will get error as `configure.ac:[LINE]: error: required file './ltmain.sh' not found`.

##### Step 2 : `./configure --prefix=$(pwd)`

- `configure` will create `Makefile`s from `Makefile.in`s

##### Step 3 : `make & make install`

- Finally `make & make install` - Do all things for you

#### Generalise Idea

- Practically, You do not have to remember the interaction of all tools, just call autoreconf yourself and let it deal with all the lower level tools. 
- `autoreconf` is your friend. You only need a rough idea of the purpose of each tool to understand errors.

##### GNU Autoconf

- `autoconf`  Create `configure` from `configure.ac`.
- `autoheader`  Create `config.h.in` from `configure.ac`.
- `autoreconf`  Run all tools in the right order.
- `autoscan`  Scan sources for common portability problems,and related macros missing from `configure.ac`.
- `autoupdate`  Update obsolete macros in `configure.ac`.
- `ifnames`  Gather identifiers from all `#if/#ifdef/...` directives.
- `autom4te`  The heart of `Autoconf`. It drives `M4` and implements the features used by most of the above tools.  Useful for creating more than just configure files.

##### GNU Automake

- `automake`  Create `Makefile.in`s from `Makefile.am`s and `configure.ac`.
- `aclocal`  Scan `configure.ac` for uses of third-party macros, and gather definitions in `aclocal.m4`.

##### GNU Libtool

- `Libtool` Helps manage the creation of static and dynamic libraries on various Unix-like operating systems

### Distributing Packege

```
$ make distcheck
=================================================
packagename-1.0 archives ready for distribution:
packagename-1.0.tar.gz
=================================================
```
- At the end of command execution you will find `packagename-1.0.tar.gz` in same folder.

### Generating Shared Library With Autotools

#### configure.ac
```
AC_INIT([PackageName], [1.0], [bug-report@address])
AM_INIT_AUTOMAKE([foreign])
LT_INIT                         # Used to initialize libtool to create shared library       
AM_MY_MACRO="-I ./"             # Delacring Custome Macro
AC_SUBST([AM_MY_MACRO])         # Passing Custome Macro to Makefile.am
AC_PROG_CC
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([Makefile sum/Makefile src/Makefile])
AC_OUTPUT
```
#### Makefile.am
```
SUBDIRS = sum src
```
#### src/main.c
```
#include<stdio.h>

int main(){
        printf("sum(0,1) = %d\n", sum(0,1));
        return 0;
}
```
#### src/Makefile.am
```
bin_PROGRAMS = sum
sum_SOURCES = main.c
sum_LDADD = ../sum/libsum.la    # add & link sum against libsum.so
sum__LDFLAGS = $(AM_MY_MACRO)   # Macro Passed By configure.ac
```
#### sum/sum.c
```
#include<stdio.h>

int sum(int a, int b)
{
        return a + b;
}
```
#### sum/Makefile.am
```
lib_LTLIBRARIES = libsum.la
libsum_la_SOURCES = sum.c
```
#### Directory Structre
```
$ ls -R
.:
configure.ac  Makefile.am  src  sum

./src:
main.c  Makefile.am

./sum:
Makefile.am  sum.c
```

#### Build Package

```
$ autoreconf --install
$ ./configure --prefix=$(pwd)
$ make install
$ ./bin/sum
sum(0,1) = 1
$
$ ldd ./bin/sum
        linux-vdso.so.1 =>  (0x00007fff606ac000)
        libsum.so.0 => /home/vishal/asdf/lib/libsum.so.0 (0x00007fdcf43a6000)
        libc.so.6 => /lib64/libc.so.6 (0x00007fdcf3fcd000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fdcf45a9000)
```
- At the end of `make install`, there are two folders named as `lib` & `bin` having shared library in `lib` & executable in `bin`.

### What Next ?

- This is just short introduction of How Autotools help us ! There are lot to discover about its competitors like [CMake](https://cmake.org/), [Scons](www.scons.org), etc.

#### Sources 

1. [Autotools](https://www.lrde.epita.fr/~adl/dl/autotools.pdf)
2. [GNU Build System](https://en.wikipedia.org/wiki/GNU_Build_System)
3. [Automake](https://www.gnu.org/software/automake/manual/automake.html)
