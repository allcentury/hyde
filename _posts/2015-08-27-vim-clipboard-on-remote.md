---
layout: post
title: Sharing your system clipboard
---

Today I ran into an issue that had me spinning my wheels for a little too long.  I'm making it a habit to write down these moments so a.) maybe someone out there might waste less time than I did and b.) I'm sure I'll have to come back to this at some point!

Today, I was ssh'd into a remote server and tried to yank something out of vim and into another vim running on my local machine.  To clarify:

On the remote(Ubuntu):
vim -> yank

On my local(OSX):
vim -> paste

I quickly realized that yanking on a remote machine does not mean it will end up on my system(OSX) clipboard. That would be too easy.  After some endless googling and bugging people in #vim #tmux and #ubuntu I got no where.  That was until I found [this post.](http://superuser.com/questions/403664/how-can-i-copy-and-paste-text-out-of-a-remote-vim-to-a-local-vim)

There's some additional set up beyond that post so I hope this to be all encompassing, here we go:

## OSX Set up

First you need to be using MacVim on OSX, I wasn't.  MacVim comes with nearly all the extra packages, but the most important one here is +clipboard .  You can check if you have that already by running:
{% highlight console %}
$ vim --version
$ mvim --version
VIM - Vi IMproved 7.4 (2013 Aug 10, compiled Aug 27 2015 16:21:48)
MacOS X (unix) version
Included patches: 1-769
Compiled by Homebrew
Huge version with MacVim GUI.  Features included (+) or not (-):
+acl             +file_in_path    +mouse_sgr       +tag_binary
+arabic          +find_in_path    -mouse_sysmouse  +tag_old_static
+autocmd         +float           +mouse_urxvt     -tag_any_white
+balloon_eval    +folding         +mouse_xterm     +tcl
+browse          -footer          +multi_byte      +terminfo
++builtin_terms  +fork()          +multi_lang      +termresponse
+byte_offset     +fullscreen      -mzscheme        +textobjects
+cindent         -gettext         +netbeans_intg   +title
+clientserver    -hangul_input    +odbeditor       +toolbar
+clipboard       +iconv           +path_extra      +transparency
+cmdline_compl   +insert_expand   +perl            +user_commands
+cmdline_hist    +jumplist        +persistent_undo +vertsplit
+cmdline_info    +keymap          +postscript      +virtualedit
+comments        +langmap         +printer         +visual
+conceal         +libcall         +profile         +visualextra
+cryptv          +linebreak       +python          +viminfo
+cscope          +lispindent      -python3         +vreplace
+cursorbind      +listcmds        +quickfix        +wildignore
+cursorshape     +localmap        +reltime         +wildmenu
+dialog_con_gui  -lua             +rightleft       +windows
+diff            +menu            +ruby            +writebackup
+digraphs        +mksession       +scrollbind      -X11
+dnd             +modify_fname    +signs           -xfontset
-ebcdic          +mouse           +smartindent     +xim
+emacs_tags      +mouseshape      -sniff           -xsmp
+eval            +mouse_dec       +startuptime     -xterm_clipboard
+ex_extra        -mouse_gpm       +statusline      -xterm_save
+extra_search    -mouse_jsbterm   -sun_workshop    -xpm
+farsi           +mouse_netterm   +syntax
...
{% endhighlight %}

From here forward, I'll assume you're using MacVim.  I was a little hesitant at first too but you can run MacVim in a terminal by using:
{% highlight console %}
$ mvim -v Filename
{% endhighlight %}

Here's what I have in my ~/.zshrc for helpers:

{% highlight bash %}
export EDITOR=/usr/local/bin/mvim
mac_vim="$EDITOR -v"
alias vi=$mac_vim
alias vim=$mac_vim
alias v=$mac_vim
{% endhighlight %}

Ok, now following garyjohn's instructions, let's open up ~/.ssh/config in vim:

{% highlight console %}
$ v ~/.ssh/config
{% endhighlight %}

Let's copy in:

{% highlight console %}
ForwardX11 yes
SendEnv WINDOWID
{% endhighlight %}

That's it for OSX.

## Remote (ie Ubuntu, Linux, *Nix, Debian, etc)

You'll need appropriate permissions, so either run as root or have sudo access.

First a lot of distro's come with a pre-installed version of VIM, unfortunately like OSX they usually don't have the write packages installed which is 50% of our issue(s). I recommend the following:

{% highlight console %}
$ sudo apt-get purge vim && sudo apt-get install vim-gtk
{% endhighlight %}

If you run vim --version you should see the gtk package ships with most of the extra options for vim, including +clipboard and +xterm_clipboard.

Let's head into /etc/ssh/sshd_config

{% highlight console %}
$ vim /etc/ssh/sshd_config
{% endhighlight %}

In there put:

{% highlight console %}
AcceptEnv WINDOWID
{% endhighlight %}

We're ready to yank.  Let's try a line in some file

{% highlight console %}
"*yy
{% endhighlight %}

Open vim locally and paste, with p.  There you have it, we just copy & pasted from a remote ssh session to your local vim.
