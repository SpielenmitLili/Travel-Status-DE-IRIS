# db-iris - Commandline Client for DB IRIS Departure Monitor

db-iris is a commandline client and Perl module for the DB IRIS departure
monitor located at iris.noncd.db.de. See the [Travel::Status::DE::IRIS
homepage](https://finalrewind.org/projects/Travel-Status-DE-IRIS/) for details.

It provides both human- and machine-readable access to departure data at german
train stations via text and JSON output.

## Installation

You have four installation options:

* Nightly `.deb` builds for Debian-based distributions
* Installing the latest release from CPAN
* Installation from source
* Using a Docker image

Except for Docker, **db-iris** is available in your PATH after installation.
You can run `db-iris --version` to verify this. Documentation is available via
`man db-iris`.

### Nightly Builds for Debian

[lib.finalrewind.org/deb](https://lib.finalrewind.org/deb) provides Debian
packages of both development and release versions. Note that these are not part
of the official Debian repository and are not covered by its quality assurance
process.

To install the latest release, run:

```
wget https://lib.finalrewind.org/deb/libtravel-status-de-iris-perl_latest_all.deb
sudo dpkg -i libtravel-statusg-de-iris-perl_latest_all.deb
sudo apt --fix-broken install
rm libtravel-status-de-iris-perl_latest_all.deb
```

For a (possibly broken) development snapshot of the Git master branch, run:

```
wget https://lib.finalrewind.org/deb/libtravel-status-de-iris-perl_dev_all.deb
sudo dpkg -i libtravel-status-de-iris-perl_dev_all.deb
sudo apt --fix-broken install
rm libtravel-status-de-iris-perl_dev_all.deb
```

Note that dpkg, unlike apt, does not automatically install missing
dependencies. If a dependency is not satisfied yet, `dpkg -i` will complain
about unmet dependencies and bail out. `apt --fix-broken install` installs
these dependencies and also silently finishes the Travel::Status::DE::IRIS
installation.

Uninstallation works as usual:

```
sudo apt remove libtravel-status-de-iris-perl
```

### Installation from CPAN

Travel::Status::DE::IRIS releases are published on the Comprehensive Perl
Archive Network (CPAN) and can be installed using standard Perl module tools
such as `cpanminus`.

Before proceeding, ensure that you have standard build tools (i.e. make,
pkg-config and a C compiler) installed. You will also need the following
libraries with development headers:

* libdb
* libssl
* libxml2
* zlib

Now, use a tool of your choice to install the module. Minimum working example:

```
cpanm Travel::Status::DE::IRIS
```

If you run this as root, it will install script and module to `/usr/local` by
default.

### Installation from Source

In this variant, you must ensure availability of dependencies by yourself.
You may use carton or cpanminus with the provided `Build.PL`, Module::Build's
installdeps command, or rely on the Perl modules packaged by your distribution.
On Debian 10+, all dependencies are available from the package repository.

To check whether dependencies are satisfied, run:

```
perl Build.PL
```

If it complains about "... is not installed" or "ERRORS/WARNINGS FOUND IN
PREREQUISITES", it is missing dependencies.

Once all dependencies are satisfied, use Module::Build to build, test and
install the module. Testing is optional -- you may skip the "Build test"
step if you like.

If you downloaded a release tarball, proceed as follows:

```
./Build
./Build test
sudo ./Build install
```

If you are using the Git repository, use the following commands:

```
./Build
./Build manifest
./Build test
sudo ./Build install
```

If you do not have superuser rights or do not want to perform a system-wide
installation, you may leave out `Build install` and use **db-iris** from the
current working directory instead.

With carton:

```
carton exec db-iris --version
```

Otherwise (also works with carton):

```
perl -Ilocal/lib/perl5 -Ilib bin/db-iris --version
```

### Running db-iris via Docker

A db-iris image is available on Docker Hub. It is intended for testing
purposes: due to the latencies involved in spawning a container for each
db-iris invocation, and the lack of persistent caching, it is less convenient
for day-to-day usage.

Installation:

```
docker pull derfnull/db-iris:latest
```

Use it by prefixing db-iris commands with `docker run --rm
derfnull/db-iris:latest`, like so:

```
docker run --rm derfnull/db-iris:latest --version
```

Documentation is not available in this image. Please refer to the
[online db-iris manual](https://man.finalrewind.org/1/db-iris/) instead.


## Managing stations

Travel::Status::DE::IRIS needs a list of train stations to operate, which is
located in `share/stations.json`. There are two recommended editing methods.

Automatic method, e.g. to incorporate changes from Open Data sources:

* modify stations.json with a script in any JSON-aware language you like
* run `./json2json` in the share diretcory. This performs consistency checks and
  transforms stations.json into its canonical format, which simplifies tracking
  of changes and reduces diff size

Manual method:

* run `./json2csv` in the share directory
* modify stations.csv automatically or manually (e.g. with LibreOffice Calc)
* run `./csv2json` in the share directory

If the changes you made are suitable for inclusion in Travel::Status::DE::IRIS,
please [open a pull request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork) afterwards.

Please only include stations which are usable with DB IRIS, that is, which have
both DS100 and EVA numbers. If

```
curl -s https://iris.noncd.db.de/iris-tts/timetable/station/EVANUMBER
```

and

```
curl -s https://iris.noncd.db.de/iris-tts/timetable/station/DS100
```

return a `<station>` element with "name", "eva" and "ds100" attributes, you're
good to go.

Note that although EVA numbers are often identical with UIC station IDs, this
is not always the case. Please do not rely on UIC IDs when managing stations.
