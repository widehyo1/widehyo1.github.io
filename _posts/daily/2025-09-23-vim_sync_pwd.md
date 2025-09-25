---
layout: post
title: terminal buffer를 통한 pwd sync
subtitle: terminal buffer를 통한 pwd sync
tags: [vim]
comments: true
author: widehyo
---

```vim
" bufid 변수에 terminal 버퍼의 ID를 지정
let terminal_bufid = 3  " 예시: terminal buffer의 bufid

" 현재 작업 중인 디렉토리 가져오기
let current_dir = getcwd()

" terminal buffer로 cd 명령을 보내기
" :call 함수는 버퍼에 명령을 실행하게 해줌
execute 'buffer ' . terminal_bufid
call feedkeys("i" . "cd " . current_dir . "\n", 'n')
```


call feedkeys('a')
call feedkeys('a') | call feedkeys('cd ' . '/home/widehyo' . '\n')
call feedkeys('a') | call feedkeys('cd ' . '/home/widehyo') | call feedkeys('\<CR>')
call feedkeys('a') | call feedkeys('cd ' . '/home/widehyo') | call term_sendkeys(2, '\r')

call term_sendkeys(2, 'cd ' . '/home/widehyo' . '\<CR>')
call term_sendkeys(2, 'cd ' . '/home/widehyo')
call term_sendkeys(buf, 'cd ' . '/home/widehyo')

						terminal-autoshelldir
This can be used to pass the current directory from a shell to Vim.
Put this in your .vimrc: 
	def g:Tapi_lcd(_, path: string)
	    if isdirectory(path)
		execute 'silent lcd ' .. fnameescape(path)
	    endif
	enddef

And, in a bash init file: 
	if [[ -n "$VIM_TERMINAL" ]]; then
	    PROMPT_COMMAND='_vim_sync_PWD'
	    function _vim_sync_PWD() {
		printf '\033]51;["call", "Tapi_lcd", "%q"]\007' "$PWD"
	    }
	fi


4. Using a JSON or JS channel					*channel-use*

If mode is JSON then a message can be sent synchronously like this: >
    let response = ch_evalexpr(channel, {expr})
This awaits a response from the other side.

When mode is JS this works the same, except that the messages use
JavaScript encoding.  See |js_encode()| for the difference.

To send a message, without handling a response or letting the channel callback
handle the response: >
    call ch_sendexpr(channel, {expr})

To send a message and letting the response handled by a specific function,
asynchronously: >
    call ch_sendexpr(channel, {expr}, {'callback': Handler})

Vim will match the response with the request using the message ID.  Once the
response is received the callback will be invoked.  Further responses with the
same ID will be ignored.  If your server sends back multiple responses you
need to send them with ID zero, they will be passed to the channel callback.

The {expr} is converted to JSON and wrapped in an array.  An example of the
message that the receiver will get when {expr} is the string "hello":
	[12,"hello"] ~

The format of the JSON sent is:
    [{number},{expr}]

In which {number} is different every time.  It must be used in the response
(if any):

    [{number},{response}]

This way Vim knows which sent message matches with which received message and
can call the right handler.  Also when the messages arrive out of order.

A newline character is terminating the JSON text.  This can be used to
separate the read text.  For example, in Python:
	splitidx = read_text.find('\n')
	message = read_text[:splitidx]
	rest = read_text[splitidx + 1:]

The sender must always send valid JSON to Vim.  Vim can check for the end of
the message by parsing the JSON.  It will only accept the message if the end
was received.  A newline after the message is optional.

When the process wants to send a message to Vim without first receiving a
message, it must use the number zero:
    [0,{response}]

Then channel handler will then get {response} converted to Vim types.  If the
channel does not have a handler the message is dropped.

It is also possible to use ch_sendraw() and ch_evalraw() on a JSON or JS
channel.  The caller is then completely responsible for correct encoding and
decoding.

