
### Compiling Hello World With Autotools

configure.ac
```
AC_INIT([PackageName], [1.0], [bug-report@address])
AM_INIT_AUTOMAKE
AC_PROG_CC
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([Makefile src/Makefile])
AC_OUTPUT
```
Makefile.am
```
SUBDIRS = src
```
src/main.c
```
#include<stdio.h>

int main(){
        puts("Hello World !");
        return 0;
}
```
src/Makefile.am
```
bin_PROGRAMS = hello
hello_SOURCES = main.c
```
