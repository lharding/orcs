# Overriding Revision Control System

ORCS is a tool for managing "dotfiles" across multiple machines, via the version control tool (or not) of your choice. It's "overriding" because it allows you to create cascading "overrides" where a config-set builds on any number of other config-sets in a directed acyclic graph.

## Dependencies

Python 2 (yes, I wrote this some time ago). 

## Installation

The `orcs` script is self-contained and can be downloaded on its own via curl to help you bootstrap a new machine:
```
curl https://raw.githubusercontent.com/lharding/orcs/master/orcs > orcs
```

## Usage

### General Principle

ORCS manages dotfiles by symlinking them to their 'real' counterparts kept in directories it manages. These directories can live anywhere in your filesystem; the convention I use is to keep it at `.orcs`. Within each directory is a set of config files, and one or more ORCS configuration files. Each configuration file ("`orcsfile`") contains directives for creating symlinks at arbitray filesystem locations pointing to files within its current directory.

Orcsfiles can also `include` other orcsfiles, in a manner similar to the `sh` `source` command. This is how the Overriding part of ORCS' operation is accomplished: subsequent operations in the tree of orcsfiles built by various `include` directives override previous operations, allow a file from a "base" set to replaced by (e.g.) a host-specific version.

Finallly, ORCS operates on the orcsfile found at `$HOME/.orcsfile` when run. This can itself be a symlink to a specified entry-point, allowing a single directory tree to manage configuration for multiple machines or users by simply pointing their .orcsfiles to different real orcsfiles within the directory.

### Commands

#### `orcs`

Compile the tree of orcsfiles beginning at `$HOME/.orcsfile` into a list of symlink names and targets, and update `$HOME` to match, moving any found configuration files into the orcs directory as it goes.

#### `orcs track <FILE>`

Move `FILE` into a directory whose path relative to the top-level `orcsfile` is the same as `FILE`'s path relative to `$HOME` and add the necessary directives to the top-level orcsfile to create and maintain links to `FILE`'s previous location.

If `FILE` is a directory, the directory's contents will be tracked (see `link` directive, below).

This command only moves the file and updates the top-level orcsfile, so be sure to run `orcs` afterwards.

### orcsfile format

A list of directives, one per line, from the following set:

#### link

`link <REALFILE> <TARGET_PATH>`

`link` is the backbone of ORCS. It's actually possible to use ORCS only understanding the function of this directive, and ignoring all other functionality. `link` directs ORCS to creat a symlink pointing to `REALFILE` *relative to the directory containing the orcsfile in which the `link` directive is found* (unless `REALFILE` is an absolute path, in which case the absolute path is used), at the absolute path `TARGET_PATH`.

If `REALFILE` is a directory, `link` processes all children of that directory as if they were the target of their own `link` directive` (but see `unmanage`, below).

Example:

`link .config/nvim ~/.config/nvim`

#### unmanage

`unmanage <TARGET_PATH>`

`unmanage` directs ORCS to exclude from its processing a file previously included by a `link` directive. This can be useful in cases where e.g. where you include configuration for a tool included on most but not all of your machines.

Example:

`unmanage ~/.config/nvim/ginit.vim`

#### include

`include <ORCSFILE>`

`include` directs ORCS to read an orcsfile at the given path and process the contained commands before continuing processing the current orcsfile. Relative paths are supported and are relative to the current orcsfile.

Example:

`include ../otherhost/orcsfile`