```vim
function! PrintTerminalBuffer()
  for listed_buffer in filter(getbufinfo(), 'v:val.listed')
    let bufnr = listed_buffer.bufnr
    let buftype = getbufvar(bufnr, '&buftype')
    let buftype = (buftype == '' ? 'normal' : buftype)
    if buftype == 'terminal'
      echo 'execute buffer! ' .. bufnr
      echomsg listed_buffer
      echomsg getbufvar(bufnr, "&")
      return
    endif
  endfor

  execute 'terminal! ++curwin'
endfunction
```

getbufvar({buf}, {varname} [, {def}])				*getbufvar()*
		The result is the value of option or local buffer variable
		{varname} in buffer {buf}.  Note that the name without "b:"
		must be used.
		The {varname} argument is a string.
		When {varname} is empty returns a |Dictionary| with all the
		buffer-local variables.
		When {varname} is equal to "&" returns a |Dictionary| with all
		the buffer-local options.
		Otherwise, when {varname} starts with "&" returns the value of
		a buffer-local option.
		This also works for a global or buffer-local option, but it
		doesn't work for a global variable, window-local variable or
		window-local option.
		For the use of {buf}, see |bufname()| above.
		When the buffer or variable doesn't exist {def} or an empty
		string is returned, there is no error message.
		Examples: >
			:let bufmodified = getbufvar(1, "&mod")
			:echo "todo myvar = " . getbufvar("todo", "myvar")

