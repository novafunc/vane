# Preface

Vane was developed to overcome a few annoyances and limitations of GNU Stow. Namely,

- Symbolic links present compatibility issues with sandbox environments like Flatpak.
- GNU Stow requires installation via a package manager.
- Improper removal of managed files leaves dead links in destination directories, which need manual cleanup.
- A proper workflow of stow delete, pulling chances, and restowing may introduce a change that results in the entire restow to fail.

Alternatives like chezmoi and home-manager address some annoyances and introduce others, such as higher complexity.

So I built Vane. It just does what I want, without annoyances, while being simpler to use and iterate on.

# Features

## Real files, not symlinks

Vane does not use links for a couple of reasons.

- Hard links are undesirable because they do not work across fileystem boundaries. They are also unpredictable; some programs may overwrite a hard link, severing the connection to file in the managed directory.
- Symbolic links are problematic when it comes to sandboxing. A flatpak may struggle to follow a relative link that goes outisde of the sandbox and requires access to the managed directory.

To avoid these issues, real files are used instead. Real files are not perfect either, but here is how vane works around some issues.

- Files are only re-copied if the file at its destination has diverged from the managed file.
- If the managed file and destination file diverage, you can choose to overwrite the destination file or adopt its changes into the managed file.

## State management

Vane keeps track of all files it copies to destinations. If you delete some files from a module without first running a ```--delete``` or ```--delete-all```, no problem. You can remove orphaned files with:

```
vane --prune
```

## OS-specific configuration

Similar to GNU Stow, modules are used. Modules are prefixed to indicate whether they should be copied to their destination.

- ```common-``` are copied on every OS
- ```linux-``` only get copied on Linux
- ```macos-``` only get copied on macOS

Other OSes like FreeBSD and OpenBSD have not been tested. However, they should still work. The prefix is determined by Python's ```sys.platform```.

# Example usage

## Initial setup

```
# install vane script to ~/.local/bin
cp ./path/to/vane "$HOME/.local/bin"
chmod +x "$HOME/.local/bin/vane"

# create directory for vane modules
mkdir -p "$HOME/vane/modules"

# move into vane directory
cd "$HOME/vane"

# (optional) create a git repo to manage vane
git init
```

## Creating modules

An example for managing ```~/.bashrc```:

```
# create module directory
mkdir -p "$HOME/vane/modules/common-bash"

# copy files to module
cp "$HOME/.bashrc" "$HOME/vane/modules/common-bash"
```

An example for managing some nvim files in ```~/.config/nvim```:

```
# create module directory
mkdir -p "$HOME/vane/modules/common-nvim"

# create directory structure inside module directory
mkdir -p "$HOME/vane/modules/common-nvim/.config/nvim"

# copy files to module
cp "$HOME/.config/nvim/init.lua" "$HOME/vane/modules/common-nvim/.config/nvim"
cp "$HOME/.config/nvim/coc-settings.json" "$HOME/vane/modules/common-nvim/.config/nvim"
```

## Applying vane

To copy managed files to their destinations, simply run

```
vane
```

## Cleanup

If you want to remove the copied files, you remove a single module's files with:

```
vane --delete <module>
```

or remove all modules with:

```
vane --delete-all
```
