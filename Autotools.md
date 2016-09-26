
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
Preparing the Package

```
$ autoreconf --install
$ ./configure --prefix=$(pwd)
$ make
$ bin/hello
Hello World !
$
```
- At this stage there are lot of other files are generated. we will look for it later.

- Makefile.in, config.h.in, configure* : expected con guration templates
- aclocal.m4 : definitions for third-party macros used in configure.ac
- depcomp*, install-sh*, missing* : auxiliary tools used during the build 
- autom4te.cache/ : Autotools cache files

> Generating Packege

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

> Preparing the Package

```
$ autoreconf --install
$ ./configure --prefix=$(pwd)
$ make install
$ ./bin/sum
sum(0,1) = 1
$
```
- At the end of make install, there are two folders named as lib & bin having shared library in lib & executable in bin
