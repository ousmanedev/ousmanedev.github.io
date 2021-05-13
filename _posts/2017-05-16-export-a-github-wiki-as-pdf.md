---
layout: post
title: "Export a Github wiki as PDF"
date: "2017-05-16"
---

At [Botamp](https://www.botamp.com), we use the [Koala library](https://github.com/arsduo/koala) to interract with the Facebook Graph API. Thus, I happen to read their [wiki](https://github.com/arsduo/koala/wiki) a lot so that I decided to turn it into a well structured PDF ebook that I can quickly and easily read. If you don't have time for the reading, check this [project](https://github.com/ousmanedev/wikitopdf) out.

## What is a Github wiki ?

According to the [Github documentation](https://help.github.com/articles/about-github-wikis/),

> GitHub Wikis are a place in your repository where you can share long-form content about your project, such as how to use it, how it's been designed, manifestos on its core principles, and so on. Whereas a README is intended to quickly orient readers as to what your project can do, wikis can be used to provide additional documentation.

A Github wiki, is itself a Git repository containing a bunch of files written in [markdown](https://guides.github.com/features/mastering-markdown/#intro)  and hosted at: `github.com/owner/repo.wiki.git`

## Requirements

To be able to export a Github wikis as PDF, you will need to have the following tools installed on your computer.

- [Git](http://git-scm.com): You will need it to clone the wiki repository on your local machine.
- [Pandoc](http://pandoc.org): It's a swiss army knife when it comes to converting documents.
- [LaTeX](https://www.latex-project.org): It's a high-quality typesetting system well known in the scientific world.

## The process

We will write a small bash script to achieve our _Github wiki to pdf_ task. The script tasks will mainly be:

- Clone the wiki repository on your local machine
- Convert all of the markdown files into a single LaTeX (.tex) file
- Generate a PDF file from the LaTeX one.

## Let's get our hands dirty

### Get started

Create a file named `wikitopdf.sh` and add the following code into it:

`#!/usr/bin/env bash`

What you just added into the script is called the `shebang`. It's there to tell the computer that the script is to be run with bash.

### Argument checking

You just created the script file. But how will it be used from the command line ? A simple way will be :
```shell
$ wikitopdf.sh owner/repo
```

`owner/repo` here, follows the github repositories naming format. So to export the Koala project's wiki, all you will have to do is:

wikitopdf.sh arsduo/koala

Now, what if you forget to provide the repo name as argument to the script ? Won't it be kind from the script to remind you that ? If yes, add this to the script:
```bash
if [ "$1" == "" ]; then
 echo "You must provide a Github repo name in the format owner/repo"
 exit 1
fi
```
The `$1` argument is the first argument provided to the script. If it's empty,  a message is shown and the script exits.

### Clone the wiki repository

As I said before, a github wiki is itself a Git repository. Since you need to work with that repository files, you have to clone it.
```bash
WIKI_FOLDER="wiki"

git clone "https://github.com/$1.wiki.git" $WIKI_FOLDER
```

Here we use the `git clone` command to clone the remote repository into a folder named `wiki`. Notice that we use the `$1` variable here again.

### Convert all of the markdown files into a single LaTex file

Now that you have all of the wiki files on your computer, you need to convert all of them into a single LaTex file that will later be used to generate the PDF file.
```bash
TEX_FILE="wiki.tex"

for file in "$WIKI_FOLDER"/*.md
do
 echo "chapter{${file:${#WIKI_FOLDER}+1:${#file}-${#WIKI_FOLDER}-4}}" >> $TEX_FILE

 pandoc "$file" -f "markdown_github" -t "latex" >> $TEX_FILE
done
```

What we do here, is lopping over all of the markdown (.md) files located into the wiki repository folder. For each of the file we:

1. Add a `chapter{name}` command into our LaTex file. The `chapter{name}` command in LaTex, creates a new chapter. The `name` here, is the current file name without the folder name and the `.md` extension.
2. Convert it from Markdown to LaTeX and append the results to our LaTeX file.

### Generate the PDF file
```
rm -rf $WIKI_FOLDER # Delete the wiki folder, we don't need it anymore

PDF_FILE="wiki.pdf"
```

# Generate a PDF file from our LaTex file
```bash
pandoc $TEX_FILE -s -o $PDF_FILE --variable documentclass="report" --toc
```

# Remove the LaTex file, we don't need it anymore
```bash
rm $TEX_FILE

echo "PDF file generated at $PDF_FILE"
```

# Exit with success
```
exit 0
```

## Going further

With some LaTeX and Pandoc knowledge, you can easily customize this solution.

Thank you for reading.