<		Can also be used as a |method|: >
			GetBufnr()->getbufvar(varname)


      echomsg getbufinfo(bufnr)
      echomsg getbufvar(bufnr, "&")
{
  "lnum": 0,
  "bufnr": 7,
  "variables": {
    "changedtick": 2
  },
  "popups": [],
  "name": "!/bin/bash",
  "changed": 1,
  "lastused": 1758800967,
  "loaded": 1,
  "windows": [
    1004
  ],
  "hidden": 0,
  "listed": 1,
  "changedtick": 2,
  "linecount": 1
}
{'indentkeys': '0{,0},0),0],:,0#,!^F,o,O,e', 'quoteescape': '\', 'key': '', 'makeprg': 'make', 'lisp': 0, 'keywordprg': 'man', 'path': '.,/usr/include,,', 'spelloptions': '', 'filetype': '', 'binary': 0, 'omnifunc': '', 'matchpairs': '(:),{:},[:]', 'keymap': '', 'cindent': 0, 'errorformat': '%*[^"]"%f"%*\D%l: %m,"%f"%*\D%l: %m,%-G%f:%l: (Each undeclared identifier is reported only once,%-G%f:%l: for each function it appears in.),%-GIn file included from %f:%l:%c:,%-GIn file included from %f:%l:%c\,,%-GIn file included from %f:%l:%c,%-GIn file included from %f:%l,%-G%*[ ]from %f:%l:%c,%-G%*[ ]from %f:%l:,%-G%*[ ]from %f:%l\,,%-G%*[ ]from %f:%l,%f:%l:%c:%m,%f(%l):%m,%f:%l:%m,"%f"\, line %l%*\D%c%*[^ ] %m,%D%*\a[%*\d]: Entering directory %*[`'']%f'',%X%*\a[%*\d]: Leaving directory %*[`'']%f'',%D%*\a: Entering directory %*[`'']%f'',%X%*\a: Leaving directory %*[`'']%f'',%DMaking %*\a in %f,%f|%l| %m', 'varsofttabstop': '', 'modifiable': 0, 'autoread': 1, 'modeline': 0, 'imsearch': -1, 'cinwords': 'if,else,while,do,for,switch', 'complete': '.,w,b,u,t,i', 'synmaxcol': 3000, 'undofile': 0, 'preserveindent': 0, 'fixendofline': 1, 'tabstop': 4, 'smartindent': 1, 'include': '^\s*#\s*include', 'undolevels': 1000, 'softtabstop': 0, 'tags': './.tags;/', 'commentstring': '/*%s*/', 'balloonexpr': '', 'termwinscroll': 10000, 'tagfunc': '', 'iskeyword': '@,48-57,_,192-255', 'textmode': 0, 'endofline': 1, 'fileencoding': '', 'equalprg': '', 'textwidth': 0, 'bomb': 0, 'suffixesadd': '', 'readonly': 0, 'nrformats': 'bin,octal,hex', 'spelllang': 'en', 'expandtab': 1, 'define': '^\s*#\s*define', 'shortname': 0, 'spellfile': '', 'bufhidden': '', 'tagcase': 'followic', 'buflisted': 1, 'wrapmargin': 0, 'vartabstop': '', 'lispwords': 'defun,define,defmacro,set!,lambda,if,case,let,flet,let*,letrec,do,do*,define-syntax,let-syntax,letrec-syntax,destructuring-bind,defpackage,defparameter,defstruct,deftype,defvar,do-all-symbols,do-external-symbols,do-symbols,dolist,dotimes,ecase,etypecase,eval-when,labels,macrolet,multiple-value-bind,multiple-value-call,multiple-value-prog1,multiple-value-setq,prog1,progv,typecase,unless,unwind-protect,when,with-input-from-string,with-open-file,with-open-stream,with-output-to-string,with-package-iterator,define-condition,handler-bind,handler-case,restart-bind,restart-case,with-simple-restart,store-value,use-value,muffle-warning,abort,continue,with-slots,with-slots*,with-accessors,with-accessors*,defclass,defmethod,print-unreadable-object', 'virtualedit': '', 'fileformat': 'unix', 'formatoptions': 'tcq', 'formatprg': '', 'infercase': 0, 'completefunc': '', 'thesaurus': '', 'modified': 0, 'spellcapcheck': '[.?!]\_[\])''"^I ]\+', 'thesaurusfunc': '', 'grepprg': 'grep -n $* /dev/null', 'cinkeys': '0{,0},0),0],:,0#,!^F,o,O,e', 'dictionary': '', 'swapfile': 0, 'formatlistpat': '^\s*\d\+[\]:.)}\t ]\s*', 'buftype': 'terminal', 'syntax': '', 'formatexpr': '', 'indentexpr': '', 'comments': 's1:/*,mb:*,ex:*/,://,b:#,:%,:XCOMM,n:>,fb:-', 'cryptmethod': 'blowfish2', 'copyindent': 0, 'iminsert': 0, 'makeencoding': '', 'cinoptions': '', 'shiftwidth': 4, 'backupcopy': 'auto', 'includeexpr': '', 'autoindent': 0}

