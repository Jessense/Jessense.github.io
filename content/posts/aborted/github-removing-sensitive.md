---
title: "从 Github 历史提交中删除密码等敏感信息"
draft: true
---
使用 bfg 来删除 Github 历史提交中的敏感信息。

将 bfg jar 包下载到 `projects` 文件夹下，将 `passwords.txt` 放在 projects 文件夹，在 projects 文件夹下执行：

`git clone --mirror example.git`

在 `example.git` 文件夹下执行：

`java -jar ..\bfg-1.14.0.jar ----replace-text ..\passwords.txt`



参考：
- [git - Hide password in all previous commits on Github repo - Stack Overflow](https://stackoverflow.com/questions/48843026/hide-password-in-all-previous-commits-on-github-repo)
- [Removing sensitive data from a repository - GitHub Docs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository)



