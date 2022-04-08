---
title: "Emacs Company Clang 自动补全"
date: 2021-09-11T00:30:30+08:00
draft: true
---



## 安装

安装clang:

```bash
bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"
```

安装spacemacs:

```bash
git clone -b develop https://github.com/syl20bnr/spacemacs ~/.emacs.d
```

配置 .spacemacs （`SPC f e d`），增加以下 layer：

```
   dotspacemacs-configuration-layers
   '(
     ;; ----------------------------------------------------------------
     ;; Example of useful layers you may want to use right away.
     ;; Uncomment some layer names and press `SPC f e R' (Vim style) or
     ;; `M-m f e R' (Emacs style) to install them.
     ;; ----------------------------------------------------------------
     auto-completion
     better-defaults
     emacs-lisp
     ;; git
     helm
     (c-c++ :variables c-c++-backend 'lsp-clangd)
     lsp
     ;; markdown
     multiple-cursors
     org
     ;; (shell :variables
     ;;        shell-default-height 30
     ;;        shell-default-position 'bottom)
     spell-checking
     ;; syntax-checking
     ;; version-control
     treemacs)
```



## 不支持 C++ 17 怎么办？

在 home 或者 project root 下创建 `compile_flags.txt` 文件，文件内容如下：

```
-std=c++17
-g
```

windows 上 clangd 也支持 C++17 了，只需要把 compile_flags.txt 放在项目根目录即可。



https://gitter.im/emacs-lsp/lsp-mode?at=5df3bf48578ecf4b1f9963ca

https://sarcasm.github.io/notes/dev/compilation-database.html

https://clangd.llvm.org/installation.html

[Configuring Emacs as a C/C++ IDE - LSP Mode - LSP support for Emacs](https://emacs-lsp.github.io/lsp-mode/tutorials/CPP-guide/)



## desktop integration