{'lnum': 0, 'bufnr': 7, 'variables': {'changedtick': 2}, 'popups': [], 'name': '!/bin/bash', 'changed': 1, 'lastused': 1758800967, 'loaded': 1, 'windows': [1004], 'hidden': 0, 'listed': 1, 'changedtick': 2, 'linecount': 1}
{
  "indentkeys": "0{,0},0),0],:,0#,!^F,o,O,e",
  "quoteescape": "\\",
  "key": "",
  "makeprg": "make",
  "lisp": 0,
  "keywordprg": "man",
  "path": ".,/usr/include,,",
  "spelloptions": "",
  "filetype": "",
  "binary": 0,
  "omnifunc": "",
  "matchpairs": "(:),{:},[:]",
  "keymap": "",
  "cindent": 0,
  "varsofttabstop": "",
  "modifiable": 0,
  "autoread": 1,
  "modeline": 0,
  "imsearch": -1,
  "cinwords": "if,else,while,do,for,switch",
  "complete": ".,w,b,u,t,i",
  "synmaxcol": 3000,
  "undofile": 0,
  "preserveindent": 0,
  "fixendofline": 1,
  "tabstop": 4,
  "smartindent": 1,
  "include": "^\\s*#\\s*include",
  "undolevels": 1000,
  "softtabstop": 0,
  "tags": "./.tags;/",
  "commentstring": "/*%s*/",
  "balloonexpr": "",
  "termwinscroll": 10000,
  "tagfunc": "",
  "iskeyword": "@,48-57,_,192-255",
  "textmode": 0,
  "endofline": 1,
  "fileencoding": "",
  "equalprg": "",
  "textwidth": 0,
  "bomb": 0,
  "suffixesadd": "",
  "readonly": 0,
  "nrformats": "bin,octal,hex",
  "spelllang": "en",
  "expandtab": 1,
  "define": "^\\s*#\\s*define",
  "shortname": 0,
  "spellfile": "",
  "bufhidden": "",
  "tagcase": "followic",
  "buflisted": 1,
  "wrapmargin": 0,
  "vartabstop": "",
  "lispwords": "defun,define,defmacro,set!,lambda,if,case,let,flet,let*,letrec,do,do*,define-syntax,let-syntax,letrec-syntax,destructuring-bind,defpackage,defparameter,defstruct,deftype,defvar,do-all-symbols,do-external-symbols,do-symbols,dolist,dotimes,ecase,etypecase,eval-when,labels,macrolet,multiple-value-bind,multiple-value-call,multiple-value-prog1,multiple-value-setq,prog1,progv,typecase,unless,unwind-protect,when,with-input-from-string,with-open-file,with-open-stream,with-output-to-string,with-package-iterator,define-condition,handler-bind,handler-case,restart-bind,restart-case,with-simple-restart,store-value,use-value,muffle-warning,abort,continue,with-slots,with-slots*,with-accessors,with-accessors*,defclass,defmethod,print-unreadable-object",
  "virtualedit": "",
  "fileformat": "unix",
  "formatoptions": "tcq",
  "formatprg": "",
  "infercase": 0,
  "completefunc": "",
  "thesaurus": "",
  "modified": 0,
  "thesaurusfunc": "",
  "grepprg": "grep -n $* /dev/null",
  "cinkeys": "0{,0},0),0],:,0#,!^F,o,O,e",
  "dictionary": "",
  "swapfile": 0,
  "formatlistpat": "^\\s*\\d\\+[\\]:.)}\\t ]\\s*",
  "buftype": "terminal",
  "syntax": "",
  "formatexpr": "",
  "indentexpr": "",
  "comments": "s1:/*,mb:*,ex:*/,://,b:#,:%,:XCOMM,n:>,fb:-",
  "cryptmethod": "blowfish2",
  "copyindent": 0,
  "iminsert": 0,
  "makeencoding": "",
  "cinoptions": "",
  "shiftwidth": 4,
  "backupcopy": "auto",
  "includeexpr": "",
  "autoindent": 0
}


term_getansicolors({buf})				*term_getansicolors()*
		Get the ANSI color palette in use by terminal {buf}.
		Returns a List of length 16 where each element is a String
		representing a color in hexadecimal "#rrggbb" format.
		Also see |term_setansicolors()| and |g:terminal_ansi_colors|.
		If neither was used returns the default colors.

		{buf} is used as with |term_getsize()|.  If the buffer does not
		exist or is not a terminal window, an empty list is returned.

		Can also be used as a |method|: >
			GetBufnr()->term_getansicolors()

<		{only available when compiled with GUI enabled and/or the
		|+termguicolors| feature}



['process 1744 run']

echomsg term_getansicolors(7)

['#000000', '#e00000', '#00e000', '#e0e000', '#0000e0', '#e000e0', '#00e0e0', '#e0e0e0', '#808080', '#ff4040', '#40ff40', '#ffff40', '#4040ff', '#ff40ff', '#40ffff', '#ffffff']



string					*string* *String* *expr-string* *E114*
------
"string"		string constant		*expr-quote*

Note that double quotes are used.

