.. _lnx_allg:

################
Linux Allgemein
################

Linux Kernel Release Nummerierung:
----------------------------------

Quelle: https://www.youtube.com/watch?v=VfEHZnwdpw4&list=PL03Lrmd9CiGdBvVUXpZCKK88-Vpd5VwEo&index=5

  * ist keine semantische Nummerierung https://semver.org
  * X.YY.ZZ
  
    * X.YY = Major Version (not major/minor)
    * ZZ = Bugfixes
  * Alle 9-10 Wochen wird ein Major/Minor erstellt
  * 4.19.x -> 4.20.x upgrade can contain more features/breaking changes than 4.20.x->5.0.x
  * Repo vom Kernel: https://github.com/torvalds/linux
  * Typischerweise ist ein stable release 2-3 Monate aktiv, danach ist es EOL
  * Daher gibt es longterm stable releases
  
    * Backport ca. 2 Jahre
    * Release jedes Jahr
  
Semantische Versionierung
--------------------------

Quelle: https://semver.org/

Given a version number MAJOR.MINOR.PATCH, increment the:

1. MAJOR version when you make incompatible API changes
2. MINOR version when you add functionality in a backward compatible manner
3. PATCH version when you make backward compatible bug fixes

Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.

Is there a suggested regular expression (RegEx) to check a SemVer string?
There are two. One with named groups for those systems that support them (PCRE [Perl Compatible Regular Expressions, i.e. Perl, PHP and R], Python and Go).

See: https://regex101.com/r/Ly7O1x/3/

And one with numbered capture groups instead (so cg1 = major, cg2 = minor, cg3 = patch, cg4 = prerelease and cg5 = buildmetadata) that is compatible with ECMA Script (JavaScript), PCRE (Perl Compatible Regular Expressions, i.e. Perl, PHP and R), Python and Go.

See: https://regex101.com/r/vkijKf/1/