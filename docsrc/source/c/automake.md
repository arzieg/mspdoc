# Automake

https://earthly.dev/blog/autoconf/

Benötigt wird autotools. Autotools besteht aus 3 Komponenten: 

* autoconf
* automake
* aclocal

Prozess: 

1. Programm schreiben

2. configure.ac anlegen

Minimal: 

```
AC_INIT([helloworld], [0.1], [maintainer@example.com])
AM_INIT_AUTOMAKE
AC_PROG_CC
AC_CONFIG_FILES([Makefile])
AC_OUTPUT
```

3. Makefile.am anlegen

```
AUTOMAKE_OPTIONS = foreign
bin_PROGRAMS = helloworld
helloworld_SOURCES = src/main.c
```

4. Finale Scripte erzeugen

```
aclocal
autoconf
automake --add-missing
```

5. Distribution erstellen

`make dist`

6. Programm erstellen

`make`


