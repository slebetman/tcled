tcled
=====

Pure Tcl Console Text Editor
----------------------------

Features:
---------
- Basic syntax highlighting. The highlighting is line based and can easily cope with Tcl style # comments but because it is line based it can't cope with multiline C style /* comments */
- Implements search, goto and save.
- Handles Home and End keys.
- Changed the tab to 4 characters (can be easily modified if you prefer 8 or other values).
- Auto resizes the editor to fit the terminal window.
- Implements a simple auto-indenting. When inserting a newline by pressing the Enter or Return key the leading whitespace of the previous line is copied and automatically inserted.
- Converts spaces to tabs when pasting text.
- Implements key rate limiting for control characters and escape sequences. This is to improve responsiveness especially on slow machines/connections so that you don't accidentally "over-delete" when you press the delete key for too long.
- Implements undo and redo.
- Implements suspending and resuming the editing session.
- Implements tab completion based on words already in the current document.
- Supports CTags's tags file (if one is found) so you can look up function and variable definitions.

Usage:
------
- Arrow keys : Moves the cursor around. Typing anything inserts text at the cursor.
- Backspace : Deletes the character before the cursor.
- Delete : Deletes the character behind the cursor.
- Home : Moves the cursor to the first non-whitespace character on the current line. Pressing it a second time moves the cursor to the beginning of the line.
- End : Moves the cursor to the end of the line.
- Page Up and Page Down : Moves the cursor backwards and forwards one full page at a time.

Basically the usual navigation keys behaves as expected. The "^" character below denotes pressing the Ctrl key.
- ^a : Moves the cursor to the beginning of the line.
- ^c : Exits the program.
- ^d : Deletes the current line.
- ^e : Moves the cursor to the End of the line.
- ^f : Find/Search. The search pattern is regexp based so characters like ".", "(" and "[" needs to be escaped.
- F3 : Repeat the previous search.
- ^g : Goto line number. If you type "here" as the line number you will goto the current line. Since goto keeps a history of all previous gotos the "here" index is useful for bookmarking the current line.
- ^o : Page Up. Moves the cursor backwards one full page.
- ^p : Page Down. Moves the cursor forwards one full page.
- ^q : Quits/Exits the program. Ask to save the file if buffer is modified.
- ^s : Save the file.
- ^z : Undo the previous edit.
- ^y : Redo the last undo.
- ^w : Suspend the session, optionally save the file and exit. The suspended session is saved to a file with a .tsuspend extension. Opening this file will resume where you left off.
- Tab : When typing autocompletes the current word.
- ^Down Arrow : Go to definition of word under cursor (if found in tags file).
- ^Up Arrow : Return from definition.

Command Line Arguments:
-----------------------
- -s filename : Append the syntax rules defined in a file to the current list of syntax rules.
- -S filename : Replace the current syntax rules with ones defined in the file.
- -f extension : Force the syntax highlighter to use the rules for the given extension.
- -G line_number : Open file and go to line.
- -F regexp : Open file and executes a find/search.
- -define variable value : Allows you to modify global variables.

Code:
-----
The control character and escape sequence handling have been re-written to be more general and to report
unhandled cases. This is to make it easier to add new features to the code. For example, if you want to
implement a feature and bind it to ^k just run the editor and press ^k. It will tell you "Unhandled control
character:0xb" so that you know you should add the code as a \u000b case in handleControls. The same goes
for escape sequences. For example, pressing F12 will generate the message "Unhandled sequence:[24~"
