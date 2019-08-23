![](https://travis-ci.org/jlenv/julia-build.svg?branch=master)

# julia-build

Julia-build is a command-line utility tool that makes it easy to compile, 
install and remove virtually any version of [Julia](https://www.julialang.org), 
using downloaded source files.

Julia-build is exposed as a plugin for [jlenv](https://github.com/jlenv/julia-build)
that provides the `jlenv install` command.
Or simply as `julia-build` when used as a standalone program.

# Table of Contents

<!--tc-->
  * [Installation](#installation)
  * [Upgrading](#upgrading)
  * [Usage](#usage)
  * [Documentation](#documentation)
<!--te-->

---

julia-build is a command-line utility that makes it easy to install virtually any
version of Julia, from source.

It is available as a plugin for [jlenv](https://github.com/jlenv/julia-build) that
provides the `jlenv install` command, or as a standalone program.

## Installation

```sh
# As an jlenv plugin
$ mkdir -p "$(jlenv root)"/plugins
$ git clone https://github.com/jlenv/julia-build.git "$(jlenv root)"/plugins/julia-build

# As a standalone program
$ git clone https://github.com/jlenv/julia-build.git
$ PREFIX=/usr/local ./julia-build/install.sh
```

## Upgrading

```sh
# As an jlenv plugin
$ cd "$(jlenv root)"/plugins/julia-build && git pull
```

## Usage

```sh
# As an jlenv plugin
$ jlenv install --list                    # lists all available versions of Julia
$ jlenv install v0.6.0                    # installs Julia v0.6.0 to ~/.jlenv/versions

# As a standalone program
$ julia-build --definitions               # lists all available versions of Julia
$ julia-build 0.6.0 ~/local/julia-v0.6.0  # installs Julia v0.6.0 to ~/local/julia-0.6.0
```

julia-build does not check for system dependencies before downloading and
attempting to compile the Julia source. Please ensure that [all requiredlibraries](https://github.com/JuliaLang/julia#required-build-tools-and-external-libraries) 
are available on your system.
The Chef [jlenv](https://github.com/jlenv/jlenv-cookbook) cookbook automatically
ensures system dependencies are in place.

## Documentation

Please see `julia-build` [documentation page](https://julia-build.github.io/docs/README.md) 
for more detail and solutions to common problems.
