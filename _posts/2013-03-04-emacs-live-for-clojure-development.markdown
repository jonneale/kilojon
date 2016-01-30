---
layout: post
title: Configuring emacs (with emacs live) for Clojure development
---
#Configuring emacs (with emacs live) for Clojure development

##Why

I use Clojure all day every day in my job at [uSwitch.com](http://www.uswitch.com/). Personally, I think emacs is the only editor you should be using for Clojure development,
particularly after my meeting with RMS in 2012, when we spent an evening discussing Clojure over dinner. Over the last couple of years, I've experimented with a number of
different emacs setups. I had a lot of success with the emacs starter kit, thanks to Gaz Jones' 
[excellent tutorial](http://blog.gaz-jones.com/2012/02/01/setting_up_emacs_for_clojure_development.html) for setting it up. 

Recently, however, I've been pairing more and more, and it just makes sense for me to move away from my accidentally weird and wonderful personal setup, to something other people
can actually use.


To this end, I started looking around for alternatives. I've been aware of [Sam Aaron's](http://sam.aaron.name/) [emacs live](https://github.com/overtone/emacs-live) project since I heard people raving about it at the
2012 Clojure Conj. I was lucky enough to chat with Sam at [Emacs conf](http://emacsconf.herokuapp.com/), and he encouraged me to have a go with it for a while. The following 
is an ongoing chronicle of my personal customisations and discoveries whilst using the base emacs live package.

##Emacs Live
Emacs live has a great and very detailed readme. If you just want a quick start, I would follow it to the letter, and just run the described
script (once you have read and understood what it's doing! Beware rogue internet scripts) which should just do everything for you.

Personally, I forked the project so that I can commit my own additions. [You can see my personalisations here](https://github.com/jonneale/emacs-live)

The first difficulty I ran into out of the box was that the version of emacs I had installed (an early version of emacs 24) did not like some of the emacs live
customisations. I use homebrew, so all I had to do was uninstall the previous version, and install the latest version, remembering to install it with Cocoa support,
like so

`brew install --cocoa emacs`

`ln -s /usr/local/Cellar/emacs/24.3/Emacs.app /Applications/Emacs.app`

##Customisations
Once I had the vanilla emacs live package up and running, I found I had some trouble typing the "#" symbol on my UK keyboard, using the usual ALT-3 combination.
I've run into this issue before, so to fix it, I added the following line to my init.el

`(global-set-key (kbd "s-3") '(lambda () (interactive) (insert "#")))`

Next, I added a couple of incredibly useful custom functions written by my colleague [Ragnar Dahlen](https://github.com/ragnard) which limit the amount of text printed out to the
nrepl buffer.

`(require 'nrepl)`

`(defun nrepl-limit-print-length ()
  (interactive)
  (nrepl-send-string-sync "(set! *print-length* 100)" "clojure.core"))`

`(defun nrepl-unlimit-print-length ()
  (interactive)
  (nrepl-send-string-sync "(set! *print-length* nil)" "clojure.core"))`

I find these so useful because I often seem to evaluate a massive data structure by mistake, and without these functions, I either have to kill my nrepl process
or wait several minutes for the data to finish printing.

Next, I have another customisation Ragnar showed me. It very simply formats json documents for easy reading. As I spend most of my day looking at json APIs, this
can come in very handy

`(defun beautify-json ()
  (interactive)
  (let ((b (if mark-active (min (point) (mark)) (point-min)))
        (e (if mark-active (max (point) (mark)) (point-max))))
    (shell-command-on-region b e
                             "python -mjson.tool" (current-buffer) t)
    (esk-indent-buffer)))`

One thing I really missed with emacs live was the ability to navigate between buffers using the shift and arrow keys. C-x O seems very cumbersome to me. Instead,
I use the [WindMove](http://www.emacswiki.org/emacs/WindMove) functionality, achieved by adding this snippet:

`(when (fboundp 'windmove-default-keybindings)
      (windmove-default-keybindings))`
      
Next, I don't particularly like the font that comes as standard with Emacs Live. I also found it very hard to change because of [this issue](https://github.com/overtone/emacs-live/issues/25).
Luckily, following the guidance in the linked issue, I was able to change to something I find a bit easier to read by adding the following lines:

`(remove-if (lambda (x)
             (eq 'font (car x)))
           default-frame-alist)
(cond
 ((and (window-system) (eq system-type 'darwin))
  (add-to-list 'default-frame-alist '(font . "Anonymous Pro 16"))))`
  
Finally (for now!) I really like being about to go true fullscreen at a keypress. This was reasonably straightforward

`(defun toggle-fullscreen ()
  "Toggle full screen"
  (interactive)
  (set-frame-parameter
     nil 'fullscreen
     (when (not (frame-parameter nil 'fullscreen)) 'fullboth)))`
     
`(global-set-key [f7] 'toggle-fullscreen)`

And that's it! I appreciate that everyone has their own emacs setup, but so far this works for me. I'll continue to update this post as I find more essential
customisations. Feel free to leave a comment for anything I've missed


