---
layout: post
title: Writing Prose with Vim
description: ""
category: 
tags: [vim, pandoc]
---
{% include JB/setup %}

I thought I'd share how I use Vim to write prose, in case it might be
useful to someone else. Also it would be great to hear about how other
people do the same thing. There are many different ways this could be
done, so I will take some time to motivate my choices. But I will not
discuss why I am using Vim for this in the first place. This is written
with other Vim-users in mind, people who are already using Vim for writing
code but still use a 'modern' word processor when writing a book or
a paper.

### Plug-ins

I only use one plug-in when writing prose, the excellent
[VimRoom](http://projects.mikewest.org/vimroom/). I run Vim from
gnome-terminal because it has a full-screen mode. When I launch VimRoom,
the editing window is perfectly centered on the screen and because the
terminal window is full-screen, my environment is free from distractions.

### Hard-wrapped or soft-wrapped?

When a text is 'soft-wrapped' the lines do not have a line-break character
at the end of them, instead the application automatically wraps the text
so it fits on the screen. When a text is 'hard-wrapped' every line on the
screen ends with a line-break character.

Using soft-wrap may seem more intuitive if you are used to writing in word
processors. But I recommend using hard-wrap because this is what Vim was
originally designed to do and is still what it does best. Vim is the
descendant of ed, which is a *line-editor*. If the screen lines do not
match the actual lines, some Vim commands become unintuitive. This could
at least in part be overcome with some mappings, such as mapping 'j' to
'gj', but then you are working against the philosophy of Vim. The biggest
problem with soft-wrap in Vim is scrolling, you can't scroll by screen
lines which makes scrolling feel very jumpy.

Hard-wrapped text too has its disadvantages; you don't want to have to
press return at the end of each line like you are using a type writer. If
you edit a paragraph, the line-breaks get moved around and some lines
continue off the screen while others become really short. Luckily Vim can
take care of both these issues for you in a seamless way.

To avoid having to add the line-breaks yourself when you reach the end of
a line you can use:

    set textwidth=74
    set formatoptions=t1

When editing a paragraph, Vim can reformat it in real time, so that it
feels like you are actually working in soft-wrap mode. There are two
things to be considered here though: first, Vim needs to know what
a 'paragraph' is; second, this can lead to reformatting of text even when
you don't want it.

To address the second problem, I have found that I like to have automatic
reformatting when I am in insert mode, but I don't want it in normal mode.
I use auto-commands to achieve this:

    augroup PROSE
        autocmd InsertEnter * set formatoptions+=a
        autocmd InsertLeave * set formatoptions-=a
    augroup END

If I still make a normal mode edit that requires the text to be
reformatted I use 'Q' to reflow the current paragraph:

    noremap Q gqap

To get back to what Vim considers to be a paragraph; with the options
I have used above, a paragraph is any block of text separated by blank
lines. This makes sense to me because it is also what constitutes
a paragraph in markdown syntax. There is an alternative option which lets
you write paragraphs without having blank lines between them, but that
requires leaving a space at the end of each line in the paragraph, and
having dead whitespace at the end of lines makes me cringe.

### But this makes my text look awful when I paste it into my word processor!

Yes, having a text full of line-breaks and empty lines is not at all
desirable when you move on to the distributing phase. First of all I would
like to explain how I think about this; when I write in Vim, I am writing
source code, even if it is prose, even if I am not using any special
syntax. I am writing source code and it should be formatted so that it can
be read with the simplest of tools, like the unix `cat` program. When
I want it to look pretty, I need to use a tool that can take my source
code and generate something else.

If you want to paste your text into a word processor, you could use this
custom command that takes your hard-wrapped text and soft-wraps it
instead:

    command! -range=% SoftWrap
                \ <line2>put _ |
                \ <line1>,<line2>g/.\+/ .;-/^$/ join |normal $x

This creates a `:SoftWrap` command that reformats your document.

But I recommend something more powerful. I recommend the excellent
[Pandoc](http://johnmacfarlane.net/pandoc/) 'universal document
converter'. It can among other things convert markdown to pdf (via LaTeX),
which is how I usually use it. Its markdown syntax has loads of extensions
like tables, document title block, footnotes, automatic bibliography and
loads of other stuff. Markdown is great because its syntax is readable in
itself, even if you don't know it.

### Tips when using Pandoc

If you are outside of the US and want to convert a markdown text to pdf
using pandoc and LaTeX, you may want to change the paper size to A4 and
make sure the hyphenation is done right for your language. What you need
to do is to use a custom template. You will find the default template on
your machine in a path similar to
`/usr/share/pandoc/templates/latex.template`. Copy that file and do the
necessary changes. This is what I change when writing in Swedish:

    1c1,3
    < \documentclass$if(fontsize)$[$fontsize$]$endif${article}
    ---
    > \documentclass[a4paper,12pt]{article}
    > \usepackage[swedish]{babel}
    > \usepackage{ae}

(If you don't read diff, this means that the first line is removed and
replaced with the bottom three, without the leading '> '.)

Use the pandoc `--template` option when building your pdf.

### Spelling in Vim

Vim has a decent spelling feature which is enabled with the `spell` and
`spelllang` options. I don't at all like the way Vim presents the
suggested corrections when you press `z=`. Instead I use this mapping
which gives you a nice drop-down menu instead:

    nnoremap \s ea<C-X><C-S>

It has the added advantage of being more similar to `[s` and `]s` which
are the keys that you press to jump between the badly spelled words.

### Switching between writing code and prose

I keep this in my `.vimrc` so that I can easily switch between the
settings I use when writing prose and the settings I use when writing
code.

    command! Prose inoremap <buffer> . .<C-G>u|
                \ inoremap <buffer> ! !<C-G>u|
                \ inoremap <buffer> ? ?<C-G>u|
                \ setlocal spell spelllang=sv,en
                \     nolist nowrap tw=74 fo=t1 nonu|
                \ augroup PROSE|
                \   autocmd InsertEnter <buffer> set fo+=a|
                \   autocmd InsertLeave <buffer> set fo-=a|
                \ augroup END

    command! Code silent! iunmap <buffer> .|
                \ silent! iunmap <buffer> !|
                \ silent! iunmap <buffer> ?|
                \ setlocal nospell list nowrap
                \     tw=74 fo=cqr1 showbreak=… nu|
                \ silent! autocmd! PROSE * <buffer>

Now I can switch using `:Prose` and `:Code` commands. I also use `:Code`
sometimes to temporarily disable the automatic formatting in insert mode.
The mappings at the beginning put undo-breaks after each finished
sentence.

That's it! Thank you for reading. If you already use Vim for prose but in
a entirely different way I would love to hear about it.
