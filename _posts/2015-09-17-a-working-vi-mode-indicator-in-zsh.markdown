---
layout: post
title:  "A Working VI Mode Indicator in ZSH"
date:   2015-09-17 13:18:53
comments: true
categories: vi
---
Thanks to [@dougblackio][1]'s article [Adding Vi To Your Zsh][2], I have this awesome `vi` mode indicator in my zsh.

![vi mode indicator in zsh][3]

Unfortunately, I found that I had to type `bindkey -v` every time I started a new shell to get this to work. After some research [here][4], I figured out I needed to add `precmd() { RPROMPT="" }` to my `.zshrc`. That resolved the issue.

I also removed [@dougblackio][1]'s custom git status code with this final result:

    bindkey -v
    
    bindkey '^P' up-history
    bindkey '^N' down-history
    bindkey '^?' backward-delete-char
    bindkey '^h' backward-delete-char
    bindkey '^w' backward-kill-word
    bindkey '^r' history-incremental-search-backward
    
    precmd() { RPROMPT="" }
    function zle-line-init zle-keymap-select {
       VIM_PROMPT="%{$fg_bold[yellow]%} [% NORMAL]%  %{$reset_color%}"
       RPS1="${${KEYMAP/vicmd/$VIM_PROMPT}/(main|viins)/} $EPS1"
       zle reset-prompt
    }
    
    zle -N zle-line-init
    zle -N zle-keymap-select
    
    export KEYTIMEOUT=1

Run this command to automatically install this script into `~/.zshrc`

    curl http://www.coryklein.com/fileshare/zsh-vi-mode-indicator.txt >> myfile

 [1]: https://twitter.com/dougblackio
 [2]: http://dougblack.io/words/zsh-vi-mode.html
 [3]: http://coryklein.com/wp-content/uploads/2014/10/vi-mode-zsh.png
 [4]: http://zshwiki.org/home/zle/vi-mode
