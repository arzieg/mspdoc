# Automake

https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install


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

There are various directories defined for us by autotools—including bindir, libdir, and pkglibdir—but we can also define our own.

For example, if we wanted to install some Ruby scripts as part of our program, we could define a rubydir variable and tell automake to install our Ruby files there:

```
rubydir = $(datadir)/ruby
ruby_DATA = my_script.rb my_other_script.rb
```

Additional prefixes can be added before the install directory to further nuance automake’s behaviour.



4. Finale Scripte erzeugen

```
aclocal
autoconf
automake --add-missing
./configure
```


5. Distribution erstellen

`make dist`

6. Disstribution testen

`make distcheck`

7. Programm installieren

```
./configure
make
make install
```


