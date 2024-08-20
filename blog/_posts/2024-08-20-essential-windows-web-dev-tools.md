---
layout: post
title: My Essential Tools for Software Development on Windows
description: A rundown of 6 of the tools I use every day to build software.
published: true
---

I'm going to need to set up a new Windows PC for software development soon.
Usually this process involves a week or two of being mindful of what tools I'm using and noting them down; then I can make sure I install them on the new machine and that I have a copy of any settings changes that I usually make. 

Since I've got my list out anyway I thought I'd write a few of my favourite tools down here.
I use some of them every day and others once in a blue moon, but they've all earned a place in my virtual toolbelt. 

Some of these tools are 'trendy', others are a bit niche. I'm sure that there are alternatives that have more complete features / more modern interfaces, but I'm a strong believer in picking a tool and getting to know it well. I've been using a lot of these for nearly a decade, and they work great for me!

## Visual Studio Code

[Visual Studio Code ](https://code.visualstudio.com/) is the ubiquitous text editor from Microsoft that can be used for almost any coding task.
If I'm writing anything for the frontend, it's almost always done in VSCode.
It also works well for backend things (e.g. it has decent C# support), but I usually reach for a more heavyweight IDE for those langauges, as I find VSCode a bit slow and the refactoring support less robust.

### VSCode Extensions

I try not to have too many extensions installed on my VSCode install at a given time, but here are a few that I always install. There are a few others that I imagine are very common, but these are my top 4:

#### [Rainbow CSV](https://marketplace.visualstudio.com/items?itemName=mechatroner.rainbow-csv)
If I need to fiddle with some data in a CSV file for something, then this extension makes reviewing or editing it a breeze.

#### [Catppuccin Macchiato Theme](https://marketplace.visualstudio.com/items?itemName=catppuccin.catppuccin-vsc)
This is my go-to theme at the moment.

#### [Draw.io Integration](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio)

I mostly use [draw.io](https://diagrams.net) for diagramming. This extension is greate because I can save e.g. a `.drawio.png` file that renders the diagram as a `.png` when viewing documentation in a browser, but still contains an embedded editable diagram hidden inside. 

This extension lets me open the diagram in vscode while editing my documentation, and make changes that are instantly rendered. Removing as much friction as possible in editing the diagrams while editing documentatoin means that it's much more likely to stay in sync with the words around it. I love it!

#### [Prettier - Code Formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
I have VSCode set to automatically format on save for most file types, and use Prettier for most frontend code. For backend code, I tend to use a language specific one, like [black](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) for Python.

I like using a code formatters because it means I never have to make a decision about how to format code. I just press save and get consistently formatted code which I find much more scannable. I know not everyone likes this loss of control, but for me the "I don't have to care about this" aspect is a big win.

## Oh My Posh

[Oh My Posh](https://ohmyposh.dev/) is a theme engine for terminal prompts.
On Windows, I set this up with Windows Terminal and Powershell as my default prompt.

I like Oh My Posh for a few reasons:

1. It's _cross platform_: I can use the same config on my Windows dev machine, a mac, and on a linux server I log into a lot.
2. It's _customisable_: I based my setup on one of the built-in themes, but have modified it to work for me. I don't like it being too busy but some situational awareness is nice. For example, I can tell if my git worktree is clean or not by the colour of the branch name in my prompt: no need to run a command.

My theme for Oh My Posh is still something I'm tinkering with (I only changed from the default a few months ago). You can see it's current incarnation on [this gist on Github](https://gist.github.com/rbrunt/233b5b2d9772d3d5bc02d57900b806b0).


## Lazygit

While I can get by with classic git command line commands if I need to, I find a UI much nicer for day-to-day work: it gives you more situational awareness than the output of a single git command, and for common tasks like adding some but not all changes to a commit, I find it much faster.

I used to swear by GitKraken for my git client needs. GitKraken is super powerful, but can be super slow. A couple of years ago I discovered [Lazygit](https://github.com/jesseduffield/lazygit), and I've never looked back.

Lazygit has the benefits of a git GUI, but the productivity benefits of a primarly keyboard driven program, plus it's really fast!


## ScreenToGif

Sometimes the best way to communicate some interaction you've built or a weird bug you've found is through moving images. [ScreenToGif](https://www.screentogif.com/) is a really nice app for recording these.

As the name suggests, it shines at producing Gif files from what's on your screen.
I use it for pasting little demos into a Pull Request to explain the change better, or to show to a stakeholder in Slack or Teams.

If you need to record a longer demo or need a different output format then it can also export into any format that FFmpeg supports, which is basically anything you'll ever need.


## Everything

[Everything](https://www.voidtools.com/) is a little search engine utility that finds files by their filename.
It indexes your entire hard drive in seconds by using the NTFS file table and then lets you search for any file on your PC with the tap of a button.

I use it by binding it to `<Win>+E`, so whenever I need to open a file I know the name of, it's just a few keypresses away.
If I don't know the name of a file but remember roughly what it was called, it's still useful because you can use Regex and wildcards to filter the list down, and then just scroll until you find it.

## Agent Ransack

On the theme of searching, my final tool for today is [Agent Ransack](https://www.mythicsoft.com/agentransack/).
Agent Ransack is a tool for searching through all files in a directory tree for some query. It lets you only search for certain types of files (e.g. you could exclude searching through `.dll` files), but by default it'll (surprisingly quickly) search through any file in the directory you point it to.
It's essentially `find` and `grep` wrapped up in a UI running on windows.

Over the years I've used Agent Ransack to find the number of times a certain error appears in gigabytes of xml log files, check if a customer's name appears in a dataset, or just quickly check how many times a class is used in a Java codebase without having to spin up a full IDE.


While inside a project in VSCode I tend to use the universal search built-in, so I don't get Agent Ransack out as often as I used to, but it's still really helpful in some situations, especially if there are non-plain-text files like office documents to look through.

