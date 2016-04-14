# KIWI — Next Generation

[![Build Status](https://travis-ci.org/SUSE/kiwi.svg?branch=master)](https://travis-ci.org/SUSE/kiwi)
[![Health](https://landscape.io/github/SUSE/kiwi/master/landscape.svg?style=flat)](https://landscape.io/github/SUSE/kiwi/master)

## Contents

  * [Motivation](#motivation)
  * [Installation](#installation)
    - [Compatibility](#compatibility)
    - [Example Appliance Descriptions](#example-appliance-descriptions)
  * [Quick Start](#quick-start)
  * [Supported Distributions](#supported-distributions)
    - [Dropped Features](#dropped-features)
  * [Building in the buildservice](#building-in-the-buildservice)
  * [Contributing](#contributing)
  * [Developing](#developing)
  * [Packaging and Versioning](#packaging-and-versioning)
  * [Documentation](#documentation)
  
## Motivation

The idea of KIWI is simple: Provide a human readable appliance
description and build the system for any kind of target or service.

Since the early days the project was well received and within SUSE
all product appliances, product media, appliances for private and
public cloud as well as on top projects like SUSE Studio uses KIWI.
New opportunities with partners, other distribution vendors and
technologies are ahead. However, is KIWI really well prepared for
future challenges ?

To my mind the former [KIWI](https://github.com/openSUSE/kiwi) version
has some major weaknesses which has to be fixed prior to continue
future development:

* Not based on a modern programming language
* Major design flaws but hardly any unit tests
* No arch specific build integration tests
* Lots of legacy code for old distributions

In order to address all of these the question came up:

How to modernize the project without producing regressions or
making any user unhappy ?

As I did not see a good way to achieve this in the former code
base the decision to start a rewrite of KIWI with a maintained
and stable version in the background was made.

After some coffee, lots of hacking hours and peanuts later,
I'm happy to introduce this next generation KIWI project to you.

Users will be able to use both versions in parallel. In addition
the next generation KIWI will be fully compatible with the current
format of the appliance description. This means, users can build an
appliance from the same appliance description with the legacy and
the next generation KIWI, if the distribution and all configured
features are supported by the used KIWI version.

This provides an opportunity for users to test the next generation
KIWI with their appliance descriptions without risk. If it builds
and works as expected I recommend to switch to the next generation
KIWI, if not please open an Issue on https://github.com/SUSE/kiwi.

## Installation

Packages for the new KIWI version are provided at the
[openSUSE buildservice](https://build.opensuse.org/package/show/Virtualization:Appliances:Builder/python3-kiwi)

Add the repository with `zypper ar` (see following code) and replace the
distribution placeholder. The best approach is to click on the desired
distribution from the buildservice link above and there follow the
__Go to download repository__ link

```bash
$ sudo zypper ar -f \
    http://download.opensuse.org/repositories/Virtualization:/Appliances:/Builder/<DIST>
$ sudo zypper in python3-kiwi
```

Please note the package uses the alternatives mechanism to setup
a symbolic link named `kiwi` to the real executable named `kiwi-py3`.
If the link target `/usr/bin/kiwi` already exists on your system, the
alternative setup will skip the creation of the link target because it
already exists.

### Compatibility

The legacy KIWI version can be installed and used together with
the next generation KIWI. The python3-kiwi package makes use
of the alternatives mechanism as follows. If both versions are
installed users will reach the legacy KIWI with

```
$ sudo kiwi ...
```

and the next generation KIWI with

```
$ sudo kiwi-py3 ...
```

If only the next generation kiwi is installed the alternatives link __kiwi__
exists. The next generation KIWI provides a completion like we have it
in the legacy KIWI too. However if both versions are installed the
completion for the next generation KIWI conflicts with the legacy KIWI
and thus can only be reached if invoked with __sudo kiwi-py3__
because of the __kiwi__ name conflict.

From an appliance description perspective both KIWI versions are fully
compatible. Users can build their appliances with both versions and the
same appliance description. If the appliance description uses features the
next generation KIWI does not provide, the build will fail with an exception
early. If the appliance description uses next generation features like the
selection of the initrd system, it's not possible to build that with the
legacy KIWI, unless the appliance description properly encapsulates the
differences into a profile.

The next generation KIWI also provides the __--compat__ option and the
__kiwicompat__ tool to be able to use the same commandline as provided
with the legacy KIWI version.

### Example Appliance Descriptions

For use with the next generation KIWI there is also a GitHub project hosting
example appliance descriptions. Users who need an example to start with
should checkout the project as follows:

```bash
$ git clone https://github.com/SUSE/kiwi-descriptions
```

## Quick Start

Once installed building and testing an image from the Examples
GitHub can be done as follows:

```bash
$ sudo kiwi-py3 --type vmx system build \
       --description kiwi-descriptions/suse/x86_64/suse-leap-42.1-JeOS \
       --target-dir /tmp/myimage

$ cd /tmp/myimage

$ qemu -drive \
    file=LimeJeOS-Leap-42.1.x86_64-1.42.1.raw,format=raw,if=virtio
```

## Supported Distributions

The next generation KIWI can build appliances for distributions
which are equal or newer compared to the following list:

* SUSE Linux Enterprise 12
* Red Hat Enterprise 7
* openSUSE 13.2
* openSUSE Leap 42
* openSUSE Tumbleweed

For anything older please consider to use the legacy
KIWI version __v7.x.x__

### Dropped Features

The following features have been dropped:

* Split systems

  The legacy kiwi version supports building of split systems which
  uses a static definition of files and directories marked as read-only
  or read-write. Evolving technologies like overlayfs makes this
  feature obsolete.

* ZFS filesystem

  The successor for zfs is btrfs in the opensource world. All major
  distributions put on btrfs. This and the proprietary attitude of
  zfs obsoletes the feature.

* Reiserfs filesystem

  The number of people using this filesystem is decreasing. For
  image building reiserfs was an interesting filesystem however with
  btrfs and xfs there are good non inode based alternatives out there.
  Therefore we don't continue supporting reiserfs.

* Btrfs seed based live systems

  A btrfs seed device is an alternative for other copy on write
  filesystems like overlayfs. Unfortunately the stability of the seed
  device when used as cow part in a live system was not as good as
  we provide with overlayfs and clicfs. Therefore this variant is
  no longer supported. We might think of adding this feature back
  if people demand it.

* VDI image subformat

  The vdi disk image format is supported by the legacy kiwi version
  but we are not aware of any user. The missing business perspective
  makes this feature obsolete.

## Building in the buildservice

The next generation KIWI is fully integrated with the buildservice.
As an example you can find the integration testing system in the
buildservice here:

https://build.opensuse.org/project/subprojects/Virtualization:Appliances:Images

In order to use the next generation KIWI to build an appliance in the
buildservice it is only required to add the Builder project as repository
to the KIWI XML configuration like in the following example:

```xml
<repository type="rpm-md" alias="kiwi-next-generation">
    <source path="obs://Virtualization:Appliances:Builder/SLE_12_SP1"/>
</repository>
```

The Builder project configuration in the buildservice is setup to
prefer the next generation KIWI over the legacy version. Thus adding
the Builder repository inherits this project setup and activates
building with the next generation KIWI.

## Contributing

The core appliance builder is developed in Python and follows the
test driven development rules. The XML, schema, and stylesheets are
taken from the old version of KIWI. Also the entire boot code
(written in bash) is taken from the old KIWI codebase.

The Python project uses `pyvenv` to setup a development environment
for the desired Python version. The script `pyvenv` is already
installed when using Python 3.3 and higher (see https://docs.python.org/3.3/whatsnew/3.3.html#pep-405-virtual-environments for details).

The following procedure describes how to create such an environment:

1. Create the virtual environment:

   ```
$ pyvenv-3.4 .env3
```

2. Activate the virtual environment:

   ```
$ source .env3/bin/activate
```

3. Install KIWI requirements inside the virtual environment:

   ```
$ pip3.4 install -r .virtualenv.dev-requirements.txt
```

4. Install KIWI in "development mode":

   ```
$ ./setup.py develop
```

You're done!

Once the development environment is activated and initialized with
the project required Python modules, you are ready to work.

The __develop__ target of the `setup.py` script automatically creates
the application entry point called `kiwi`, which allows to simply
call the application from the current code base:

   ```
$ kiwi-py3 --help
```

In order to leave the development mode just call:

```
$ deactivate
```

To resume your work, change into your local Git repository and
run `source .env3/bin/activate` again. Skip step 3 and 4 as
the requirements are already installed.


## Developing and Running Test Cases

When developing KIWI, the preferred method is to use Tox:

```
$ tox
```

The previous call would run `tox` for different Python versions,
checks the source code for errors, and builds the documentation.

If you want to see the target, use the option `-l` to print a list:

```
$ tox -l
```

To only run a special target, use the `-e` option. The following
example runs the test cases for the 3.4 interpreter only:

```
$ tox -e 3.4
```

## Packaging and Versioning

The version schema is based on `bumpversion` and follows the
standard rules as shown below.

* For backwards-compatible bug fixes

```
$ bumpversion patch
```

* For additional functionality in a backwards-compatible manner.
  When changed set the patch level back to zero

```
$ bumpversion minor
```

* For incompatible API changes. When changed set the patch and minor
  level back to zero

```
$ bumpversion major
```

The creation of RPM package sources has to be done by calling
the following make target:

```
$ make build
```

The sources are collected below the `dist/` directory. In there you
will find all required files to submit a package to the Open Build
Service or just build it with `rpmbuild`.


## Documentation

The documentation is implemented as manual pages based on Sphinx
using the ReST markup. In order to build the manual pages for testing
just call:

```
$ cd doc
$ make man
```
