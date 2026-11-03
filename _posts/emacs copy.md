brew services restart emacs

Or, if you don't want/need a background service you can just run:

  /opt/homebrew/opt/emacs/bin/emacs --fg-daemon

==> **Summary**

🍺  /opt/homebrew/Cellar/emacs/28.2: 4,097 files, 110.4MB

==> **Running `brew cleanup emacs`...**

Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.

Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).

==> **Caveats**

==> **emacs**

To restart emacs after an upgrade:

  brew services restart emacs

Or, if you don't want/need a background service you can just run:

  /opt/homebrew/opt/emacs/bin/emacs --fg-daemon
 **EMACS**

  

  

;; ===================================

;; Basic Customization

;; ===================================

  

(setq inhibit-startup-message t)    ; Hide the startup message

(load-theme 'material t)            ; Load material theme

(global-linum-mode t)               ; Enable line numbers globally

(setq make-backup-files nil)

(setq auto-save-default nil) ;backup in one place. flat, no tree structure

(setq backup-directory-alist '(("" . "~/.emacs.d/backup")))

  

  

  

  

  

  

  

  

  

(when (< emacs-major-version 26)

  (error "Your version of emacs is old and must be upgraded before you can use these packages! Version >= 26 is required."))

  

(setq warning-minimum-level :emergency)

(setq frame-resize-pixelwise t

      x-frame-normalize-before-maximize t)

;; set coding system so emacs doesn't choke on melpa file listings

(set-language-environment 'utf-8)

(setq locale-coding-system 'utf-8)

(set-default-coding-systems 'utf-8)

(set-terminal-coding-system 'utf-8)

(unless (eq system-type 'windows-nt)

  (set-selection-coding-system 'utf-8))

(prefer-coding-system 'utf-8)

(setq buffer-file-coding-system 'utf-8)

(setq x-select-request-type '(UTF8_STRING COMPOUND_TEXT TEXT STRING))

  

;; set things that need to be set before packages load

(setq outline-minor-mode-prefix "\C-c\C-o")

(add-hook 'outline-minor-mode-hook

          (lambda () (local-set-key "\C-c\C-o"

                                    outline-mode-prefix-map)))

(setq save-abbrevs 'silently)

(setq max-specpdl-size 10000

      max-lisp-eval-depth 5000)

  

;; load the package manager

(require 'package)

(when (< emacs-major-version 27)

  (package-initialize))

  

(require 'cl-lib)

;; Use y/n instead of yes/no

(fset 'yes-or-no-p 'y-or-n-p)

;; Add additional package sources

(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages/") t)

  

;; Fix gnu package archive verification in Emacs 26.2 by disabling broken TLS 1.3 support

;;  per https://www.reddit.com/r/emacs/comments/cdei4p/failed_to_download_gnu_archive_bad_request/

(if (and (= emacs-major-version 26) (= emacs-minor-version 2))

    (setq gnutls-algorithm-priority "NORMAL:-VERS-TLS1.3"))

  

;; Assume yes to the package installation prompts if EMACS_AUTOINSTALL_PACKAGES=yes

(if (string-equal (getenv "EMACS_AUTOINSTALL_PACKAGES") "yes")

    (defadvice package-install-selected-packages (around auto-confirm compile activate)

        (letf (((symbol-function 'yes-or-no-p) (lambda (&rest args) t))

                  ((symbol-function 'y-or-n-p) (lambda (&rest args) t)))

         ad-do-it)))

  

;; A list of the packages we always want

(setq package-selected-packages

      '(;; gnu packages

        windresize

        diff-hl

        adaptive-wrap

        pdf-tools

        yasnippet

        yasnippet-snippets

        multiple-cursors

        visual-regexp

        command-log-mode

        undo-tree

        better-defaults

        minions

        ace-window

        ctable

        yaml-mode

        deferred

        epc

        s

        rotate

        howdoi

        multi-term

        with-editor

        anzu

        counsel

        flx-ido

        smex

        hydra

        ivy-hydra

        wgrep

        which-key

        outline-magic

        smooth-scroll

        unfill

        company

        company-math

        web-mode

        markdown-mode

        polymode

        poly-markdown

        poly-org

        poly-noweb

        eval-in-repl

        exec-path-from-shell

        dumb-jump

        flycheck

        htmlize

        dictionary

        untitled-new-buffer))

  

;; hide compilation buffer when complete

;; from http://emacs.stackexchange.com/questions/62/hide-compilation-window

(add-hook 'compilation-finish-functions

          (lambda (buf str)

            (run-at-time

             "2 sec" nil 'delete-windows-on

             (get-buffer-create "*compilation*"))

            (run-at-time

             "2 sec" nil 'delete-windows-on

             (get-buffer-create "*Compile-Log*"))))

  

;; install packages if needed

(unless (cl-every 'package-installed-p package-selected-packages)

  (message "Missing packages detected, please wait...")

  (package-refresh-contents)

  (package-install-selected-packages))

  

(package-initialize)

  

;; add custom lisp directory to path

(unless

    (file-exists-p (concat user-emacs-directory "lisp"))

  (make-directory (concat user-emacs-directory "lisp")))

  

;; add custom lisp directory to path

(let ((default-directory (concat user-emacs-directory "lisp/")))

  (setq load-path

        (append

         (let ((load-path (copy-sequence load-path))) ;; Shadow

           (append 

            (copy-sequence (normal-top-level-add-to-load-path '(".")))

            (normal-top-level-add-subdirs-to-load-path)))

         load-path)))

  

;; on OSX Emacs needs help setting up the system paths

(when (memq window-system '(mac ns))

  (require 'exec-path-from-shell)

  ;; From https://github.com/aculich/.emacs.d/blob/master/init.el

  ;; Import additional environment variables beyond just $PATH

  (dolist (var '("PYTHONPATH"         ; Python modules

                 "INFOPATH"           ; Info directories

                 "JAVA_OPTS"          ; Options for java processes

                 "SBT_OPTS"           ; Options for SBT

                 "RUST_SRC_PATH"      ; Rust sources, for racer

                 "CARGO_HOME"         ; Cargo home, for racer

                 "EMAIL"              ; My personal email

                 "GPG_TTY"

                 "GPG_AGENT_INFO"

                 "SSH_AUTH_SOCK"

                 "SSH_AGENT_PID"

                 ))

    (add-to-list 'exec-path-from-shell-variables var))

  (exec-path-from-shell-initialize))

  

;; Add to the list of the packages we want

  

(when (executable-find "R")

  (add-to-list 'package-selected-packages 'ess)

  (add-to-list 'package-selected-packages 'poly-R))

(when (executable-find "python")

  (add-to-list 'package-selected-packages 'poetry)

  (add-to-list 'package-selected-packages 'conda))

(when (executable-find "pdflatex")

  (add-to-list 'package-selected-packages 'auctex)

  (add-to-list 'package-selected-packages 'ivy-bibtex))

(when (executable-find "git")

  (add-to-list 'package-selected-packages 'git-commit)

  (add-to-list 'package-selected-packages 'magit))

(when (executable-find "julia")

  (add-to-list 'package-selected-packages 'julia-mode)

  (add-to-list 'package-selected-packages 'julia-repl))

(when (or (executable-find "ghc")

          (executable-find "stack"))

  (add-to-list 'package-selected-packages 'haskell-mode)

  (add-to-list 'package-selected-packages 'company-ghci))

(when (executable-find "jupyter")

  (add-to-list 'package-selected-packages 'ein))

(when (executable-find "pandoc")

  (add-to-list 'package-selected-packages 'pandoc-mode)

  (add-to-list 'package-selected-packages 'ox-pandoc))

(when (executable-find "scala")

  (add-to-list 'package-selected-packages 'scala-mode)

  (add-to-list 'package-selected-packages 'sbt-mode))

  

;; install packages if needed

(unless (cl-every 'package-installed-p package-selected-packages)

  (message "Missing packages detected, please wait...")

  (package-refresh-contents)

  (package-install-selected-packages))

  

;; ;; clean up the mode line

(setq minions-mode-line-lighter "☰")

(minions-mode 1)

  

;; No, we do not need the splash screen

(setq inhibit-startup-screen t)

  

(require 'better-defaults)

;; better defaults are well, better... but we don't always agree

(with-eval-after-load "menu-bar"

  (menu-bar-mode 1))

(with-eval-after-load "scroll-bar"

  (scroll-bar-mode 1))

(setq select-active-regions 'only)

  

;; scrolling behavior

(setq mouse-wheel-scroll-amount '(1 ((shift) . 1))) ; one line at a time

(setq mouse-wheel-progressive-speed nil) ; don't accelerate scrolling

(setq mouse-wheel-follow-mouse 't) ; scroll window under mouse

(setq scroll-preserve-screen-position t)

(setq scroll-conservatively 100000)

(setq scroll-error-top-bottom t)

(setq scroll-preserve-screen-position t)

;; scroll without moving point

(require 'smooth-scroll)

(global-set-key [(control down)] 'scroll-up-1)

(global-set-key [(control up)] 'scroll-down-1)

(global-set-key [(control left)] 'scroll-right-1)

(global-set-key [(control right)] 'scroll-left-1)

  

;; Use y/n instead of yes/no

(fset 'yes-or-no-p 'y-or-n-p)

  

(transient-mark-mode 1) ; makes the region visible

(line-number-mode 1)    ; makes the line number show up

(column-number-mode 1)  ; makes the column number show up

  

;; make home and end behave

(global-set-key (kbd "<home>") 'move-beginning-of-line)

(global-set-key (kbd "<end>") 'move-end-of-line)

  

;; enable toggling paragraph un-fill

(define-key global-map "\M-Q" 'unfill-paragraph)

  

  ;;; line wrapping

;; neck beards be damned, we don't need to hard wrap. The editor can soft wrap for us.

(remove-hook 'text-mode-hook 'turn-on-auto-fill)

;; (add-hook 'visual-line-mode-hook 'adaptive-wrap-prefix-mode)

;; 

(setq-default truncate-lines t)

(global-visual-line-mode 1)

(add-hook 'prog-mode-hook

          (lambda()

            (setq truncate-lines t)

            (outline-minor-mode t)))

  

;; indicate visual-line-mode wrap

(setq visual-line-fringe-indicators '(left-curly-arrow right-curly-arrow))

(setq visual-line-fringe-indicators '(left-curly-arrow right-curly-arrow))

;; but be gentle

(when (fboundp 'set-fringe-bitmap-face)

               (defface visual-line-wrap-face

                 '((t (:foreground "gray")))

                 "Face for visual line indicators.")

               (set-fringe-bitmap-face 'left-curly-arrow 'visual-line-wrap-face)

               (set-fringe-bitmap-face 'right-curly-arrow 'visual-line-wrap-face))

  

;; don't require two spaces for sentence end.

(setq sentence-end-double-space nil)

  

;; The beeping can be annoying--turn it off

(setq visible-bell t

      ring-bell-function #'ignore)

  

;; save place -- move to the place I was last time I visited this file

(save-place-mode t)

  

;; regular cursor

                                        ;(setq-default cursor-type '(bar . 5))

(setq-default blink-cursor-blinks 0)

(add-hook 'after-init-hook

          (lambda()

            (setq blink-cursor-blinks 0)))

  

;; easy navigation in read-only buffers

(setq view-read-only t)

(with-eval-after-load "view-mode"

  (define-key view-mode-map (kbd "s") 'isearch-forward-regexp))

  

  

;; set up read-only buffers

(add-hook 'read-only-mode-hook 

          (lambda()

            (cond

             ((and (not buffer-read-only)

                   (not (eq (get major-mode 'mode-class) 'special)))

              (hl-line-mode -1)

              (setq-local blink-cursor-blinks 0)

              (setq-local cursor-type '(bar . 3))

              (company-mode t))

             ((and buffer-read-only

                   (not (eq (get major-mode 'mode-class) 'special)))

              (hl-line-mode t)

              (setq-local blink-cursor-blinks 1)

              (setq-local cursor-type 'hollow)

              (company-mode -1)))))

  

;; show parentheses

(show-paren-mode 1)

(setq show-paren-delay 0)

  

;; Use CUA mode to make life easier. We do _not__ use standard copy/paste etc. (see below).

[I(cua-mode t)

  

(cua-selection-mode t) ;; cua goodness without copy/paste etc.

  

;; load windows-style keys using windows key instead of control.

(require 'win-win)

  

;; ;; Make control-z undo

(let ((map (make-sparse-keymap)))

  ;; remap `undo' and `undo-only' to `undo-tree-undo'

  ;; (define-key map [remap undo] 'undo-tree-undo)

  ;; (define-key map [remap undo-only] 'undo-tree-undo)

  ;; bind standard undo bindings (since these match redo counterparts)

  ;; (define-key map (kbd "C-/") 'undo-tree-undo)

  ;; (define-key map "\C-_" 'undo-tree-undo)

  ;; redo doesn't exist normally, so define our own keybindings

  (define-key map (kbd "C-?") 'undo-tree-redo)

  (define-key map (kbd "M-_") 'undo-tree-redo)

  (define-key map (kbd "C-S-z") 'undo-tree-redo)

  ;; just in case something has defined `redo'...

  (define-key map [remap redo] 'undo-tree-redo)

  ;; we use "C-x U" for the undo-tree visualizer

  (define-key map (kbd "\C-x U") 'undo-tree-visualize)

  ;; bind register commands

  (define-key map (kbd "C-x r u") 'undo-tree-save-state-to-register)

  (define-key map (kbd "C-x r U") 'undo-tree-restore-state-from-register)

  ;; set keymap

  (setq undo-tree-map map))

  

(global-undo-tree-mode t)

  

;; Make C-g quit undo tree

(define-key undo-tree-visualizer-mode-map (kbd "C-g") 'undo-tree-visualizer-quit)

(define-key undo-tree-visualizer-mode-map (kbd "<escape> <escape> <escape>") 'undo-tree-visualizer-quit)

  

  

  

;;

;; Make right-click do something close to what people expect

(require 'mouse3)

(global-set-key (kbd "<mouse-3>") 'mouse3-popup-menu)

;; (global-set-key (kbd "C-f") 'isearch-forward)

;; (global-set-key (kbd "C-s") 'save-buffer)

;; (global-set-key (kbd "C-o") 'counsel-find-file)

(define-key cua-global-keymap (kbd "<C-S-SPC>") nil)

(define-key cua-global-keymap (kbd "<C-return>") nil)

(setq cua-rectangle-mark-key (kbd "<C-S-SPC>"))

(define-key cua-global-keymap (kbd "<C-S-SPC>") 'cua-rectangle-mark-mode)

  

;; zoom in/out like we do everywhere else.

(global-set-key (kbd "C-+") 'text-scale-increase)

(global-set-key (kbd "C--") 'text-scale-decrease)

(global-set-key (kbd "<C-mouse-5>") 'text-scale-decrease)

(global-set-key (kbd "<C-mouse-4>") 'text-scale-increase)

;; page up/down

(global-set-key (kbd "<C-prior>") 'beginning-of-buffer)

(when (< emacs-major-version 26)

  (error "Your version of emacs is old and must be upgraded before you can use these packages! Version >= 26 is required."))

  

(setq warning-minimum-level :emergency)

  

(setq frame-resize-pixelwise t

      x-frame-normalize-before-maximize t)

  

;; set coding system so emacs doesn't choke on melpa file listings

(set-language-environment 'utf-8)

(setq locale-coding-system 'utf-8)

(set-default-coding-systems 'utf-8)

(set-terminal-coding-system 'utf-8)

(unless (eq system-type 'windows-nt)

  (set-selection-coding-system 'utf-8))

(prefer-coding-system 'utf-8)

(setq buffer-file-coding-system 'utf-8)

(setq x-select-request-type '(UTF8_STRING COMPOUND_TEXT TEXT STRING))

  

;; set things that need to be set before packages load

(setq outline-minor-mode-prefix "\C-c\C-o")

(add-hook 'outline-minor-mode-hook

          (lambda () (local-set-key "\C-c\C-o"

                                    outline-mode-prefix-map)))

(setq save-abbrevs 'silently)

(setq max-specpdl-size 10000

      max-lisp-eval-depth 5000)

  

;; load the package manager

(require 'package)

(when (< emacs-major-version 27)

  (package-initialize))

  

(require 'cl-lib)

;; Use y/n instead of yes/no

(fset 'yes-or-no-p 'y-or-n-p)

;; Add additional package sources

(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages/") t)

  

;; Fix gnu package archive verification in Emacs 26.2 by disabling broken TLS 1.3 support

;;  per https://www.reddit.com/r/emacs/comments/cdei4p/failed_to_download_gnu_archive_bad_request/

(if (and (= emacs-major-version 26) (= emacs-minor-version 2))

    (setq gnutls-algorithm-priority "NORMAL:-VERS-TLS1.3"))

  

;; Assume yes to the package installation prompts if EMACS_AUTOINSTALL_PACKAGES=yes

(if (string-equal (getenv "EMACS_AUTOINSTALL_PACKAGES") "yes")

    (defadvice package-install-selected-packages (around auto-confirm compile activate)

        (letf (((symbol-function 'yes-or-no-p) (lambda (&rest args) t))

                  ((symbol-function 'y-or-n-p) (lambda (&rest args) t)))

         ad-do-it)))

  

;; A list of the packages we always want

(setq package-selected-packages

      '(;; gnu packages

        windresize

        diff-hl

        adaptive-wrap

        pdf-tools

        yasnippet

        yasnippet-snippets

        multiple-cursors

        visual-regexp

        command-log-mode

        undo-tree

        better-defaults

        minions

        ace-window

        ctable

        yaml-mode

        deferred

        epc

        s

        rotate

        howdoi

        multi-term

        with-editor

        anzu

        counsel

        flx-ido

        smex

        hydra

        ivy-hydra

        wgrep

        which-key

        outline-magic

        smooth-scroll

        unfill

        company

        company-math

        web-mode

        markdown-mode

        polymode

        poly-markdown

        poly-org

        poly-noweb

        eval-in-repl

        exec-path-from-shell

        dumb-jump

        flycheck

        htmlize

        dictionary

        untitled-new-buffer))

  

;; hide compilation buffer when complete

;; from http://emacs.stackexchange.com/questions/62/hide-compilation-window

(add-hook 'compilation-finish-functions

          (lambda (buf str)

            (run-at-time

             "2 sec" nil 'delete-windows-on

             (get-buffer-create "*compilation*"))

            (run-at-time

             "2 sec" nil 'delete-windows-on

             (get-buffer-create "*Compile-Log*"))))

  

;; install packages if needed

(unless (cl-every 'package-installed-p package-selected-packages)

  (message "Missing packages detected, please wait...")

  (package-refresh-contents)

  (package-install-selected-packages))

  

(package-initialize)

  

;; add custom lisp directory to path

(unless

    (file-exists-p (concat user-emacs-directory "lisp"))

  (make-directory (concat user-emacs-directory "lisp")))

  

;; add custom lisp directory to path

(let ((default-directory (concat user-emacs-directory "lisp/")))

  (setq load-path

        (append

         (let ((load-path (copy-sequence load-path))) ;; Shadow

           (append 

            (copy-sequence (normal-top-level-add-to-load-path '(".")))

            (normal-top-level-add-subdirs-to-load-path)))

         load-path)))

  

  

  

  

  

  

(defvar myPackages

  '(better-defaults                 ;; Set up some better Emacs defaults

    material-theme                  ;; Theme

    elpy ;;emacs lisp python environment

    flycheck ;;

    )

  )

  

  

;; Scans the list in myPackages

;; If the package listed is not already installed, install it

(mapc #'(lambda (package)

          (unless (package-installed-p package)

            (package-install package)))

      myPackages)

  

;; User-Defined init.el ends here