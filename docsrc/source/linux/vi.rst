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