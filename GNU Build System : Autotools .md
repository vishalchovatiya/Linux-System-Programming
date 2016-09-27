### Brief
- Autotools is suite of programming tools used to make source code packages portable to many Unix-like systems
- Autotools mainly consists of the GNU utility programs Autoconf, Automake, and Libtool

### Why we need Autotools

- Portable Build, Uniform Packages

### Compiling Hello World With Autotools

- Creating following files in empty directory

#### configure.ac
```
AC_INIT([PackageName], [1.0], [bug-report@address])
AM_INIT_AUTOMAKE
AC_PROG_CC
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([Makefile src/Makefile])
AC_OUTPUT
```
#### Makefile.am
```
SUBDIRS = src
```
#### src/main.c
```
#include<stdio.h>

int main(){
        puts("Hello World !");
        return 0;
}
```
#### src/Makefile.am
```
bin_PROGRAMS = hello
hello_SOURCES = main.c
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

**Behind autoreconf**

- `libtoolize` : If you use with `LT_INIT`(to create shared lib) in configure.ac otherwise you got error prompt as `configure.ac:[LINE]: error: required file './ltmain.sh' not found`.
- `aclocal` : Scan `configure.ac` for uses of third-party macros, and gather definitions in `aclocal.m4`.
- `autoconf` : Create `configure` from `configure.ac` & `aclocal.m4`.
- `autoheader` : Create `config.h.in` from `configure.ac`.
- `automake --add-missing` : Create `Makefile.in`s from `Makefile.am`s, `configure.ac` & `aclocal.m4`. `--add-missing` option will add required file(like `config.guess`, `config.sub`, `missing`, `depcomp`, `install-sh`, etc) to carry out build process.

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

### Generating Packege

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
AC_CONFIG_MACRO_DIR([m4])
LT_INIT

AC_PROG_CC
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([Makefile sum/Makefile src/Makefile])
AC_OUTPUT
```
#### Makefile.am
```
SUBDIRS = sum src
ACLOCAL_AMFLAGS = -I m4
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
sum_LDADD = ../sum/libsum.la
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

> Build Package

```
$ autoreconf --install
$ ./configure --prefix=$(pwd)
$ make install
$ ./bin/sum
sum(0,1) = 1
$
```
- At the end of make install, there are two folders named as `lib` & `bin` having shared library in `lib` & executable in `bin`.
