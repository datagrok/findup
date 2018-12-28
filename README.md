# findup

**findup** ("find up") locates a given filename in the nearest ancestor directory.

```
Usage: findup [-C DIRECTORY] FILENAME

Look for FILENAME in the current or given DIRECTORY and all of its
ancestors until found. Return the full path of the closest match on
standard output.
```

## Examples

Imagine we keep multiple projects in some directory, `My Projects`. Here we have a `Python Project` (initialized by virtualenv) and a `C Project`:

```
/path/to/My Projects/
├── C Project/
│   ├── .git/
│   ├── Makefile
│   ├── lib/
│   └── usr/
│       └── lib/
│           └── libfoo/
└── Python Project/
    ├── bin/
    │   ├── activate
    │   └── python2
    ├── include/
    ├── lib/
    └── Web Assets/
```

### Recompile code from anywhere within the project directory

Imagine we're on the command line deep within our project's hierarchy, hacking on source code. After each edit, we want to run `make` to recompile. But Make has to be told where its `Makefile` is located.
```
$ pwd
/path/to/My Projects/C Project/usr/lib/libfoo
$ findup Makefile
/path/to/My Projects/C Project/Makefile
```
So we can use this command to compile our project from any directory within:
```
$ make -f "$(findup Makefile)"
```
Note: many makefiles are written with the assumption that the current directory is the one containing the makefile. So it may be better to run:
```
$ make -C "$(dirname "$(findup Makefile)")"
```
To accomplish the original goal -- less typing -- we can make that into an alias. Set into `~/.bash_aliases`:
```
alias rmake='make -C "$(dirname "$(findup Makefile)")"'
```

### Activate a virtualenv from anywhere within it

Imagine we're on the command line deep within our Python project's hierarchy, hacking on source code. Then we remember we forgot to `activate` the virtualenv.

```
$ pwd
/path/to/My Projects/Python Project/Web Assets
$ findup bin/activate
/path/to/My Projects/Python Project/bin/activate
```
So we can use this command to activate the current virtualenv from any directory within:
```
$ source "$(findup bin/activate)"
```
Note here that you may specify a filename containing multiple subdirectories from the ancestor.

### Locate the `.git` directory in this repository

```
$ echo "The git directory is: "$(findup .git)""
```
Note: `git` has a more robust built-in mechanism to do this: `git rev-parse --git-dir`


### Locate the root of this git repository

```
$ echo "The root of this repository is: $(findup .git/..)"
```
Note: `git` has a more robust built-in mechanism to do this: `git rev-parse --show-toplevel`

### Use `incontext`

My tool [incontext](https://github.com/datagrok/incontext) helps manage project-specific environment settings, automatically activating virtualenvs, and the like. It uses `findup` to do so.

## Dependencies

- a POSIX shell, like `bash` or `dash`.
- coreutils `readlink`

## Implementation decisions

This version is implemented in shell script, but I would prefer a safer, faster, compiled language. I have begun alternative implementations:

- [findup in Rust](https://github.com/datagrok/findup-rs)
- [findup in C](https://github.com/datagrok/findup-c), using GNUlib.

Whichever implementation language I use, I want installs to:

- require few dependencies that are not already installed on most systems. That limits me to C, Rust, Golang, shell script, and maybe Python.
- I'd prefer one that doesn't create binaries that are 10s of megabytes large. (Golang...)
- I'd also consider a Guile Scheme implementation because I hope in vain for Guile Scheme to replace POSIX shell as the de-facto always-available system programming language for Unix machines.

## Other tools like findup:

`findup` is a trivially simple tool that seems to have no standard implementation. So there are a proliferation of independent implementations. I found these from a few minutes of searching "find up" on GitHub:

- [jlindsey/findup](https://github.com/jlindsey/findup) C; MIT. This almost looks like what I want but as far as I can tell, the implementation is missing.
- [h2non/findup.rs](https://github.com/h2non/findup.rs) rust; MIT; library.
- [h2non/findup](https://github.com/h2non/findup) golang; MIT; library.
- [todddeluca/python-findup](https://github.com/todddeluca/python-findup) python; MIT; library.
- [Filirom1/findup](https://github.com/Filirom1/findup) javascript.
- [goblindegook/findup-node-modules](https://github.com/goblindegook/findup-node-modules) javascript; library, not general-purpose.
- [jonschlinkert/find-file-up](https://github.com/jonschlinkert/find-file-up) javascript; library.
- [jonschlinkert/find-pkg](https://github.com/jonschlinkert/find-pkg) javascript; not general-purpose.
- [js-cli/node-findup-sync](https://github.com/js-cli/node-findup-sync) javascript; library.
- [laat/upfind/blob/master/lib/upfind.ts](https://github.com/laat/upfind/blob/master/lib/upfind.ts) proprietary; typescript; library.
- [robrichard/find-up-directory-tree](https://github.com/robrichard/find-up-directory-tree) javascript; library.
- [shannonmoeller/find-config](https://github.com/shannonmoeller/find-config) javascript; library. Feature: searches in XDG_CONFIG_DIRS.
- [sindresorhus/find-up-cli](https://github.com/sindresorhus/find-up-cli) javascript; MIT.
- [tanhauhau/find-up-glob](https://github.com/tanhauhau/find-up-glob) javascript.
- [ysugimoto/artisan-findup](https://github.com/ysugimoto/artisan-findup) golang; not general-purpose

Labels used above:

- library: the package is intended to be used as a library, so it is useful only to other software written in the same language. It has no CLI that we can call from other programs or shell scripts.
- not general-purpose: the code does something similar to mine but makes some assumptions that are specific to some use-case. We can't use it for the general purpose of "find the named file in the nearest ancestor directory."
- rust: Rust is great! I'd prefer an implementation in Rust, but I don't know how to package a Rust program for my linux distribution.
- javascript: I don't want to introduce a heavyweight dependency on npm or node.
- golang: the lack of dependencies is a plus, but I kindof dislike the idea of having a 50MB executable for such a simple utility.

## License: AGPL-3.0+

All of the code herein is copyright 2017 [Michael F. Lamb](http://datagrok.org) and released under the terms of the [GNU Affero General Public License, version 3][AGPL-3.0+] (or, at your option, any later version.)

[AGPL-3.0+]: http://www.gnu.org/licenses/agpl.html
