# uperl-perlbuild

A drop-in replacement for the `perl-build` command from the
[`Perl-Build`](https://metacpan.org/dist/Perl-Build) CPAN distribution.

It fetches, configures, builds, and installs a perl from source. The heavy
lifting is done by the [`perl-build`](https://github.com/uperl/rust-perl-build)
Rust crate (a port of `Perl::Build`); this binary is just the command-line
front end.

## Usage

```
uperl-perlbuild [options] <stuff> <destination> [-- <configure options>...]
```

`<stuff>` is one of:

| form | example | how it is handled |
| --- | --- | --- |
| a version | `5.40.2` | resolved to a CPAN archive through MetaCPAN |
| a URL | `https://.../perl-5.40.2.tar.gz` | downloaded, then built |
| a tarball | `path/to/perl-5.40.2.tar.gz` (`.gz` / `.bz2` / `.xz`) | built from the local file |
| `blead` | `blead` | `github.com/Perl/perl5` tip, built with `-Dusedevel` |

`<destination>` is the install prefix (`-Dprefix`); a relative path is made
absolute against the current directory.

Anything after `<destination>` (or after a literal `--`) is passed straight to
`./Configure`. With nothing given, `-de` is used.

```sh
uperl-perlbuild 5.40.2 /opt/perl-5.40.2
uperl-perlbuild 5.40.2 /opt/perl-5.40.2-threads -- -de -Dusethreads
uperl-perlbuild ~/src/perl-5.40.2.tar.xz /opt/perl-5.40.2
uperl-perlbuild blead /opt/perl-blead
```

### Options

| option | effect |
| --- | --- |
| `-D <define>` | append `-D<define>` to the `./Configure` options (repeatable) |
| `-A <append>` | append `-A<append>` to the `./Configure` options (repeatable) |
| `-U <undef>` | append `-U<undef>` to the `./Configure` options (repeatable) |
| `--test` / `--no-test` | run the test suite after building (default: off) |
| `-j`, `--jobs <n>` | build and test with `<n>` parallel jobs (default: the number of detected processor threads) |
| `--build-dir <dir>` | unpack and build here (default: a temporary directory) |
| `--tarball-dir <dir>` | download source tarballs here (default: a temporary directory) |
| `--patches <plugin>` | set `PERL5_PATCHPERL_PLUGIN` for `patchperl` |
| `--symlink-devel-executables` | symlink versioned dev executables (`perl5.41.0` → `perl`) |
| `--noman` | skip manpages (adds `-Dman1dir=none -Dman3dir=none`) |
| `--definitions` | list the perl versions available on CPAN, then exit |
| `--version` | print version information, then exit |
| `-h`, `--help` | print help, then exit |

`-D`, `-A`, and `-U` accept `-Dfoo`, `-D foo`, and `-D=foo`. Unrecognised
options are passed through as positional arguments, matching the original's
`Getopt::Long` `pass_through` behaviour.

## `patchperl`

`Perl::Build` applies
[`Devel::PatchPerl`](https://metacpan.org/pod/Devel::PatchPerl) source fix-ups
before `./Configure`. There is no Rust port of that logic, so the backend
shells out to the `patchperl` program when it is found on `PATH`; without it,
older perls may fail to build on a modern toolchain. Install it with:

```sh
cpanm App::patchperl
```

## Differences from `perl-build`

* Version resolution and release listings come from MetaCPAN
  (`--definitions` queries the API rather than a bundled table), so they need
  network access.
* Tarballs are detected by a `.gz` / `.bz2` / `.xz` suffix, exactly as the
  original; `.tgz` is treated as a version string.
* `--version` reports this tool and whether `patchperl` was found, rather than
  the `Devel::PatchPerl` module version.
* `-j` / `--jobs` defaults to the number of processor threads detected at
  runtime; the original builds serially unless `-j` is given.

## License

MIT
