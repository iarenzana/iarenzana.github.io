.---
title:  "Emacs setup for Go Development"
date:   2015-12-01 14:25:00
description: "How to set up the emacs text editor to develop in Go."
categoy: Tech
tags: golang development emacs config programming go
---

Emacs for Go Development
=

Intro
-

There's a lot out there written on how to set up Emacs as a Go development text editor. Up until now, I've been using [Sublime Text](http://www.sublimetext.com/) to do all my Go development. The problem is, [GoSublime](https://github.com/DisposaBoy/GoSublime) has not seen an update in more than a year, and I worry about the future compatibility of the project with future versions of Sublime. I know, there's [vim-go](https://github.com/fatih/vim-go), but I've always been kind of a big fan of emacs and, funny enough, vi commands don't flow as naturally as emacs ones for me. Not to say emacs is easier than vi, it's not, but it's just a matter of preference.

Scenario
-

I'm working on OS X and I'm setting up Emacs 24 to develop in Go.

Install Necessary Components
-
Naturally, we will need Emacs and Go installed in order to start writing software. To do this, let's use [Homebrew](http://brew.sh/) to install both:

```bash
brew update
brew install emacs
brew install go
```

As of this writing, Go 1.5.1 and Emacs 24.5 should be installed.
You should have the following in your `${HOME}/.bashrc`:

```bash
export EDITOR=emacs
export GOPATH=${HOME}/dev/go
export PATH=$PATH:${GOPATH}/bin
```

At this point, we should get a few Go components that will be very useful:

```bash
go get -u golang.org/x/tools/cmd/goimports
go get -u golang.org/x/tools/cmd/vet
go get -u golang.org/x/tools/cmd/oracle
go get -u golang.org/x/tools/cmd/godoc
go get -u github.com/golang/lint/golint
go get -u github.com/nsf/gocode
```

This will install goimports, go vet, oracle (not the DB, mind you), godoc, lint and gocode.

Configure Emacs
--
By now, we have the necessary packages that emacs will need to do formatting, check for unused imports, and some cleanup. The first thing we'll do is add the sources for emacs to pick up the necessary plugins. Add the following to your `${HOME}/.emacs` file:

```lisp
;;Load package-install sources
(when (>= emacs-major-version 24)
  (require 'package)
  (add-to-list
  'package-archives
  '("melpa" . "http://melpa.org/packages/")
   t)
   (package-initialize)
```

go-mode
---
Open emacs `M-x package-install go-mode`. This will install go-mode, that will do syntax detection for Go in Emacs. It will also pull up documentation of the base library using godoc (installed earlier).

![IMAGE 1](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/12/syntax.png)

go-eldoc
---
Install go-eldoc: `M-x package-install go-eldoc`. And load it on your `~/.emacs`:

```lisp
(defun go-mode-setup ()
  (go-eldoc-setup))
```

gofmt
---
Load gofmt in `~/.emacs`. This is necessary to allow for a consistent code formatting.

```lisp
;;Format before saving
(defun go-mode-setup ()
  (go-eldoc-setup)
    (add-hook 'before-save-hook 'gofmt-before-save))
    (add-hook 'go-mode-hook 'go-mode-setup)
```

goimports
---
goimports keeps your imports section nice and tidy, it will add and remove import statements as necessary. Ad these lines to your `~/.emacs`:

```lisp
(defun go-mode-setup ()
  (go-eldoc-setup)
  (setq gofmt-command "goimports")
  (add-hook 'before-save-hook 'gofmt-before-save))
(add-hook 'go-mode-hook 'go-mode-setup)
```

godef
---

![IMAGE 2](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/12/godef.png)

godef allows us to look up function definitions like you would on a regular IDE as shown below. Add this to your `~/.emacs`:

```lisp
;;Godef, shows function definition when calling godef-jump
(defun go-mode-setup ()
  (go-eldoc-setup)
  (setq gofmt-command "goimports")
  (add-hook 'before-save-hook 'gofmt-before-save)
  (local-set-key (kbd "M-.") 'godef-jump))
(add-hook 'go-mode-hook 'go-mode-setup)
```

Custome Compile Command
---
You can configure the command to be executed by default when running `M-x compile`, in this case we want to build, test, and make sure our code is clean. Add this to your .emacs file:

```lisp
;;Custom Compile Command
(defun go-mode-setup ()
  (setq compile-command "go build -v && go test -v && go vet && golint")
  (define-key (current-local-map) "\C-c\C-c" 'compile)
  (go-eldoc-setup)
  (setq gofmt-command "goimports")
  (add-hook 'before-save-hook 'gofmt-before-save)
  (local-set-key (kbd "M-.") 'godef-jump))
  (add-hook 'go-mode-hook 'go-mode-setup)
```

Autocomplete
---

![IMAGE 3](https://raw.githubusercontent.com/iarenzana/iarenzana.github.io/master/assets/images/2015/12/autocomplete.png)

No explanation necessary :) Add the following to you `~/.emacs` file and install with M-x package install go-autocomplete:

```lisp
(ac-config-default)
(require 'auto-complete-config)
(require 'go-autocomplete)
```

Lint
---
You can run golint with `M-x golint` by adding this to your init file:

```lisp
;;Configure golint
(add-to-list 'load-path (concat (getenv "GOPATH")  "/src/github.com/golang/lint/misc/emacs"))
(require 'golint)
```

Extras
---
There's a couple of things that I need in addition to this. First, is a project explorer. To do this, you can run M-x package-install project-explorer. You can toggle the explorer on with M-x e with this key binding.

```lisp
;;Project Explorer
(require 'project-explorer)
(global-set-key (kbd "M-e") 'project-explorer-toggle)
```

For portability, make it so the packages get installed by default on the machine.

```lisp
(defvar my-packages
  '(;;;; Go shit
    go-mode
    go-eldoc
    go-autocomplete

        ;;;;;; Env
    project-explorer)
  "My packages!")

;; fetch the list of packages available
(unless package-archive-contents
  (package-refresh-contents))

;; install the missing packages
(dolist (package my-packages)
  (unless (package-installed-p package)
    (package-install package)))
```

Wrap up
-
At this point we have everything we need to start coding. I particularly like the Solarized dark color scheme. Check it out [here](http://ethanschoonover.com/solarized).

This is the resulting `~/.emacs` file:

<script src="https://gist.github.com/iarenzana/369132f40524b7dc0927.js"></script>

This should be all you need to comfortably start writing Go in Emacs.
