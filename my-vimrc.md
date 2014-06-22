---
layout: post
title:  "my .vimrc"
permalink: /my-vimrc/
---

Here's my [current] `~/.vimrc` file:

{% highlight vim %}
set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

Bundle 'Gist.vim'
Bundle 'altercation/vim-colors-solarized'
Bundle 'mattn/webapi-vim'
Bundle 'tpope/vim-surround'
Bundle 'kien/ctrlp.vim'
Bundle 'godlygeek/tabular'
Bundle 'tpope/vim-markdown'
Bundle 'fxn/vim-monochrome'
Bundle 'VMop/zed.vim'

syntax on
set number

"so I can still use shift+arrows
map C <Right>
map D <Left>
map   <S-Right>
map <S-Left>
map B <Down>
map  <S-Up>
map
     <S-Down>

set background=dark
colorscheme solarized

"use *.md for markdown
au BufRead,BufNewFile *.md set filetype=markdown

"ruby filetypes
au BufRead,BufNewFile Gemfile set filetype=ruby
au BufRead,BufNewFile config.ru set filetype=ruby
au BufRead,BufNewFile Vagrantfile set filetype=ruby
au BufRead,BufNewFile Puppetfile set filetype=ruby

"display trailing whitespace
set listchars=trail:.
set list
{% endhighlight %}

Start simply:

{% highlight bash %}
git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
{% endhighlight %}

Add the two vundle lines to your `~/.vimrc` file:

{% highlight vim %}
set rtp+=~/.vim/bundle/vundle/
call vundle#rc()
{% endhighlight %}

Then the "bundles" are as easy as running `:BundleInstall` from the Vim command prompt.

What are these plugins?

####[Gist.vim](https://github.com/mattn/gist-vim)

Add Gists from Vim, selections, full file, private, etc.  Includes editing an existing Gist ([webapi-vim](https://github.com/mattn/webapi-vim) is a dependency for Gist.vim).

####[vim-surround](https://github.com/tpope/vim-surround)

Replace surrounding delimiters, i.e. turn `"hello, world"`, into `'hello, world'`.  This one is a _little_ complicated, it uses commands, not the command _prompt_, and there are __a lot__ of options.  Check out the readme at the above link.

####[ctrl-p](https://github.com/kien/ctrlp.vim)

This is a fuzzy and MRU (most recently used) search tool for Vim.  Press ctrl+p to start searching, highlight a file, then press ctrl+t to open the file in a new tab, for example.

####[Tabular](https://github.com/godlygeek/tabular)

This one aligns things like Ruby really wants you to:

{% highlight ruby %}
{
  :these => 'things',
  :things => 'are',
  :not => 'aligned'
}
{% endhighlight %}

`:Tab /=>` yields the expected result:

{% highlight ruby %}
{
  :these  => 'things',
  :things => 'are',
  :not    => 'aligned'
}
{% endhighlight %}
