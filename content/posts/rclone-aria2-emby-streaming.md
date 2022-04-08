---
title: "Rclone + Google Drive + Aria2 + Emby 搭建个人影音系统"
draft: false
---

## What are we building?

无限容量

## rclone

### 详细教程

1. https://rclone.org/install/

2. https://rclone.org/drive/

   2.1 https://rclone.org/drive/#making-your-own-client-id

3. [在Debian/Ubuntu上使用rclone挂载Google Drive网盘 - Rat's Blog](https://www.moerats.com/archives/481/)

4. [解决Rclone挂载Google Drive时上传失败和内存占用高等问题 - Rat's Blog](https://www.moerats.com/archives/877/)

### 快速步骤

安装：

```bash
curl https://rclone.org/install.sh | sudo bash
```

配置：

```bash
rclone config
```

配置参考2和4，这里建议生成自己的的 Google Drive client_id（详见2.1）。

挂载：

```bash
mkdir ~/GoogleDrive
rclone mount RemoteName:Folder ~/GoogleDrive --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 000
```

其中 DriveName 是上一步配置中设置的remote的Name，Folder是Google Drive的一个子文件夹。

## Aria2

[P3TERX/aria2.sh: Aria2 一键安装管理脚本 增强版](https://github.com/P3TERX/aria2.sh)

```bash
wget -N git.io/aria2.sh && chmod +x aria2.sh
./aria2.sh
```

## 将下载完成的内容上传到 Google Drive

将已完成下载的内容移动到另一个文件夹的脚本：

```shell
DOWNLOAD_DIR="~/Downloading"
AVDC_DIR="~/Downloaded/"
for i in $DOWNLOAD_DIR/*
do
    COMPLETE=1
    for j in $DOWNLOAD_DIR/*
    do
        if [ "$j" == "$i.aria2" ]
        then
            COMPLETE=0
        fi
    done
    if [[ $COMPLETE == 1 ]] && ! [[ "$i" == *aria2 ]] && ! [[ "$i" == *torrent ]]
    then
        mv "$i" "$AVDC_DIR"
    fi
done
```

将Downloaded文件夹增量同步到Google Drive：

```bash
rsync -Pa --remove-source-files ~/Downloaded/ ~/GoogleDrive/Folder/
```

`--remove-source-files`参数的作用是删除Downloaded文件夹中完成上传的文件。



## emby

https://emby.media/linux-server.html