A string constant accepts these special characters:
\...	three-digit octal number (e.g., "\316")
\..	two-digit octal number (must be followed by non-digit)
\.	one-digit octal number (must be followed by non-digit)
\x..	byte specified with two hex numbers (e.g., "\x1f")
\x.	byte specified with one hex number (must be followed by non-hex char)
\X..	same as \x..
\X.	same as \x.
\u....	character specified with up to 4 hex numbers, stored according to the
	current value of 'encoding' (e.g., "\u02a4")
\U....	same as \u but allows up to 8 hex numbers.
\b	backspace <BS>
\e	escape <Esc>
\f	formfeed 0x0C
\n	newline <NL>
\r	return <CR>
\t	tab <Tab>
\\	backslash
\"	double quote
\<xxx>	Special key named "xxx".  e.g. "\<C-W>" for CTRL-W.  This is for use
	in mappings, the 0x80 byte is escaped.
	To use the double quote character it must be escaped: "<M-\">".
	Don't use <Char-xxxx> to get a UTF-8 character, use \uxxxx as
	mentioned above.
\<*xxx>	Like \<xxx> but prepends a modifier instead of including it in the
	character.  E.g. "\<C-w>" is one character 0x17 while "\<*C-w>" is four
	bytes: 3 for the CTRL modifier and then character "W".

Note that "\xff" is stored as the byte 255, which may be invalid in some
encodings.  Use "\u00ff" to store character 255 according to the current value
of 'encoding'.

Note that "\000" and "\x00" force the end of the string.




feedkeys({string} [, {mode}])				*feedkeys()*
		Characters in {string} are queued for processing as if they
		come from a mapping or were typed by the user.

		By default the string is added to the end of the typeahead
		buffer, thus if a mapping is still being executed the
		characters come after them.  Use the 'i' flag to insert before
		other characters, they will be executed next, before any
		characters from a mapping.

		The function does not wait for processing of keys contained in
		{string}.

		To include special keys into {string}, use double-quotes
		and "\..." notation |expr-quote|. For example,
		feedkeys("\<CR>") simulates pressing of the <Enter> key. But
		feedkeys('\<CR>') pushes 5 characters.
		A special code that might be useful is <Ignore>, it exits the
		wait for a character without doing anything.  *<Ignore>*

		{mode} is a String, which can contain these character flags:
		'm'	Remap keys. This is default.  If {mode} is absent,
			keys are remapped.
		'n'	Do not remap keys.
		't'	Handle keys as if typed; otherwise they are handled as
			if coming from a mapping.  This matters for undo,
			opening folds, etc.
		'L'	Lowlevel input.  Only works for Unix or when using the
			GUI. Keys are used as if they were coming from the
			terminal.  Other flags are not used.  *E980*
			When a CTRL-C interrupts and 't' is included it sets
			the internal "got_int" flag.
		'i'	Insert the string instead of appending (see above).
		'x'	Execute commands until typeahead is empty.  This is
			similar to using ":normal!".  You can call feedkeys()
			several times without 'x' and then one time with 'x'
			(possibly with an empty {string}) to execute all the
			typeahead.  Note that when Vim ends in Insert mode it
			will behave as if <Esc> is typed, to avoid getting
			stuck, waiting for a character to be typed before the
			script continues.
			Note that if you manage to call feedkeys() while
			executing commands, thus calling it recursively, then
			all typeahead will be consumed by the last call.
		'!'	When used with 'x' will not end Insert mode. Can be
			used in a test when a timer is set to exit Insert mode
			a little later.  Useful for testing CursorHoldI.

		Return value is always 0.

		Can also be used as a |method|: >
			GetInput()->feedkeys()



call feedkeys("ils\<CR>")

드디어 찾았다!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!


call feedkeys("i\<C-u>ls\<CR>")


```vim
function! SyncTerminalBuffer()
  for listed_buffer in filter(getbufinfo(), 'v:val.listed')
    let bufnr = listed_buffer.bufnr
    let buftype = getbufvar(bufnr, '&buftype')
    let buftype = (buftype == '' ? 'normal' : buftype)
    if buftype == 'terminal'
      execute 'buffer! ' .. bufnr
      call feedkeys("i\<C-u>ls\<CR>")
      return
    endif
  endfor
endfunction

call SyncTerminalBuffer()


```


							*winnr()*
