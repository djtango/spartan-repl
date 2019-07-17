# Spartan REPL
A Clojure REPL hooked up to vanilla Vim in two lines of code. No [(Spac)Emacs](https://www.braveclojure.com/basic-emacs/), no [Cider](https://github.com/clojure-emacs/cider/), no [fireplace](https://github.com/tpope/vim-fireplace).  
  
---

*spar•tan*  
_(noun)_  
> a Canadian dessert apple of a variety with crisp white flesh and maroon-flushed yellow skin.

## Requirements
VIM (vanilla)
Clojure >= 1.8 ([CLI](https://clojure.org/guides/getting_started) or [lein](https://leiningen.org/#install) both work)

## Socket REPL
Clojure 1.8 introduced the ability for any arbitrary clojure program to have a [socket REPL](https://clojure.org/reference/repl_and_main) that can be externally switched on via a JVM opt flag:

```edn
;; $HOME/.clojure/deps.edn
{:aliases
  {
    ,,,
    :socket {:jvm-opts ["-Dclojure.server.repl={:port,50505,:accept,clojure.core.server/repl}"]}
    ,,,
  }
}
```

Alternatively for Leiningen:
```clj
;; $HOME/.lein/profiles.clj
{
  :user {:aliases {,,,
                   :jvm-opts ["-Dclojure.server.repl={:port,50505,:accept,clojure.core.server/repl}"]
                   ,,,
  }}
}
```

## Configuring VIM
Save this macro in your VIMRC (optionally bound to `p`) then highlight the expression(s) you want to send to the repl and invoke the macro.

```vim
" this is not very paste friendly...
" ^M is return
" ^] is ESC
"$HOME/.vimrc
let @p = '"1ygg0"2y3w:new^M"2pa)^M^["1p:% ! nc -N localhost 50505^M'
```

What this does:  
* yank the selected code to register 1
* move to the top of the current file you are in (NOTE: this assumes your ns declaration is on line 1)
* yank until the end of your namespace name to register 2
* create a fresh unnamed buffer
* paste the namespace declaration and close it
* paste your copied code to be evaluated
* pipe the entire contents of your buffer to your repl (via netcat on port 50505)

## Tying it all together
* Create a fresh REPL by using either:
```sh
lein repl # by default user profile will be merged in via lein
# or
clojure -A:socket # use the global socket alias which turns on the socket repl
```
* in your repl navigate to the sparta namespace:
```clj
(ns spartan-repl.sparta)
```
* this namespace should be empty, test it via:
```clj
madness?
  ;; Syntax error compiling at (REPL:0:0).
  ;; Unable to resolve symbol: madness? in this context
```

* over in sparta.clj evaluate the code within the comment block using the `@p` macro
* your REPL session should now have `madness?` bound
```clj
madness?
;;=> :THIS-IS-SPARTA!
```


## Limitations
* anything preceding the namespace declaration - don't do it :slightly_smiling_face: _what did you expected from a 2 line hack?_
