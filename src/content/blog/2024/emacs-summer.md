---
title: 'Emacsで始めるClojureのススメ - Clojure環境構築から始めるEmacsのススメ'
pubDate: 2024-08-04
---


# 自己紹介 - Conao3

![img](/blob/2024/6c1e2eb9-aeae-4f13-9d3b-9be0e0c39104.jpg)

-   業務
    -   Pythonista → Clojurian
    -   転職して広島から目黒に引っ越しました。遊んでくれる人募集してます！

-   趣味
    -   EmacsLisp
        -   Author: leaf, seml-mode, org-generate, keg,,,
        -   Co-maintainer: ddskk, cask,,,

-   興味
    -   Lisp, Type, Compiler, GraphQL


# 2020年代の Emacs 入門

![img](/blob/2024/781ef476-d00d-4cad-82b8-f327f8242f15.png)

-   基本構成
    -   `ivy`, `flycheck`, `company`

公開時、 `vertico` が新進気鋭で盛り上がっていたが、あえて保守的なパッケージ選定。 そのおかげか Emacs 入門記事としてはヒットし、 Google 検索でも公式の直後に表示されるようになった。 (Emacs-JP 名義だったことも多分に影響している)

![img](/blob/2024/d0b36cb2-afec-4e42-a1b7-0a3b830a7365.png)


# さて、、 Emacs 使っているよ! という人


# 風になりたい奴だけが Emacs を使えばいい

![img](/blob/2024/11f37e05-5bfa-43fe-9edf-83293ad76e01.png)

私選おすすめ tomoya さん記事。

確かに Emacs の初期学習コストは高い。 しかし、そのコストを乗り越えて余るほどの力を与えてくれる。

風になるかどうか決めるのはあなた次第


# Emacs で始める Clojure 入門

![img](/blob/2024/b8b1d680-f47d-42c6-b5ff-dae53e7a32fa.png)

Emacs と Clojure の相性はとても良い。 正確には Clojure hacker たちの時間が大量に Emacs とそのエコシステムに投下されていることにより、この環境がある。

