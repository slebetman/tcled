tcled
=====
Simple text editor with syntax hilighting in a single file tcl script!

This software is released under a simple 2-clause BSD license.

Pure Tcl Console Text Editor
----------------------------
This editor is based on Steve Redler's minimal console text editor
in Tcl (http://wiki.tcl.tk/11820). I took Steve's code and modified it to do
syntax hilighting. Since then it's had enough features added to it that it is
now good enough for me to use as my primary code editor.

Part of the reason I wrote this is because I got tired of console text editors
behaving very differently from regular GUI text editors. As such, this has key
bindings that's as close as possible to GUI text editors: ctrl-S to save files,
F3 to search, ctrl-G to go to line etc.

Another reason I wrote this was because I needed a decent editor, with syntax
hilighting, in environments that don't have compilers and where it's difficult
or impossible to install binaries correctly. Cisco routers and Tivo boxes for
example (in my case it was GreenPacket routers back when I was employed by
GreenPacket to develop software for their routers). Fortunately these routers
almost always include tcl and fortunately tcl is powerful enough that it's
possible to write a simple text editor without any additional modules.

Features:
---------
- Basic syntax highlighting. The highlighting is line based and can easily cope
  with Tcl style # comments but because it is line based it can't cope with
  multiline C style /* comments */
- "Normal" key bindings - emulate the behavior of GUI text editors as closely as
  possible so you don't need to learn yet another way of doing things.
- Implements a simple auto-indenting. When inserting a newline by pressing the
  Enter or Return key the leading whitespace of the previous line is copied and
  automatically inserted.
- Converts spaces to tabs when pasting text.
- Implements key rate limiting for control characters and escape sequences.
  This is to improve responsiveness especially on slow machines/connections so
  that you don't accidentally "over-delete" when you press the delete key for 
  too long.
- All the usual features you'd expect in a simple text editor: undo, redo, search,
  goto etc.
- Implements suspending and resuming the editing session.
- Implements tab completion based on words already in the current document.
- Supports CTags's tags file (if one is found) so you can look up function and
  variable definitions.
- Supports X/Gnome/Windows/Mac clipboard if available (requires Tk, no Tk
  widgets will be created just using the cross-platform clipboard support)
- Properly support mouse selection (note: This means that copy on X-selection
  no longer works. Use ^c to copy instead).
- Support mousewheel scrolling.
- Supports executing shell commands (see documentation for F2 below). The result
  of the shell command can then either be inserted directly into the document
  or copied to the clipboard. With this feature Unix is your IDE (as it was
  always meant to be)!
- Supports per-project config file.

Usage:
------
- `Arrow keys` : Moves the cursor around. Typing anything inserts text at the
  cursor.
- `Backspace` : Deletes the character before the cursor.
- `Delete` : Deletes the character behind the cursor.
- `Home` : Moves the cursor to the first non-whitespace character on the
  current line. Pressing it a second time moves the cursor to the beginning of
  the line.
- `End` : Moves the cursor to the end of the line.
- `Page Up` and `Page Down` : Moves the cursor backwards and forwards one full
  page at a time.
- `Shift + Arrow keys` : Select text.
- `Tab` while text is selected : Block indent.
- `Shift + Tab` while text is selected : Block de-indent.

Basically the usual navigation keys behaves as expected. In addition, you can use
the mousewheel to scroll (on xterm compatible terminals - which is most terminals).
The "^" character below denotes pressing the Ctrl key.

- `^a` : Moves the cursor to the beginning of the line.
- `^c` : Copies selected text to clipboard.
- `^d` : Deletes the current line.
- `^e` : Moves the cursor to the End of the line.
- `^f` : Find/Search. The search pattern is regexp based so characters like
  ".", "(" and "[" needs to be escaped.
- `^g` : Goto line number. If you type "here" as the line number you will goto
  the current line. Since goto keeps a history of all previous gotos the "here"
  index is useful for bookmarking the current line.
- `^o` : Page Up. Moves the cursor backwards one full page.
- `^p` : Page Down. Moves the cursor forwards one full page.
- `^q` : Quits/Exits the program. Ask to save the file if buffer is modified.
- `^r` : Reloads the file from disk.
- `^s` : Save the file.
- `^v` : Paste text from clipboard.
- `^w` : Suspend the session, optionally save the file and exit. The suspended
  session is saved to a file with a .tsuspend extension. Opening this file will
  resume where you left off.
- `^x` : Cut text to clipboard.
- `^y` : Redo the last undo.
- `^z` : Undo the previous edit.
- `^Down Arrow` : Go to definition of word under cursor (if found in tags file).
- `alt-shift->` : Go to definition (alternative for ^Down Arrow)
- `^Up Arrow` : Return from definition.
- `alt-shift-<` : Return form definition (alternative for ^Up Arrow)
- `^Left Arrow` : Go to previous word.
- `^Right Arrow` : Go to next word.
- `Tab` : When typing autocompletes the current word.
- `Alt + ;` : Toggle tab guides.
- `Alg + /` : Comment/uncomment selection
- `F2` : Execute a shell command
- `F3` : Repeat the previous search.


Command Line Arguments:
-----------------------
- `-s filename` : Append the syntax rules defined in a file to the current list
  of syntax rules.
- `-S filename` : Replace the current syntax rules with ones defined in the file.
- `-f extension` : Force the syntax highlighter to use the rules for the given
  extension.
- `-G line_number` : Open file and go to line.
- `+line_number` : Shortcut for -G line_number.
- `-F regexp` : Open file and executes a find/search.
- `-define variable value` : Allows you to modify global variables.

Code:
-----
The control character and escape sequence handling have been re-written to be
more general and to report unhandled cases. This is to make it easier to add
new features to the code. For example, if you want to implement a feature and
bind it to ^k just run the editor and press `^k`. It will tell you
`"Unhandled control character:0xb"` so that you know you should add the code as
a `\u000b` case in handleControls. The same goes for escape sequences. For
example, pressing `F12` will generate the message `"Unhandled sequence:[24~"`

Syntax Hilighting Rules:
------------------------
The rules for syntax hilighting are currently hardcoded in the file and
contained within the variable syntaxRules located at the top of the code. The
syntax rules is in the form:

    {filepattern} {{regexp} {formatting}..}

Comments (after #) are ignored. Syntax hilighting is line based so we can't
have multi-line rules like C-style comments.
If more than one rule applies to piece of text then the most encompassing rule
wins. 
For example for the text:

    "$example"
	
both the script variable (due to `$`) and the string rules `(".*?")` apply. 
However since the string rule encompasses the script variable rule then the 
string rule wins and the text is colored according to the string rule.

But within each rule the opposite is true. If the regexp matches a piece of 
text multiple times then the most specific match wins. For example for the Tcl 
variable regexp:

    {(?:set|append|incr) ([a-zA-Z_][a-zA-Z0-9_]*)}

the text:

    set x

matches twice. Once for `set x` and another time for `x`. Since `x` is more
specific then only it will be colored by the rule. This overcomes Tcl's lack of
look-behind in its regexp engine.

Formatting is defined by ANSI escape sequence. For example bright green is
`{1;32}`. The arrays `fg`, `bg` and `style` above makes it more convenient to
define the formatting. Using the previous example bright green may be written
as `{$style(bright);$fg(green)}`.

Also, due to the way the renderig engine works, the syntax hilighting rules
cannot distinguish between tabs and spaces. So for the purpose of writing the
syntax regexp `" "`, `"\s"`, `"\t"` and `"[[:space:]]"` are synonymous.

Config Files:
-------------
If the file `.tcled.conf` exists in the current directory or in any parent
directories then it will be read as a config file. This mechanism means that
you can have different configurations for different projects.

The syntax of the config file is simply a list of `setting` and `value` on
each line. Everything after a `#` symbol to the end of each line are considered
comments and will be ignored. Empty lines are ignored.

The following settings are available:

- `execShell` - Shell to use to execute shell commands (F2). Defaults to `/bin/bash`.
- `tabstop` - Number of spaces per tab. Default is `4`.
- `usetabs` - Use tabs or spaces. Default is `true` which means use tabs instead of spaces.
- `searchcase` - Is search case sensitive? Defaults to `false`.
- `tabGuide` - Turn on tab guide on or off. Defaults to `false` which turns it off.
- `filealias` - Maps a file extension to another for the purposes of syntax highlighting. This setting takes two arguments instead of one. You can specify multiple `filealias` in a config file.

Example config file:

    # Management mandates 8 spaces:
    usetabs false
	tabstop 8
	
	filealias ts js # use js syntax for typescript

