.. _vi_allg:

################
VI Allgemein
################



:zR									-> Uncollapse all
:set list / :set nolist				-> Anzeigen von special characters
:set paste                          -> Insert resetten, kein Kommentarfeld am Anfang
:set nopaste					    -> Reset von :set paste, also alles wie vorher


Delete: 
 from line to line:  
	:<start>,<end>d
	:.,5d #deletes lines between the current line and the fifth line.
	:.,$d #removes all lines starting from the current line till the end.
	:%d #clears the entire file

Insert: 
 insert a file:
	:r foo.txt    Insert the file foo.txt below the cursor.
	:0r foo.txt   Insert the file foo.txt before the first line.
	:r !ls        Insert a directory listing below the cursor.
	:$r !pwd      Insert the current working directory below the last line.

No Autoindent: 
:setl noai nocin nosi inde=

Line Numbers
:set nu   - show 
:set nonu - disable


Files
======
:ls        = list buffer
n <strg>   = Springe zu File nr. N
:buffer N  = Springe zu File nr. N
<strg>^    = Toggle zw. File % und # hin und her
:bprevious = Springe eine Datei vor
:bnext     = Zeige nächste Datei
:bfirst    = erste Datei
:blast     = letzte Datei

Windows
=========
<strg>w+v  = veritkaler Split
<strg>w+s  = horizontaler Split

<strg>w+w oder <strg>w+<strg>w (geht schneller)  = toggle zum nächsten Fenster
<strg>w+h  = springe nach links
<strg>w+j  = springe nach unten
<strg>w+k  = springe nach oben
<strg>w+l  = springe nach rechts

<strg>w+c  = close the active windows
<strg>w+o  = close all windows except the active one

<strgw>    = Verteile alle Fenster gleich
<strg>w+_  = Maximize height of the active window
<strg>w+|  = Maximize width of the active window
[N]<strg>w+_ = Set active window height to [N] rows
[N]<strg>w+| = Set active window width to [N] columns
