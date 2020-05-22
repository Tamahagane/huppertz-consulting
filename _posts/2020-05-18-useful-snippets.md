---
title: "Useful Snippets"
description: "Collection of useful code snippets"
layout: post
toc: true
comments: false
hide: false
search_exclude: true
categories: [tooling, python, javascript]
---
 
# Useful Code Snippets

## TensorFlow

### Logging TensorFlow Data into a directory within cwd
-------------------------------------------

```python
#create directory first

import os
log_dir = os.path.join(os.getcwd(), "logs","log1")
print(log_dir)

writer = tf.summary.FileWriter(log_dir)
writer.add_graph(sess.graph) #sess is the current Tensorflow session
```

## bash

### Delete Files older than x days

```bash
find /path/to/files/ -type f -name '*.jpg' -mtime +30 -exec rm {} \;
```
First part is the path where your files are located. Don’t use wildcard * if you have a lot of files because you will get Argument list too long error.
Second part -type is the file type f stands for files
Third part -name is limiting *,jpg files
Fourth part -mtime gets how many days the files older then will be listed. +30 is for files older then 30 days.
Fifth part -exec executes a command. In this case rm is the command, {} gets the filelist and \; closes the command


### Create Virtual Environment with virtualenv
```bash
virtualenv name_of_venv
source venv\Scripts\activate
```

### Bash search text in files
```bash
grep -rnw '/path/to/somewhere/' -e 'pattern'
-r or -R is recursive,
-n is line number, and
-w stands for match the whole word.
-l (lower-case L) can be added to just give the file name of matching files.
```

## pandas

### Normalise columns in a dataframe
```python
diabetes[cols_to_norm] = diabetes[cols_to_norm].apply(lambda x: (x - x.min())/(x.max()-x.min()))
```

## git

### Shallow cloning of git repo
```bash
git clone --depth=1 <git.repo>
```

## javascript

### Timestamp

```javascript
var ts = Math.round((new Date()).getTime() / 1000);
```
##Misc

### Vim Paste to Command Line
```
CTRL+R dann * oder "
```

### Symbolic Links on folders in windows command prompt
```cmd
mklink /J "C:\Link To Folder" "C:\Users\Name\Original Folder"
```

### Symbolic Link for Files
```cmd
mklink /H linkname target
```

[Source](https://www.howtogeek.com/howto/16226/complete-guide-to-symbolic-links-symlinks-on-windows-or-linux/)

