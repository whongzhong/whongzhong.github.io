---
layout: post
title: 一次性更新所有文件的创建时间
date: 2023-12-22 12:27 +0800
categories: notes
tags: [shell]
---

<style>
    pre, code {
        white-space: pre-wrap;
    }
</style>

Macos使用onedrive有一个恶心的bug，当我们使用onedrive将文件同步到本地时，文件的创建时间会被修改成同步时的时间，只有修改时间还是原始的正确属性：
<img width="695" alt="Screenshot 2023-12-22 at 13 01 14" src="https://github.com/whongzhong/whongzhong.github.io/assets/40679859/628b8c7f-67b3-4541-8c26-e5ee17a3b2e3">

而在管理文件时，比如转移照片时，时间属性又十分重要，因此需要有一个合适的方法来将所有时间修改为正确的时间。

使用Stat命令可以获得文件的所有时间信息：
```shell
bash: stat -x $filename

  File: "LICENSE"
  Size: 1068         FileType: Regular File
  Mode: (0644/-rw-r--r--)         Uid: (  501/ username)  Gid: (   20/   staff)
Device: 1,17   Inode: 34875345    Links: 1
Access: Fri Dec 22 08:51:42 2023
Modify: Thu Dec 21 20:25:45 2023
Change: Thu Dec 21 20:25:48 2023
 Birth: Thu Dec 21 20:25:45 2023

```

通过`stat`命令，我们可以获得指定格式的文件时间属性，其中的数字都是我们生成的对应格式的时间属性：
```shell
bash: stat -t %Y%m%d%H%M.%S $filename

16777233 34875345 -rw-r--r-- 1 username staff 0 1068 "202312220851.42" "202312212025.45" "202312212025.48" "202312212025.45" 4096 8 0x40 LICENSE
```

`touch`命令支持用户通过`-t -d`等属性修改文件的所有时间属性，因此我们可以用过stat获取正确的时间元数据，将其替换掉文件的其他不正确时间元数据，其分别支持的时间格式为：
```shell
-t [[CC]YY]MMDDhhmm[.SS]
-d YYYY-MM-DDThh:mm:SS[.frac][tz]
```

在这里我使用了-t命令来修改文件属性，为了批量修改所有元数据，我通过shell来遍历所有文件并把命令批量生成：
```shell
for file in ./*; do
	mtime=$(stat -t %Y%m%d%H%M.%S $file | cut -d\  -f10) #获取stat命令对应位置的输出
	echo "touch -t $mtime \"$file\"" >> scripts.sh; 
done
```
事实上我想在一行命令里面直接把所有操作完成：
```shell
mtime=$(stat -t %Y%m%d%H%M.%S $filename | cut -d\  -f10); touch -t "$mtime" "$filename"
```
但是不知道为什么这么做总是导致touch命令报错，认为时间的格式不正确：
```shell
touch: out of range or illegal time specification: [[CC]YY]MMDDhhmm[.SS]
```
找了半天没有结果，只能用上面的方法取而代之了。