winnr([{arg}])	The result is a Number, which is the number of the current
		window.  The top window has number 1.
		Returns zero for a popup window.

		The optional argument {arg} supports the following values:
			$	the number of the last window (the window
				count).
			#	the number of the last accessed window (where
				|CTRL-W_p| goes to).  If there is no previous
				window or it is in another tab page 0 is
				returned.
			{N}j	the number of the Nth window below the
				current window (where |CTRL-W_j| goes to).
			{N}k	the number of the Nth window above the current
				window (where |CTRL-W_k| goes to).
			{N}h	the number of the Nth window left of the
				current window (where |CTRL-W_h| goes to).
			{N}l	the number of the Nth window right of the
				current window (where |CTRL-W_l| goes to).
		The number can be used with |CTRL-W_w| and ":wincmd w"
		|:wincmd|.
		Also see |tabpagewinnr()| and |win_getid()|.
		Examples: >
			let window_count = winnr('$')
			let prev_window = winnr('#')
			let wnum = winnr('3k')

<		Can also be used as a |method|: >
			GetWinval()->winnr()

bufwinid({buf})						*bufwinid()*
		The result is a Number, which is the |window-ID| of the first
		window associated with buffer {buf}.  For the use of {buf},
		see |bufname()| above.  If buffer {buf} doesn't exist or
		there is no such window, -1 is returned.  Example: >

	echo "A window containing buffer 1 is " . (bufwinid(1))
<
		Only deals with the current tab page.

		Can also be used as a |method|: >
			FindBuffer()->bufwinid()

bufwinnr({buf})						*bufwinnr()*
		Like |bufwinid()| but return the window number instead of the
		|window-ID|.
		If buffer {buf} doesn't exist or there is no such window, -1
		is returned.  Example: >

	echo "A window containing buffer 1 is " . (bufwinnr(1))

<		The number can be used with |CTRL-W_w| and ":wincmd w"
		|:wincmd|.

		Can also be used as a |method|: >
			FindBuffer()->bufwinnr()




{'lnum': 157, 'bufnr': 7, 'variables': {'changedtick': 3}, 'popups': [], 'name': '!/bin/bash', 'changed': 0, 'lastused': 1758805212, 'loaded': 0, 'windows': [], 'hidden': 0, 'listed': 1, 'changedtick': 3, 'linecount': 1}


echo bufwinnr(7) " returns -1
echo bufwinid(7) " returns -1




win_findbuf({bufnr})					*win_findbuf()*
		Returns a |List| with |window-ID|s for windows that contain
		buffer {bufnr}.  When there is none the list is empty.

		Can also be used as a |method|: >
			GetBufnr()->win_findbuf()


echo bufwinid(bufnr()) " 1011

win_gotoid({expr})		Number	go to window with ID {expr}


win_gotoid({expr})					*win_gotoid()*
		Go to window with ID {expr}.  This may also change the current
		tabpage.
		Return TRUE if successful, FALSE if the window cannot be found.

		Can also be used as a |method|: >
			GetWinid()->win_gotoid()


echo bufnr() " 43
echo win_findbuf(43) " []
echo win_findbuf(43) " [1039]

call setbufvar(43, 'winid', 1039)
echo getbufvar(43, 'winid') " 1039


win_execute({id}, {command} [, {silent}])		*win_execute()*
		Like `execute()` but in the context of window {id}.
		The window will temporarily be made the current window,
		without triggering autocommands or changing directory.  When
		executing {command} autocommands will be triggered, this may
		have unexpected side effects.  Use |:noautocmd| if needed.
		Example: >
			call win_execute(winid, 'set syntax=python')
