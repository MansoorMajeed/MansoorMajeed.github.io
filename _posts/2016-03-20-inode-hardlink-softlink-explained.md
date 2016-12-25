---
id: 717
title: Inodes, Hard links and Soft links demystified
date: 2016-03-20T14:41:03+00:00
author: Mansoor
layout: post
guid: https://digitz.org/blog/?p=717
permalink: /inode-hardlink-softlink-explained/
dsq_thread_id:
  - 4677701709
categories:
  - Learn
  - Linux
  - Sysadmin
tags:
  - hardlink
  - inode
  - links
  - Linux
  - symlink
---
If you&#8217;re a beginner in Linux, chances are you&#8217;ve come across this question yourself. What exactly is a hard link? And how the heck does it differ from a soft link ( symbolic link or symlink ). Sometimes even experienced Linux Admins have this question.
  
Most of us know what a symlink is but gets really confused when we hear about &#8220;hard link&#8221;. It&#8217;s not that hard though, let me explain it for you.

Before we get into knowing the difference between soft link and hard link, we have a few basic things to understand. One of them is &#8220;Inode&#8221;

#### What exactly is an Inode and Inode number?

Inode is a data-structure that represents a file or a directory. An Inode number is a unique number given to an inode. In plain English, this is what it means.

  * Everything in a unix system is represented as a file ( including regular files like your movies, documents, audio files )
  * A file is stored in a file system ( ext3, ext4 etc ).
  * But when you store a file in a file system, you need to have some kind of identification mechanism so that you can retrieve the file later, modify it, or even delete it. If there is no unique identification mechanism, you have a filesystem full of random garbage data that you have no way of identifying or using.
  * So, we need a way to identify each file in a file system. We use inode number for that.
  * Inode number of a file points to the inode corresponding to that file. And each inode represents a file.
  * When I say &#8220;represent&#8221;, I mean, each inode contains a lot of information about the file [ size of the file, owner, group, permissions and all kinds of stuff like that ]
  * There is a limit to the maximum number of inodes in a file system. So, it is possible that you can&#8217;t store any more files even if you have plenty of storage left ( That&#8217;s right, you have GBs of storage space left, but you have a huuuuge number of files in your disk that you have used up all the inodes and you cannot store any more data. )

Okay, that&#8217;s enough with the theory. Let&#8217;s see some practical stuff to easily understand the inodes and inode number.

<pre class="toolbar:1 lang:bash decode:true">$ touch file
$ 
$ # We use the following command to see the inode number
$ 
$ ls -i file 
797380 file
$ # Let's try that with a directory
$
$ mkdir dir1
$ 
$ ls -id dir1
797359 dir1
$ # 'd' switch is used to display the description of the directory
$ # rather than the contents of the directory
$ 
$ # NOw let's try and change the contents of the file
$ # and see if the inode number is changing
$ echo "hello there" &gt; file 
$ ls -i file 
797380 file
$ 
$ # Nope, the inode number did not change
$ # how about if we copy the file?
$ cp file file1
$ ls -il
total 28
797359 drwxr-xr-x 2 mansoor mansoor 4096 Mar 20 13:38 dir1
797380 -rw-r--r-- 1 mansoor mansoor   12 Mar 20 13:40 file
797385 -rw-r--r-- 1 mansoor mansoor   12 Mar 20 13:40 file1
$ # of course the inode number for the copied file is different
$ # how about moving the file?
$ mv file file2
$
$ ls -i file2 
797380 file2

$ # See, the inode number did not change, just the file name changed.
</pre>

So, now you know what the inode and inode number stands for. Let&#8217;s get back to &#8220;hard link and soft link&#8221; part

#### What exactly is a Soft link?

A soft link is also known as a symbolic link or a symlink. As the name suggests, it&#8217;s a symbolic link to another file. But come one that&#8217;s not a good definition.

