+++
title = "Bringing Your Dotfiles Under Version Control - Update"
author = "Saket K"
date = 2021-08-07T16:48:26+02:00
categories = ["tech", "how-to"]
tags = ["bash", "*nix"]
series = []
summary = "An update on bringing your *nix configuration files under version control"
draft = true
+++
Recently, I was hacking away on my Mac when I stumbled across a dated but still relevant post from 2013 on "[How to bring your dotfiles under version control](https://tekguru.wordpress.com/2013/08/19/howto-bring-your-dotfiles-under-version-control/)".

I'm sure this is common practice among most *nix users but I read the post and wondered why I'd never bothered to set things up this way! It sounded ideal to have my most important configuration / personalization files under version control and **online** so that I could easily set up on a new system.

In a nutshell, the post by 'Tek Guru' explains how to move your 'dotfiles' (configuration files like .profile, .zshrc or .vimrc) to a separate directory, create a link from your home directory to them (so your applications know where to find them) and then how to set up a git repository to put them under version control. You can then push the repo to Github so you can clone it to any system you like and install your personalized configuration in an instant!

I'm not going to go into the step-by-step here, but since I made a few updates to the way things worked based on my needs, I felt I should describe them for anyone who might be interested.

# Updated setup.sh script

In the original post, Tek Guru provides a simple script that creates the symlinks from the home directory to the  'dotfiles' directory into which they'd moved the files. For my taste, this script was missing a number of features that I'd like:

1. It required you to specify in the script itself which dotfiles you want to symlink
1. It hard-coded the paths to the files 

Since I was going to make some improvements to the script any way, I re-wrote it and added the following features:

1. You can define what you want to call your 'dotfiles directory'. I defaulted to `~/etc` to stay a little close to *nix conventions
1. You can define a `.dotfileignore` file in which you can create globs that exclude dotfiles you do not want to include in your version control (things like history files that don't really need backing up, in my opinion)
1. It checks to see if you already have a symlink to the file in question and skips over it if it's already there.
1. It gives you a listing of how many new files were processed and how many files were already present.
1. **TODO**: You can specify a `push` parameter indicating whether or not it should commit the changes into git and push them to Github.

>```DISCLAIMER: Please make sure you back up your configuration files before running this script! I have tested it under very limited conditions on my system and cannot accept any liability if you run the script and it messes up your configurations. ```

This is the `setup_dotfiles.sh` as I have it at the moment:

```bash {linenos=table}
#!/usr/bin/env bash -x
# This file:
#
#  - checks for dotfiles in the home directory that have not been added to the dotfiles directory
#    and then moves those files to the dotfiles and places a symlink in the home directory pointing to the file
#  - checks for the presence of a .dotfileignore file in the dotfile directory in which you can specify
#    which dotfiles to exclude in this action
#
# Usage:
#    - ./setup_dotfiles.sh
#
# Author: Saket K 
#
# setup_dotfiles.sh

dotfilesdir="${HOME}/etc" #change the etc directory name to whatever name you prefer
ignorefile="${dotfilesdir}/.dotfileignore"
skipfile=0
countnewfiles=0

# Check the etc directory exists
if [[ ! -d $dotfilesdir ]]; then
    # if not: create etc directory
    if mkdir "$dotfilesdir"; then
        echo "Created ${dotfilesdir}."
    else
        echo "Could not create ${dotfilesdir}."
        exit 4
    fi
fi

for file in ${HOME}/.*     # loop through dotfiles in your home directory
do
  if [ ! -d "$file" ]; then # check if the item is an actual file and not a directory
    while read -r; do   # read in the ignore file and match the current file against the entries
        if [[ "$file" == ${HOME}/$REPLY ]]; then
           skipfile=1  # if there's a match indicate we need to skip this file
           break
        fi
    done < "${ignorefile}";
    if [ $skipfile -eq 0  ]; then
        filename="${file##*/}"   # this parameter expansion removes the path of the file
        if [ -L "$file" ]; then  # check if a symbolic link already exists
            echo "Symbolic link for $file already exists"
        else
            mv $file ${dotfilesdir}
            ln -s ${dotfilesdir}/$filename ${file}
            countnewfiles=$((countnewfiles+1))
        fi
    else
        skipfile=0 #reset skipfile
    fi
  fi
done
echo "$countnewfiles files added."
```

The .dotfileignore file looks like this:


```bash {linenos=table}
.DS_Store
.zcomp*
.zsh_history
```
