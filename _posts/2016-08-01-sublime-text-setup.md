---
layout: post
title: Sublime Text 3 set-up
date: 2016-08-01 11:15:02.000000000 +02:00
categories: []
tags:
- SublimeText
- Code
- GitHub
status: publish
type: post
published: true
---

## Preamble

I am a big fan of Sublime Text! It is a lightweight text editor, inexpensive license, and with contributions by hundreds of users, highly extensible and customizable. From a practical perspective, I prefer to use it instead of IDEs, such as Jupyter or RStudio, because I also write a lot of little bash/shell scripts or just one-liners embedded in markdown (my projects notebooks). Also, the [pipeline](https://github.com/adomingues/NGSpipe2go) I am using is based on groovy. Sometimes I write code in all 4 languages in a single day, and thus it is easy to see why I prefer a single development environment instead of having to memorize different shortcuts/layouts. Personally it makes my life easier. Also, I love the multi-line editing features of sublime text and the ability to search within projects, etc.

Recently I upgrade to version 3, and re-installed my most used packages. I will leave the list here for future reference, and in case someone else is interested.


## Packages

Installed packages:

- [package control](https://packagecontrol.io/installation), to manage all packages
- [AcademicMarkdown](https://github.com/mangecoeur/AcademicMarkdown), neat highlighting of markdown syntax.
- MarkdownEditing
- All [Autocomplete](https://github.com/alienhard/SublimeAllAutocomplete) auto complete using matches from any of the open files.
- [BracketHighlighter](https://github.com/facelessuser/BracketHighlighter) extends the default bracket highlighting from sublime text.
- [SublimeKnitr](https://github.com/andrewheiss/SublimeKnitr), requires R-Box, SendREPL and LaTeXing. knitr Markdown and LaTeX support.
- [LaTeX-cwl](https://packagecontrol.io/packages/LaTeX-cwl), LateX commands auto-complete
- [Spell checking](https://www.sublimetext.com/docs/3/spell_checking.html), because tipos :)
- [markdown-preview](https://github.com/revolunet/sublimetext-markdown-preview), build and preview markdown files in Sublime.
- [SideBarEnhancements](https://packagecontrol.io/packages/SideBarEnhancements), copy, delete, rename and other file opearations from your side bar.
- [SublimeLinter](https://packagecontrol.io/packages/SublimeLinter), verification of code quality for:
	- [lintr](https://github.com/jimhester/SublimeLinter-contrib-lintr), R
	- [pep8](https://github.com/SublimeLinter/SublimeLinter-pep8), Python
	- [shellcheck](https://github.com/SublimeLinter/SublimeLinter-shellcheck), Shell
- [R_comments](https://packagecontrol.io/packages/R_comments), easy insertion of nicely formatted R comments.
- [R-snippets](http://www.jvcasillas.com/code/projects/R-snippets), collection of R snippets.
- [pythonpep8autoformat](https://bitbucket.org/StephaneBunel/pythonpep8autoformat), formats old code with pep8 rules.
- [python3](https://github.com/petervaro/python), syntax highlighting.
- [SublimeGit](https://github.com/SublimeGit/SublimeGit/)
- [carlcalderon sublime color schemes](https://github.com/carlcalderon/sublime-color-schemes), I prefer not so dark schemes, and use the Tyrann Kim or the Tyrann Alex.

From the above the single most important one is `SendREPL` which allows me to send commands straight from the editor to the terminal (with `tmux`) with a keystroke `ctrl+[enter]`. It does not matter if in `tmux` there is an R terminal, python console, or pure ol' bash. This flexibility is precious.


## Extras

`AcademicMarkdown` code blocks do not highlight code in blocks labelled "bash", but only has "shell", or "sh". This is in an [issue](https://github.com/mangecoeur/AcademicMarkdown/issues/12) when converting to html via pandoc. To solve this, I simple followed [these instructions](http://www.sublimetext.com/docs/3/packages.html), and modified locally the file [AcademicMarkdown.tmLanguage](https://github.com/mangecoeur/AcademicMarkdown/blob/3e7ff4bf7498bbbfe49650cfcfe265a7bfe06e66/AcademicMarkdown.tmLanguage) from:


```xml
		<key>fenced-shell</key>
		<dict>
		    <key>begin</key>
		    <string>^(\s*[`~]{3,})(sh|shell)\s*$</string>
		    <key>end</key>
		    <string>^(\1)\n</string>
		    <key>name</key>
		    <string>markup.raw.block.markdown markup.raw.block.fenced.markdown</string>
		    <key>patterns</key>
		    <array>
		        <dict>
		            <key>include</key>
		            <string>source.shell</string>
		        </dict>
		    </array>
		</dict>
```

to

```xml
		<key>fenced-shell</key>
		<dict>
		    <key>begin</key>
		    <string>^(\s*[`~]{3,})(sh|shell|bash)\s*$</string>
		    <key>end</key>
		    <string>^(\1)\n</string>
		    <key>name</key>
		    <string>markup.raw.block.markdown markup.raw.block.fenced.markdown</string>
		    <key>patterns</key>
		    <array>
		        <dict>
		            <key>include</key>
		            <string>source.shell</string>
		        </dict>
		    </array>
		</dict>
```

Note the inclusion of "bash" in the 4th line, and it now highlights fenced code labeled as "bash". I am very proud of myself. It was fixed in the repo and a [merge request](https://github.com/mangecoeur/AcademicMarkdown/pull/19) sent in GitHub.

I was also having a hard-time getting the "comment code" shortcut to work. This is a know bug that should have been solved in my version (Build 3114) bit it isn't. Adding this to the use keybindings file [solved](http://stackoverflow.com/questions/17742781/keyboard-shortcut-to-comment-lines-in-sublime-text-3) the issue:

```json
{ "keys": ["ctrl+7"], "command": "toggle_comment", "args": { "block": false } },
{ "keys": ["ctrl+shift+7"], "command": "toggle_comment", "args": { "block": true } }
```


### Groovy

There is some bug/feature in which lines are comment with "*/ /*", or something like that, rather than with "//", which I (and apparently [others](https://gist.github.com/ddeyoung/5502723)) prefer. The solution is [here](http://stackoverflow.com/a/24577721/1274242).