<		Doing the same with `setwinvar()` would not trigger
		autocommands and not actually show syntax highlighting.

							*E994*
		Not all commands are allowed in popup windows.
		When window {id} does not exist then no error is given and
		an empty string is returned.

		Can also be used as a |method|, the base is passed as the
		second argument: >
			GetCommand()->win_execute(winid)


win_id2win({expr})					*win_id2win()*
		Return the window number of window with ID {expr}.
		Return 0 if the window cannot be found in the current tabpage.

		Can also be used as a |method|: >
			GetWinid()->win_id2win()

echo win_id2win(1039) " 6
echo win_id2win(1039) " 0 after close window

1049

```vim
function! SyncTerminalBuffer()
  for listed_buffer in filter(getbufinfo(), 'v:val.listed')
    let bufnr = listed_buffer.bufnr
    let buftype = getbufvar(bufnr, '&buftype')
    let buftype = (buftype == '' ? 'normal' : buftype)
    if buftype == 'terminal'
      let term_winid = getbufvar(bufnr, 'winid')
      if win_id2win(term_winid) != 0
        call win_gotoid(term_winid)
      endif
      execute 'buffer! ' .. bufnr
      let pwd = getcwd()
      call feedkeys("i\<C-u>cd " .. pwd .. "\<CR>")
      break
    endif
  endfor
endfunction

call SyncTerminalBuffer()
```


						*CTRL-W_:*
CTRL-W :	Does the same as typing |:| - enter a command line.  Useful in a
		terminal window, where all Vim commands must be preceded with
		CTRL-W or 'termwinkey'.

						*'termwinkey'* *'twk'*
'termwinkey' 'twk'	string	(default "")
			local to window
	The key that starts a CTRL-W command in a terminal window.  Other keys
	are sent to the job running in the window.
	The <> notation can be used, e.g.: >
		:set termwinkey=<C-L>
<	The string must be one key stroke but can be multiple bytes.
	When not set CTRL-W is used, so that CTRL-W : gets you to the command
	line.  If 'termwinkey' is set to CTRL-L then CTRL-L : gets you to the
	command line.

You can use `CTRL-W :hide` to close the terminal window and make the buffer
hidden, the job keeps running.  The `:buffer` command can be used to turn the
current window into a terminal window.  If there are unsaved changes this
fails, use ! to force, as usual.


```lua
-- terminal 버퍼에서 pwd를 이용해 vim의 :pwd 동기화
vim.api.nvim_create_autocmd({ 'TermRequest' }, {
  desc = 'Handles OSC 7 dir change requests',
  callback = function(ev)
    if string.sub(ev.data.sequence, 1, 4) == '\x1b]7;' then
      local dir = string.gsub(ev.data.sequence, '\x1b]7;file://[^/]*', '')
      if vim.fn.isdirectory(dir) == 0 then
        vim.notify('invalid dir: '..dir)
        return
      end
      vim.api.nvim_buf_set_var(ev.buf, 'osc7_dir', dir)
    end
  end
})
```

```sh
_syncpwd() {
  printf '\e]51;["call","Tapi_SyncTerminalPwd","%s"]\x07' "$PWD"
}

export PROMPT_COMMAND='_syncnvimpwd; '"$PROMPT_COMMAND"

_syncnvimpwd() {
  if [[ "$PWD" != "$__osc7_prev_pwd" ]]; then
    printf '\033]7;file://%s%s\007' "$HOSTNAME" "$PWD"
    __osc7_prev_pwd="$PWD"
  fi
}
```


echo bufnr()
call setbufvar(1, 'osc7_dir', '/qwer')
echo getbufvar(1, 'osc7_dir')
echo isdirectory(getbufvar(1, 'osc7_dir'))
echomsg isdirectory(getbufvar(1, 'osc7_dir'))

