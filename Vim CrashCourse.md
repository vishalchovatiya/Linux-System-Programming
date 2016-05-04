Command 		Detail

#Beginner

i 			Insert mode. Type ESC to return to Normal mode.

x 			Delete the char under the cursor
:wq 			Save and Quit (:w save, :q quit)
dd 			(and copy) the current line
p 			Paste
hjkl 			cursor move (????). Hint: j looks like a down arrow.
:help <command> 	Show help about <command>.

=>>> Basic Moves

:/pattern 		Search for pattern in current file
0 			Go to the first column
$ 			Go to the end of line
gg 			Go to the start of the file
G 			Go to last line
:%s/text to search for/text to replace/g	Search text occurrences in file and replace all with text given

=>>> Load / Save / Quit File

:w 			Save current file
:saveas <path/to/file> 	Save to <path/to/file>
:x, ZZ or :wq 		Save and quit (:x only save if necessary)
:q!

=>>> Undo /Redo

u 			Undo
Ctrl+r 			Redo

=>>> Copy /Paste

p			paste before, remember p is paste after current position.

yy			copy the current line, easier but equivalent to ddP

v			Start Visual selection from current cursor position (Mostly used to copy multiple line by select area press yy and then paste by pressing p)

=>>> Put the line numbers in each line (Better)

:set number

=>>> Copy/Paste multiple line (Stronger)

Step 1 Go to address from where you want to start copy.

Step 2 Go to Normal mode by pressing ESC and then v (visual selection).

Step 3 Go up and down until copy area complete.

Step 4 Press yy which is copy(yank) the selected area(press ‘y’ key twice).

Step 5 Go with cursor where you want to paste copied data and press p.

=>>> Auto Detecting/Completing Words (Faster)

In Insert mode, just type the start of a word, then type Ctrl+p, just see the magic…

=>>> Use Mouse in Vim

:set mouse=a

=>>> Open File And Go To Specific Function or Line Number


1). Go to specific line number

vim fileName +LineNumber

i.e. vim main.c +3


2). Go to specific function

vim filename +/pattern

i.e. vim main.c +/main

3).Go to specific line number if your file is already opened

Type :linenumber in Normal mode and press enter, you will be at your desired location 

For example    :42

=>>> Commenting or Editing multiple line in single stroke

Step 1 Go to Normal mode by pressing ESC key.

Step 2 Press Ctrl+v (visual block selection) from where you want to edit.

Step 3 Move cursor up and down to number of line you want to edit and press I// .

Step 4 Press ESC again and wait for second it will comment all the selected line or you can say add // in front of each selected line.

Typically:  ESC, Ctrl+v, ?or?,  I<PatternToAdd>, [ESC]

=>>> Align code in curly braces (indent)

Step 1 Go inside the culy braces in which you want to align code properly.

Step 2 Go to Normal mode by pressing ESC then press =.

Step 3 Now press i.

Step 4 To see the magic press either { or } .

Typically:  ESC, =, i, {

=>>> Working with multiple file in Vim

This is very useful when we working with multiple file. For example, if you are coding in a file and at the same time you need another file for reference, so general technique is to use another terminal or open new window. But Vim have another nice option, by which you can split current working window in two sections and work on both file at the same time. 

Split window vertically or horizontally

In Normal mode, use command

:vs filename         ———Split window vertically

:sp filename         ———Split window horizontally

Note:- use Ctrl+w to shuffle between two opened section in Normal mode.
