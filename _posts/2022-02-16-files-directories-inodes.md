---
layout: post
title: "Files, Directories, Inodes"
date: 2022-02-16 20:00:00 +0800
categories: linux files
---

It's been a while since I've updated this blog. Here's something that I learnt a few months ago about Files, Directories, and Inodes

## What are those?

Directory & File

- A table containing a list of (filename, inode number) pairs
- Each file/directory in a directory is just an entry in the (filename/directory name, inode number) table

Inode

- Identified by a number (inode number)
- Contains the metadata of a file/direcory (i.e. the pointer to the data block, last modified, etc)
  - If it's a directory inode, the data block contains the list of filenames of files within that directory and their respective inode numbers
  - If it's a file inode, the data block will contain the content
- Inode does not store the name of the file/directory
- Structure
  - Twelve direct pointers to point to data blocks directly
  - One singly indirect pointer
  - One doubly indirect pointer
  - One triply indirect pointer
- As the file grow, more data blocks will be used and direct pointers will be used to point to these data blocks
  - After reaching a threshold and using up the twelve pointers, the file system grabs a data block and start using it to store additional block pointers (to point to more data blocks), and have the indirect pointer point to this data block. As the indirection grows, the larger the file can grow

## Reading a file

This is still a hypothesis. Let's say to `cat /path/to/foo.txt`

The root directory `/` inode is 2 (for ext4 at least)

1. Load in root directory `/` inode, get pointer to `/` disk block
1. Load in `/` disk block, get inode number of `path`
1. Load in `path` inode, get pointer to `/path` disk block
1. Load in `/path` disk block, get inode number of `to`
1. Load in `to` inode, get pointer to `/path/to` disk block
1. Load in `/path/to` disk block, get inode number of `foo.txt`
1. Load in `foo.txt` inode, get pointer to `/path/to/foo.txt` disk block
1. Load in `/path/to/foo.txt` disk block (i.e. the content)

Each load is a disk I/O, unless cached

## Facebook's Haystack

[Facebook's Haystack](https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Beaver.pdf) tries to minimise the disk I/O per photo read by combining multiple photos into a large file, resulting in lesser inode overheads.

## Soft link vs Hard link

A hard link to a file (say my-file.txt), creates a new directory entry (filename, inode number) where the inode number is the same inode number as my-file.txt. Hence, if my-file.txt is deleted (which removes its own directory entry, but not the inode), the hard link is not broken.

A soft link (also known as symbolic link) to a file (say my-file.txt), creates a new directory entry (filename, inode number) and has its own inode number. However, the content of this soft link points to the specific directory entry of my-file.txt. Hence, if my-file.txt is deleted (which removes its own directory entry), the soft link will be broken.

![visualization](/assets/images/soft_link_vs_hard_link.jpg)

> Image from: https://stackoverflow.com/a/29786294

### References

<https://teaching.idallen.com/cst8207/13w/notes/450_file_system.html>
<https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Beaver.pdf>
<https://ext4.wiki.kernel.org/index.php/Ext4_Disk_Layout#Special_inodes>
<https://stackoverflow.com/questions/185899/what-is-the-difference-between-a-symbolic-link-and-a-hard-link>