```vim
function! Tapi_SyncOsc7_Dir(bufnum, arglist)
  call setbufvar(a:bufnum, 'osc7_dir', a:arglist)
endfunction

function! SyncTerminalPwd()
  let term_bufnr = bufnr()
  echomsg term_bufnr
  let osc7_dir = getbufvar(term_bufnr, 'osc7_dir')
  echomsg osc7_dir
  echomsg isdirectory(osc7_dir)
endfunction
```


```sh
_syncpwd() {
  printf '\e]51;["call","Tapi_SyncTerminalPwd","%s"]\x07' "$PWD"
}
alias syncpwd="_syncpwd"

__osc7_prev_pwd=""
_setosc7dir() {
  if [[ "$PWD" != "$__osc7_prev_pwd" ]]; then
    printf '\033]7;file://%s%s\007' "$HOSTNAME" "$PWD" # for neovim term request
    printf '\e]51;["call","Tapi_SetOsc7_Dir","%s"]\x07' "$PWD" # for vim terminal api
    __osc7_prev_pwd="$PWD"
  fi
}
alias setosc7dir="_setosc7dir"

export PROMPT_COMMAND='setosc7dir; '"$PROMPT_COMMAND"
```

```vim
function! Tapi_SetOsc7_Dir(bufnum, arglist)
  call setbufvar(a:bufnum, 'osc7_dir', a:arglist)
endfunction

function! SyncTerminalPwd()
  let term_bufnr = bufnr()
  echomsg term_bufnr
  let osc7_dir = getbufvar(term_bufnr, 'osc7_dir')
  echomsg osc7_dir
  echomsg isdirectory(osc7_dir)
endfunction
```



getbufvar({buf}, {varname} [, {def}])				*getbufvar()*
		The result is the value of option or local buffer variable
		{varname} in buffer {buf}.  Note that the name without "b:"
		must be used.
		The {varname} argument is a string.
		When {varname} is empty returns a |Dictionary| with all the
		buffer-local variables.
		When {varname} is equal to "&" returns a |Dictionary| with all
		the buffer-local options.
		Otherwise, when {varname} starts with "&" returns the value of
		a buffer-local option.
		This also works for a global or buffer-local option, but it
		doesn't work for a global variable, window-local variable or
		window-local option.
		For the use of {buf}, see |bufname()| above.
		When the buffer or variable doesn't exist {def} or an empty
		string is returned, there is no error message.
		Examples: >
			:let bufmodified = getbufvar(1, "&mod")
			:echo "todo myvar = " . getbufvar("todo", "myvar")

<		Can also be used as a |method|: >
			GetBufnr()->getbufvar(varname)

`/home/widehyo/.vim/util/common.vim`
```vim
function! Tapi_SyncTerminalPwd(bufnum, arglist)
  execute 'cd ' .. a:arglist
endfunction

function! Tapi_SetOsc7_Dir(bufnum, arglist)
  call setbufvar(a:bufnum, 'osc7_dir', a:arglist)
endfunction

function! SyncTerminalPwd()
  let term_bufnr = bufnr()
  let osc7_dir = getbufvar(term_bufnr, 'osc7_dir')
  if isdirectory(osc7_dir)
    echo 'osc7_dir: ' .. osc7_dir
    execute 'cd ' .. osc7_dir
  endif
endfunction
```

```vim
" TermOpen 이벤트에 대한 자동 명령
augroup TerminalKeymaps
  autocmd!
  autocmd TerminalOpen * call s:SetupTerminalKeymaps()
augroup END

function! s:SetupTerminalKeymaps() abort
  set hidden
  " <C-D> : 이전 버퍼로 전환
  execute 'nnoremap <buffer> <C-D> :buffer! #<CR>'

  " <leader>cd : osc7_dir으로 pwd 설정
  execute 'nnoremap <buffer> <leader>cd :call SyncTerminalPwd()<CR>'

  " 터미널 모드에서 <C-Q> → <C-\><C-n> : Normal 모드로 진입
  execute 'tnoremap <buffer> <C-Q> <C-\><C-n>'

endfunction
```