Ref: [M-x Reloaded: The Second Golden Age of Emacs - bbatsov](https://batsov.com/articles/2024/02/27/m-x-reloaded-the-second-golden-age-of-emacs/)

> Yesterday I wrote that I think Emacs is currently experiencing its (second) Golden Age. &#x2026; I’m reasonably sure that Clojure played a major role in the success of Emacs in recent years by attracting both new users and new contributors to the project.

> 昨日、Emacsは現在（2度目の）黄金期を迎えていると思うと書いた。 &#x2026; 私は、Clojureが新しいユーザーと新しい貢献者の両方をプロジェクトに引き寄せることによって、近年のEmacsの成功に大きな役割を果たしたと確信している。

実際のところ、 Emacs Lisp を覚えた次に Lisp を使ってなにかスタンドアロンのアプリを作りたいとき、 Clojure はとても良い選択肢になる。 Clojure は イミュータブルなデータ構造を持つ言語として設計されており、 JVM の資産を使いつつ、REPL を通して対話的に爆速開発することができる。


# Emacs で始める Clojure 入門


## -> Clojure で始める Emacs 入門 (2025年を生きるためのEmacs入門)

Post [2020年代のEmacs入門](https://emacs-jp.github.io/tips/emacs-in-2020)


# Build Emacs

```bash
#!/usr/bin/env bash

set -euxo pipefail -o posix

today=$(echo | pype -Mdatetime -e 'print(date.today().strftime("%Y%m%d"))' | python)

cd emacs
git fetch --all
git checkout master
git merge origin/master
git clean -fdx
cd ..

rm -rf "emacs-${today}"
cp -r emacs "emacs-${today}"

cd "emacs-${today}"
./autogen.sh
./configure --prefix $HOME/.local --with-tree-sitter --with-xwidgets --with-native-compilation=aot --with-imagemagick
make -j6
make install -j6

echo '=== Build Complete ==='
```


# leaf - modern `use-package`

```elisp
(eval-and-compile
  (customize-set-variable
   'package-archives '(("gnu"   . "https://elpa.gnu.org/packages/")
                       ("melpa" . "https://melpa.org/packages/")))
  (package-initialize)
  (use-package leaf :ensure t)

  (leaf leaf-keywords
    :ensure t
    :init
    (leaf blackout :ensure t)
    :config
    (leaf-keywords-init)))
```

Emacs29から `use-package` が builtin されたため、 `leaf` がインストールしやすくなった (えっ)


# builtinパッケージの設定


## cus-edit

```elisp
(leaf cus-edit
  :doc "tools for customizing Emacs and Lisp packages"
  :custom `((custom-file . ,(locate-user-emacs-file "custom.el"))))
```


## cus-start

```elisp
(leaf cus-start
  :doc "define customization properties of builtins"
  :preface
  (defun c/redraw-frame nil
    (interactive)
    (redraw-frame))

  :bind (("M-ESC ESC" . c/redraw-frame))
  :custom '((user-full-name . "Naoya Yamashita")
            (user-mail-address . "conao3@gmail.com")
            (user-login-name . "conao3")
            (create-lockfiles . nil)
            (tab-width . 4)
            (debug-on-error . t)
            (init-file-debug . t)
            (frame-resize-pixelwise . t)
            (enable-recursive-minibuffers . t)
            (history-length . 1000)
            (history-delete-duplicates . t)
            (scroll-preserve-screen-position . t)
            (scroll-conservatively . 100)
            (mouse-wheel-scroll-amount . '(1 ((control) . 5)))
            (ring-bell-function . 'ignore)
            (text-quoting-style . 'straight)
            (truncate-lines . t)
            (use-dialog-box . nil)
            (use-file-dialog . nil)
            (menu-bar-mode . t)
            (tool-bar-mode . nil)
            (scroll-bar-mode . nil)
            (indent-tabs-mode . nil))
  :config
  (defalias 'yes-or-no-p 'y-or-n-p)
  (keyboard-translate ?\C-h ?\C-?))
```


## autorevert

```elisp
(leaf autorevert
  :doc "revert buffers when files on disk change"
  :global-minor-mode global-auto-revert-mode)
```


## delsel

```elisp
(leaf delsel
  :doc "delete selection if you insert"
  :global-minor-mode delete-selection-mode)
```


## paren

```elisp
(leaf paren
  :doc "highlight matching paren"
  :global-minor-mode show-paren-mode)
```


## simple

```elisp
(leaf simple
  :doc "basic editing commands for Emacs"
  :custom ((kill-read-only-ok . t)
           (kill-whole-line . t)
           (eval-expression-print-length . nil)
           (eval-expression-print-level . nil)))
```


## files

```elisp
(leaf files
  :doc "file input and output commands for Emacs"
  :global-minor-mode auto-save-visited-mode
  :custom `((auto-save-file-name-transforms . '((".*" ,(locate-user-emacs-file "backup/") t)))
            (backup-directory-alist . '((".*" . ,(locate-user-emacs-file "backup"))
                                        (,tramp-file-name-regexp . nil)))
            (version-control . t)
            (delete-old-versions . t)
            (auto-save-visited-interval . 1)))
```


## startup

```elisp
(leaf startup
  :doc "process Emacs shell arguments"
  :custom `((auto-save-list-file-prefix . ,(locate-user-emacs-file "backup/.saves-"))))
```


## savehist

```elisp
(leaf savehist
  :doc "Save minibuffer history"
  :custom `((savehist-file . ,(locate-user-emacs-file "savehist")))
  :global-minor-mode t)
```


## flymake

```elisp
(leaf flymake
  :doc "A universal on-the-fly syntax checker"
  :bind ((prog-mode-map
          ("M-n" . flymake-goto-next-error)
          ("M-p" . flymake-goto-prev-error))))
```


## which-key

```elisp
(leaf which-key
  :doc "Display available keybindings in popup"
  :ensure t
  :global-minor-mode t)
```


# exec-path-from-shell - シェルから環境変数を引き継ぐ

```elisp
(leaf exec-path-from-shell
  :doc "Get environment variables such as $PATH from the shell"
  :ensure t
  :defun (exec-path-from-shell-initialize)
  :custom ((exec-path-from-shell-check-startup-files)
           (exec-path-from-shell-variables . '("PATH" "GOPATH" "JAVA_HOME")))
  :config
  (exec-path-from-shell-initialize))
```


# vertico - 新時代fuzzyfinder


## vertico

```elisp
(leaf vertico
  :doc "VERTical Interactive COmpletion"
  :ensure t
  :global-minor-mode t)
```


## marginalia

```elisp
(leaf marginalia
  :doc "Enrich existing commands with completion annotations"
  :ensure t
  :global-minor-mode t)
```


## consult

```elisp
(leaf consult
  :doc "Consulting completing-read"
  :ensure t
  :hook (completion-list-mode-hook . consult-preview-at-point-mode)
  :defun consult-line
  :preface
  (defun c/consult-line (&optional at-point)
    "Consult-line uses things-at-point if set C-u prefix."
    (interactive "P")
    (if at-point
        (consult-line (thing-at-point 'symbol))
      (consult-line)))
  :custom ((xref-show-xrefs-function . #'consult-xref)
           (xref-show-definitions-function . #'consult-xref)
           (consult-line-start-from-top . t))
  :bind (;; C-c bindings (mode-specific-map)
         ([remap switch-to-buffer] . consult-buffer) ; C-x b
         ([remap project-switch-to-buffer] . consult-project-buffer) ; C-x p b

         ;; M-g bindings (goto-map)
         ([remap goto-line] . consult-goto-line)    ; M-g g
         ([remap imenu] . consult-imenu)            ; M-g i
         ("M-g f" . consult-flymake)

         ;; C-M-s bindings
         ("C-s" . c/consult-line)       ; isearch-forward
         ("C-M-s" . nil)                ; isearch-forward-regexp
         ("C-M-s s" . isearch-forward)
         ("C-M-s C-s" . isearch-forward-regexp)
         ("C-M-s r" . consult-ripgrep)

         (minibuffer-local-map
          :package emacs
          ("C-r" . consult-history))))
```


## affe

```elisp
(leaf affe
  :doc "Asynchronous Fuzzy Finder for Emacs"
  :ensure t
  :custom ((affe-highlight-function . 'orderless-highlight-matches)
           (affe-regexp-function . 'orderless-pattern-compiler))
  :bind (("C-M-s r" . affe-grep)
         ("C-M-s f" . affe-find)))
```


## orderless

```elisp
(leaf orderless
  :doc "Completion style for matching regexps in any order"
  :ensure t
  :custom ((completion-styles . '(orderless))
           (completion-category-defaults . nil)
           (completion-category-overrides . '((file (styles partial-completion))))))
```


## embark

```elisp
(leaf embark-consult
  :doc "Consult integration for Embark"
  :ensure t
  :bind ((minibuffer-mode-map
          :package emacs
          ("M-." . embark-dwim)
          ("C-." . embark-act))))
```


## corfu

```elisp
(leaf corfu
  :doc "COmpletion in Region FUnction"
  :ensure t
  :global-minor-mode global-corfu-mode corfu-popupinfo-mode
  :custom ((corfu-auto . t)
           (corfu-auto-delay . 0)
           (corfu-auto-prefix . 1)
           (corfu-popupinfo-delay . nil)) ; manual
  :bind ((corfu-map
          ("C-s" . corfu-insert-separator))))
```


## cape

```elisp
(leaf cape
  :doc "Completion At Point Extensions"
  :ensure t
  :config
  (add-to-list 'completion-at-point-functions #'cape-file))
```


# eglot - LSP

```elisp
(leaf eglot
  :doc "The Emacs Client for LSP servers"
  :hook ((clojure-mode-hook . eglot-ensure))
  :custom ((eldoc-echo-area-use-multiline-p . nil)
           (eglot-connect-timeout . 600)))

(leaf eglot-booster
  :when (executable-find "emacs-lsp-booster")
  :vc ( :url "https://github.com/jdtsmith/eglot-booster")
  :global-minor-mode t)
```


# puni - 構造的編集

```elisp
(leaf puni
  :doc "Parentheses Universalistic"
  :ensure t
  :global-minor-mode puni-global-mode
  :bind (puni-mode-map
         ;; default mapping
         ;; ("C-M-f" . puni-forward-sexp)
         ;; ("C-M-b" . puni-backward-sexp)
         ;; ("C-M-a" . puni-beginning-of-sexp)
         ;; ("C-M-e" . puni-end-of-sexp)
         ;; ("M-)" . puni-syntactic-forward-punct)
         ;; ("C-M-u" . backward-up-list)
         ;; ("C-M-d" . backward-down-list)
         ("C-)" . puni-slurp-forward)
         ("C-}" . puni-barf-forward)
         ("M-(" . puni-wrap-round)
         ("M-s" . puni-splice)
         ("M-r" . puni-raise)
         ("M-U" . puni-splice-killing-backward)
         ("M-z" . puni-squeeze))
  :config
  (leaf elec-pair
    :doc "Automatic parenthesis pairing"
    :global-minor-mode electric-pair-mode))
```


# cider - Clojure編集環境

```elisp
(leaf cider
  :doc "Clojure Interactive Development Environment that Rocks"
  :ensure t)
```


# vim-jp-radio - ポッドキャストクライアント

```elisp
(leaf vim-jp-radio
  :vc ( :url "https://github.com/vim-jp-radio/vim-jp-radio.el"))
```


# have fun, with Emacs