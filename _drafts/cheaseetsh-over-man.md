---
layout: post
title: Cheat.sh over Man
---
As a developer, we use the command line a lot. Sometimes to perform a task, sometimes just to learn about a command.

Let's say you don't remember how to use that `grep` command, so you fire up my terminal and type `man grep`, only to be welcomed with a non friendly big chunk of text.

`Man` is a really powerful tool, but it just gives you too much informations, making it hard to find what you need.
That's when `cheat.sh`(https://cheat.sh) comes in handy. It gives you a brief explanation on how to use a command, with helpful examples.

Try this in your terminal
```shell
man grep
```

And then try this
```shell
curl cheat.sh/grep
```

See the difference ? Cheatseet = Brievity + Examples

That's awesome but hey, it's far easier to type `man grep` than `curl cheat.sh/grep`, you must be saying. I feel you, so let's wrap that long command into a bash function.

Bash functions (and aliases) can be used as commands in the terminal. They're a great way to encapsulate operations in simple commands.

Add the following code into your `~/.bash_profile`
```shell
cheat() {
    curl cheat.sh/$1
}
```
Remember to run `source ~/.bash_profile`, so that your terminal knows about our new function.

From now on, you can just type
```shell
cheat grep
```
That's still one letter more than `man grep`, but you can always rename the function with a shorter name. You just need to make sure that it's a name you can easily remember.

Thanks for reading.