A symbolic link is a file ( yes, that&#8217;s right, it&#8217;s another file ) which contains information about the location of a target file or directory.

Let&#8217;s go through the following bullets to easily understand a Symlink

  * A symlink is a file that points to another file or directory
  * A symlink is independent of the target
  * If you delete a symlink, the target is unaffected
  * If you delete the file, the symlink is still there, but it won&#8217;t work because the target now does not exist ( obviously )

Okay, now let&#8217;s see some actual examples with symlinks

<pre class="toolbar:2 lang:default decode:true">$ # Creating a symlink
$ 
$ ls
dir1  file1  file2
$ 
$ # ln -s source_file destination_link
$
$ ln -s file1 sym_file1
$ 
$ ls -l
total 32
Mar 20 13:38 dir1
Mar 20 13:40 file1
Mar 20 13:40 file2
Mar 20 14:04 sym_file1 -&gt; file1
$
$ # You can see the symlink "sym_file1" is pointing to the file "file1" 
$
$ # Now let us check the inode number of the symlink
$ ls -i sym_file1 
797441 sym_file1
$
$ # Now the inode number of the fike
$ ls -i file1
797385 file1
$ 
$ # As you can see, the inode number for the file and the symlink are different
$ # That means they are two different files, and one of them ( the symlink ) is pointing 
$ # to the other one. Pretty simple.
$
$ # Let's check the contents of the link and the file
$ cat sym_file1 
hello there
$
$ cat file1 
hello there
$
$ # They are both the same ( rightly so )
$
$ # How about we change the content of the symlink and see if it changes the original? 
$ echo 'goodbye' &gt; sym_file1 
$ cat sym_file1 
goodbye
$ 
$ cat file1 
goodbye
$ 
$ # There you go, by changing the contents of the symlink, we actually changed the original file

$
$ # Deleting a symlink
$ # just use rm to delete the link
$ ls
dir1  file1  file2  sym_file1
$ rm -f sym_file1 
$ ls
dir1  file1  file2
$</pre>

And, there, you had your crash course on symlinks and how they work.

#### What the heck is hard link then?

hard link is an interesting concept. If you have read the above explanation about Inode and inode numbers, you would know that each file is identified by a unique number called inode number.
  
Now, a hard link creates a link to the orignal inode number. Well, how does it make a difference?
  
I think it will be easier to understand from the examples.

&nbsp;

<pre class="toolbar:2 lang:default decode:true ">$ # we will create a hard link for the file "file"
$ # but before that, let's check the inode number for "file"
$ 
$ ls -i file 
797385 file
$ 
$ # To create a hard link, use the following syntax
$ ln file hardlink_file
$ 
$ ls
dir1  file  hardlink_file
$ ls -l
total 28
drwxr-xr-x 2 mansoor mansoor 4096 Mar 20 13:38 dir1
-rw-r--r-- 2 mansoor mansoor    8 Mar 20 14:05 file
-rw-r--r-- 2 mansoor mansoor    8 Mar 20 14:05 hardlink_file
$ 
$ # now let's check the inode number of the new link
$ 
$ ls -i hardlink_file 
797385 hardlink_file
$ 
$ # how about that, it's the same inode number as the file
$ 
$ # Let's create another link
$ 
$ ln file hardlink_file2
$ ls -i hardlink_file2
797385 hardlink_file2
$ 
$ # yep, again the same. 
$ # Now, let's see the contents of the files
$
$ cat file 
goodbye
$
$ cat hardlink_file
goodbye
$ 
$ # what if we change the orignal file content?
$ echo "hi there" &gt; file 
$
$ cat file 
hi there
$
$ cat hardlink_file
hi there
$
$ cat hardlink_file2 
hi there
$ 
$ # it changed the contents of the links ( we expected it to )
$ 
$ # what if we change the link instead of the original file?
$ echo "I changed" &gt; hardlink_file
$ 
$ cat file 
I changed
$
$ cat hardlink_file
I changed
$ 
$ # changed the content of all of them
$ 
$ # what if we delete the original file?
$ rm -f file 
$ ls
dir1  hardlink_file  hardlink_file2
$
$ cat hardlink_file
I changed
$ 
$ cat hardlink_file2
I changed
$ 
$ rm -f hardlink_file2
$ 
$ cat hardlink_file 
I changed
$ 
$ # how about that. as you have noticed, the hardlink is still here.
$ # This is the main difference between hardlink and softlink
$ # you can consider each file as a hard link itself. 
$ # even if you delete a hard link, the file is still there ( unless you are deleting
$ # the last hard link )
</pre>

So, what did we learn about hard link? Well, hard link is not a link to the original &#8220;file&#8221;, instead it points to the original inode of the &#8220;file&#8221;, which makes it almost similar to the orignal &#8220;file&#8221; except in the name.

So, by that logic, what if replace the original file with another file? Let&#8217;s try that before wrapping up.

<pre class="toolbar:2 lang:default decode:true ">$ ls
$ 
$ echo "hello" &gt; file
$ ln file hardlink
$ 
$ ls
file  hardlink
$ 
$ cat hardlink 
hello
$ 
$ # Now, let us create another file 
$ 
$ echo "New file" &gt; newfile
$ 
$ # let us just take a note of all the inode numbers of the files
$ 
$ ls -i
797359 file  797359 hardlink  797380 newfile
$ 
$ # as we expected, the hard link and ths file had the same inode number
$ 
$ # now let us replace "file" with "newfile"
$ 
$ cp -f newfile file 
$ 
$ ls -i
797359 file  797359 hardlink  797380 newfile
$ 
$ # did the inode numbers change? Nope. 
$ # let's check the contents to see the difference 
$ 
$ cat file 
New file
$ 
$ cat hardlink 
New file
$ 
$ cat newfile 
New file
$ 
$ # There you go,
$ # So the hard link now contains the contents of the new file. 
$ # why? because it is still pointing to the original inode number
$ # and the file with the inode number "797359" has the content "New file"
$ 
$ # it's pretty simple now, isn't? 
$ 
</pre>

And that concludes this post. Thank you for reading 😀 have a great day.

