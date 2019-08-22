![](https://travis-ci.org/jlenv/julia-build.svg?branch=master)

# julia-build

Julia-build is a command-line utility tool that makes it easy to compile, 
install and remove virtually any version of [Julia](https://www.julialang.org), 
using downloaded source files.

Julia-build is exposed as a plugin for [jlenv](https://github.com/jlenv/julia-build)
that provides the `jlenv install` command.
Or simply as `julia-build` when used as a standalone program.

* [Installing](#installing)
* [Upgrading](#upgrading)
* [Troubleshooting](#troubleshooting)
* Writing [[custom build definitions|Definitions]]

  - [jlenv](https://github.com/jlenv/jlenv)
  - [Julia definitions](https://github.com/jlenv/julia-build/tree/master/share/julia-build)
  - [Wiki](https://github.com/jlenv/julia-build/wiki)

---

## Installing

```sh
# As an jlenv plugin
$ mkdir -p "$(jlenv root)"/plugins
$ git clone https://github.com/jlenv/julia-build.git "$(jlenv root)"/plugins/julia-build

# As a standalone program
$ git clone https://github.com/jlenv/julia-build.git
$ PREFIX=/usr/local ./julia-build/install.sh
```

### Upgrading

```sh
# As an jlenv plugin
$ cd "$(jlenv root)"/plugins/julia-build && git pull
```

## Suggested build environment

Julia-build will try its best to download and compile the wanted Julia version,
but sometimes compilation fails because of unmet system dependencies, or
compilation succeeds but the new Julia version exhibits weird failures at
runtime. The following instructions are our recommendations for a sane build
environment.

* **Chef (zero), etc.**
If you don't already, we suggest you manage your software installation
prerequisites, environment and configuration using Chef, Ansible, Salt, Puppet, etc.
In the case of [Chef](https://www.chef.io/) ([Zero](https://github.com/chef/chef-zero)) 
the [jlenv](https://github.com/jlenv/jlenv-cookbook) cookbook provides the 
[resources](https://docs.chef.io/resource.html) to write Julia installation 
management recipes.


* **macOS:**

  If you haven't done so, install Xcode Command Line Tools
  (`xcode-select --install`) and [Homebrew][https://brew.sh]. Then:

    ```sh
    brew install <package-dependencies>
    ```
    Where the package-dependencies are defined in the [jlenv-cookbook](https://github.com/jlenv/jlenv) 
    package [dependencies file](https://github.com/jlenv/jlenv-cookbook/blob/master/libraries/package_deps.rb#L38)

* **Ubuntu/Debian/Mint:**

    ```sh
    apt-get install -y <package-dependencies>
    ```
    Where the package-dependencies are defined in the [jlenv-cookbook](https://github.com/jlenv/jlenv) 
    package [dependencies file](https://github.com/jlenv/jlenv-cookbook/blob/master/libraries/package_deps.rb#L38)

* **CentOS/Fedora:**

    ```sh
    yum install -y <package-dependencies>
    ```
    Where the package-dependencies are defined in the [jlenv-cookbook](https://github.com/jlenv/jlenv) 
    package [dependencies file](https://github.com/jlenv/jlenv-cookbook/blob/master/libraries/package_deps.rb#L38)

* **openSUSE:**

    ```sh
    zypper install -y <package-dependencies>
    ```
    Where the package-dependencies are defined in the [jlenv-cookbook](https://github.com/jlenv/jlenv) 
    package [dependencies file](https://github.com/jlenv/jlenv-cookbook/blob/master/libraries/package_deps.rb#L38)

* **Arch Linux:**

T.B.A.

### Notes

#### GCC compatibility

Julia is currently built using gcc-5.

## Updating julia-build

If you have trouble installing a Julia version, first try to update julia-build to
get the latest bug fixes and Julia definitions.

First locate it on your system:

```sh
which julia-build
ls "$(jlenv root)"/plugins
```
<!---
If it's in `/usr/local/bin` on a Mac, you've probably installed it via
[Homebrew][https:brew.sh]:

```
brew upgrade julia-build
```
--->

Or, if you have it installed via git as an jlenv plugin:

```sh
cd "$(jlenv root)"/plugins/julia-build && git pull
```

## Troubleshooting

### "mkdir: /Volumes/Macintosh: Not a directory"

This can occur if you have [more than one disk drive][disks] and your home directory is physically mounted on a volume that might have a space in its name, such as "Macintosh HD":

```
$ df
/dev/disk2    ...  /
/dev/disk1s2  ...  /Volumes/Macintosh HD
```

The easiest solution is to avoid building a Julia version to any path that has space characters in it. So instead of building into `~/.jlenv/versions/<version>`, which is the default for `jlenv install <version>`, instead you could install into `/opt/julias/<version>` and symlink `/opt/julias` as `~/.jlenv/versions` so jlenv continues to work as before:

```sh
sudo mkdir -p /opt/julias
sudo chown "${USER}:staff" /opt/julias
rm -rf ~/.jlenv/versions                 # This will DELETE your existing Julia versions!
ln -s /opt/julias ~/.jlenv/versions
```

Now proceed as following:

```
julia-build <version> /opt/julias/<version>
```

[disks]: https://github.com/sstephenson/ruby-build/issues/748#issuecomment-95143238

### No space left on device

Some distributions will mount a tmpfs partition with low disk space to `/tmp`, such as 250 MB. You can check this with:

    mount | grep tmp
    df -h | grep tmp

Compiling Julia can require up to 4 or 5GiB, so you should temporarily resize `/tmp` to allow more usage:

    rm -rf /tmp/julia-build*
    mount -o remount,size=5G,noatime /tmp

### Lower the number of parallel jobs

On hosts that report a large amount of CPU cores, but don't have plenty of RAM, you might get:

```
gcc: internal compiler error: Killed (program cc1)
```

The solution is to use `MAKE_OPTS=-j2` to limit `make` to maximum of 2 parallel processes:

```bash
export MAKE_OPTS=-j2
```

or also with writable temp directory:

```bash
TMPDIR=~/tmp MAKE_OPTS=-j2 jlenv install 1.0.3
```

## Usage

### Basic Usage

```sh
# As an jlenv plugin
$ jlenv install --list                 # lists all available versions of Julia
$ jlenv install v0.6.0                  # installs Julia v0.6.0 to ~/.jlenv/versions

# As a standalone program
$ julia-build --definitions             # lists all available versions of Julia
$ julia-build 0.6.0 ~/local/julia-v0.6.0  # installs Julia v0.6.0 to ~/local/julia-0.6.0
```

julia-build does not check for system dependencies before downloading and
attempting to compile the Julia source. Please ensure that [all required
libraries](https://github.com/JuliaLang/julia#required-build-tools-and-external-libraries) are available on your system.

### Advanced Usage

#### Custom Build Definitions

If you wish to develop and install a version of Julia that is not yet supported
by julia-build, you may specify the path to a custom “build definition file” in
place of a Julia version number.

Use the [default build definitions][definitions] as a template for your custom
definitions.

#### Custom Build Configuration

The build process may be configured through the following environment variables:

| Variable                 | Function                                                                                         |
| ------------------------ | ------------------------------------------------------------------------------------------------ |
| `TMPDIR`                 | Where temporary files are stored.                                                                |
| `JULIA_BUILD_BUILD_PATH`  | Where sources are downloaded and built. (Default: a timestamped subdirectory of `TMPDIR`)        |
| `JULIA_BUILD_CACHE_PATH`  | Where to cache downloaded package files. (Default: `~/.jlenv/cache` if invoked as jlenv plugin)  |
| `JULIA_BUILD_MIRROR_URL`  | Custom mirror URL root.                                                                          |
| `JULIA_BUILD_SKIP_MIRROR` | Always download from official sources, not mirrors. (Default: unset)                             |
| `JULIA_BUILD_ROOT`        | Custom build definition directory. (Default: `share/julia-build`)                                 |
| `JULIA_BUILD_DEFINITIONS` | Additional paths to search for build definitions. (Colon-separated list)                         |
| `CC`                     | Path to the C compiler.                                                                          |
| `JULIA_CFLAGS`            | Additional `CFLAGS` options (_e.g.,_ to override `-O3`).                                         |
| `CONFIGURE_OPTS`         | Additional `./configure` options.                                                                |
| `MAKE`                   | Custom `make` command (_e.g.,_ `gmake`).                                                         |
| `MAKE_OPTS` / `MAKEOPTS` | Additional `make` options.                                                                       |
| `MAKE_INSTALL_OPTS`      | Additional `make install` options.                                                               |
| `JULIA_CONFIGURE_OPTS`    | Additional `./configure` options (applies only to Julia source).                                 |
| `JULIA_MAKE_OPTS`         | Additional `make` options (applies only to Julia source).                                        |
| `JULIA_MAKE_INSTALL_OPTS` | Additional `make install` options (applies only to Julia source).                                |

#### Applying Patches

Both `jlenv install` and `julia-build` support the `--patch` (`-p`) flag to apply
a patch to the Julia source code before building. Patches are
read from `STDIN`:

```sh
# applying a single patch
$ jlenv install --patch 1.9.3-p429 < /path/to/julia.patch

# applying a patch from HTTP
$ jlenv install --patch 1.9.3-p429 < <(curl -sSL http://git.io/julia.patch)

# applying multiple patches
$ cat fix1.patch fix2.patch | jlenv install --patch 1.9.3-p429
```

#### Checksum Verification

If you have the `shasum`, `openssl`, or `sha256sum` tool installed, julia-build will
automatically verify the SHA2 checksum of each downloaded package before
installing it.

Checksums are optional and specified as anchors on the package URL in each
definition. (All bundled definitions include checksums.)

#### Keeping the build directory after installation

Both `julia-build` and `jlenv install` accept the `-k` or `--keep` flag, which
tells julia-build to keep the downloaded source after installation. This can be
useful if you need to use `gdb` and `memprof` with Julia.

Source code will be kept in a parallel directory tree `~/.jlenv/sources` when
using `--keep` with the `jlenv install` command. You should specify the
location of the source code with the `JULIA_BUILD_BUILD_PATH` environment
variable when using `--keep` with `julia-build`.

## Getting Help

Please see Julia-Build wiki for solutions to common problems.

  [jlenv]: https://github.com/jlenv/jlenv
  [definitions]: https://github.com/jlenv/julia-build/tree/master/share/julia-build
  [wiki]: https://github.com/jlenv/julia-build/wiki
