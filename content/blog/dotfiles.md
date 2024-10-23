+++
title = 'Managing dotfiles using stow on Linux'
date = 2024-09-17T13:52:22+02:00
draft = false
image = "img/tuxlogo.png"
+++
One thing that’s always a bit of a pain when you use multiple computers, or when you want to reinstall your computer at some point in time is that you have to recreate your configuration files or ‘dotfiles’ to have everything back as you want again.

There are several ways of doing this, but recently I came across a little tool called GNU stow. It is a much easier way of managing dotfiles than for instance using a git bare repository.

How stow works, is you create “stow directory”, in my case dotfiles. Inside that directory you can create further directories for “packages”. For this use, a “package” is basically a collection of files that belong together. Example: a bash directory would store all your bash configuration files like .bashrc, .bash_fucntions, .profile, etc..

Once you have your “package” directory, you can move your dotfiles for that “package” into that directory. (move, so they are no longer in their normal location). If you now call stow with the package name, for instance stow bash it will create symbolic links from all files to their original location.

If you have files under ~/.config then you will need to create a .config directory under your package directory, and then move the files or directories that contain your files under there.

For instance, I have a “package” fish, which has the following structure:

    ~/.dotfiles/fish/.config/fish
And under ~/.config you will find a symlink called fish to that directory.

So the structure is basically this:

- Stow directory

  - Package directory
    - files relative from your home directory
    - directories relative from your home directory
      - files relative from previous directory
      - directories relative from previous directory

Running stow on a package will link to files and / or directories starting from your home directory down. If a subdirectory is put in there, like the bottom one above, it will use the highest common directory entry to link to.

Once you have your structure set up under your stow directory, you can simply go into your stow directory and issue the command stow * or stow packagename and all links will be created.

If you then add your stow directory to a git repository, all you would have to do on a new system or when setting up after install is to clone that repository, make sure you have stow installed and issue the command above to have your setup restored.

Finally, as a little bonus, I created a little fish shell function because I am lazy and don’t want to cd back and forth all the time to make it easier for me to create the links for a “package”. I use fish shell, but the example below should be easy enough to refactor to bash or zsh functions if you use those.

    function dotfiles --wraps="stow"
        if [ -z $argv ]
            echo "No argument given; exiting"
        else
            set _oldpath $PWD
            cd ~/.dotfiles
            stow $argv
            cd $_oldpath
        end
    end
In conclusion, it take a little bit to set up but once that is done, it is very easy to maintain and setting up your profile configuration becomes so easy.

Example layout of my dotfiles directory:

    alacritty:
    Permissions Size User            Date Modified Git Name
    .rw-r--r--   355 throttlemeister  2 Aug 23:26   -- .alacritty.toml
    
    bash:
    Permissions Size User            Date Modified Git Name
    .rw-r--r--  1.0k throttlemeister 11 Jun  2023   -- .bash_aliases
    .rw-r--r--  1.1k throttlemeister 11 Jun  2023   -- .bash_functions
    .rw-r--r--   220 throttlemeister 11 Jun  2023   -- .bash_logout
    .rw-r--r--    92 throttlemeister 11 Jun  2023   -- .bash_profile
    .rw-r--r--  4.3k throttlemeister 11 Jun  2023   -- .bashrc
    .rwxr-xr-x    30 throttlemeister 11 Jun  2023   -- .inputrc
    .rw-r--r--   723 throttlemeister 11 Jun  2023   -- .profile
    
    conky:
    Permissions Size User            Date Modified Git Name
    .rw-r--r--   11k throttlemeister 13 Aug 23:32   -- .conkyrc
    
    fish:
    Permissions Size User            Date Modified Git Name
    drwxr-xr-x     - throttlemeister 17 Sep 14:25   -- .config
    
    git:
    Permissions Size User            Date Modified Git Name
    .rw-r--r--   166 throttlemeister 17 Sep 16:48   -- .gitconfig
    .rw-r--r--     4 throttlemeister 11 Jun  2023   -- .gitignore
    
    zsh:
    Permissions Size User            Date Modified Git Name
    .rw-r--r--  4.6k throttlemeister  2 Aug 23:15   -- .zshrc
    .rw-r--r--   374 throttlemeister  2 Aug 22:45   -- .zshrc.pre-oh-my-zsh
