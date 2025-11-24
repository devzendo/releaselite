# releaselite

'releaselite' or 'rl' (from the rl.py file found in this repository)
is a simple, cross-platform build orchestration/lifecycle management
tool. It has similar goals to Maven's project lifecycle management, but
is maybe a simpler option for non-Java/JVM-based projects. It can be used
for projects in any language, and handles the top-level phases of working with a
base of software, and various lifecycle phases, such as:
* cleaning
* deep-cleaning
* preparing the source tree for building
* compiling the main source code
* compiling the unit- and integration tests, then executing them.
* packaging the main code into an archive of some form.
* deploying (publishing) the archive to some remote repository.
* handling the version number of the project, and releasing/tagging
  archives of new releases of the project.

It does not concern itself with dependencies, since your language-specific
tool will probably handle that adequately (npm, cmake, maven, gradle, etc.)

It does not know how to actually execute any of the above phases - you need to script
these operations. It merely runs the various phases on demand, and in order.

rl.py is a Python 3 script that requires no external dependencies - it
only uses the Python standard library. You may commit this script to your
project repository as necessary.

It handles version management of your project, assuming you follow semantic
versioning with an optional -SNAPSHOT qualifier, like Maven.

It assumes you are using git as your version control system.

# Motivation
I was using Maven to provide lifecycle management and a release mechanism in
a medium-sized C++/CMake project (transputer-emulator). Maven is excellent, and
the ideal tool for this in a JVM-based project - but is a heavyweight tool for
non-JVM projects like this one. 

I looked at a number of other tools, for example, Task (https://taskfile.dev/)
and Just (https://just.systems/man/en/) - and while they are great tools for
running arbitrary commands, they don't have an idea of a lifecycle, nor a 
release process. Task is a single binary, written in go - this ease of use is
appealing. Just, however requires the NodeJS ecosystem... which is not appealing.
I wanted to get away from requiring a huge messy ecosystem to support a simple
top-level build orchestrator.

Since all modern respectable development systems have Python 3 installed, I
chose this as the implementation language - but strictly no external dependencies
required, as the Python dependency ecosystem is... well, you know.

I know Python, but writing this tool would take time away from my main projects,
and since I have a self-imposed deadline for them, I used Anthropic's Claude.AI
to write the first version of the tool. There are risks using LLM-generated
software (or any output from an LLM). I have reviewed this code, and will make
any changes I need to correct it and for future enhancements.


# Configuration
With 'rl.py' committed to the top directory of your project, create its
configuration file, 'build.conf'.

This file allows you to declare which phases of the lifecycle you want to
provide scripts/executions for. Blank lines, and everything following a
hash (#) are ignored.

The version of your project may be declared thusly:
```
version: 1.2.3-SNAPSHOT
```
The version number follows semantic versioning, with an optional '-SNAPSHOT'
suffix, denoting works in progress. This is used for the 'release' phase, see
later.

Then, phase commands may be specified, with optional platform qualifiers:
```
clean:x86_64-apple-darwin: rm -rf cmake-build-debug; rm -rf cmake-build-release
clean:x86_64-pc-windows-msvc: Remove-Item -Recurse -Force cmake-build-debug; Remove-Item -Recurse -Force cmake-build-release
```

So if you run `rl.py clean`, the appropriate POSIX shell or PowerShell command
will run, depending if you are on Intel macOS, or Windows.

If you're only working on one platform (say, Linux), the platform can be omitted:
```
clean: rm -rf cmake-build-debug; rm -rf cmake-build-release
```

Variables can be given on the 'rl.py' command line, and these can be used in the
commands. The version declared in the configuration file can be referenced by the
`$VERSION` variable. Environment variables can be referenced using the `$ENV{USER}`
interpolation syntax. For example, if Susan Hargreaves (user name s.hargreaves)
runs `rl.py -DAuthor=Susan prepare` with a 'build.conf' containing:
```
version: 7.2
prepare: echo "const char* version=\"Myprogram by $Author ($ENV{USER}): $VERSION\";" > version.h 
```
This will create a 'version.h' file containing
```
const char* version="Myprogram by Susan (s.hargreaves): 7.2";
```
(The quoting is unlikely to work - this will probably require more work in rl.py -
but the intention is there!)

## Phases
There are two types of phase: standalone, and lifecycle.

Execution of 'rl.py' will fail if any of the commands you give for any of the
phases fail.

### Standalone phases
These are phases: 'clean', 'deep-clean', and 'release'.
Use them to clean your tree partially or completely. The release phase is special,
and will be described in a later section.

### Lifecycle phases
These are phases: 'prepare', 'test-compile', 'compile', 'test', 'package',
'integration-test', and 'deploy'.

Lifecycle phases form a chain from start to finish. On the command line, you
state which phase you want to run, and 'rl.py' will execute all phases up to
and including the phase you give. Any phases you don't declare are ignored -
you do not have to configure every phase, if you don't have use for them.

If you want to run a phase in the middle of the lifecycle, but not its preceding
phases, add the '--direct' option to the command line. e.g. 'rl.py --direct test'
to run just the tests, without going through 'prepare', 'test-compile', 'compile'
phases first.


The prepare phase is intended to set your tree up so that compilation can
proceed - perhaps as the example above, you write the project version number
into a source file for compilation, or generate a buildsystem using CMake.

The test-compile phase is intended to compile your unit tests.

The compile phase is intended to compile your main code.

The test phase runs your whole test suite. If any fail, 'rl.py' will stop.

The package phase can be used to create a package of your product - perhaps a
.zip or .tar.gz file; perhaps a .deb package for installation on a Debian-based
OS; perhaps a .pkg installer for macOS, or a .msi installer for Windows.

You could add platform triples to the package declarations to create all the
above types of package.

The integration-test phase runs your integration tests, whatever that means
for your project.

Finally the deploy phase is intended to take the packages created in the
package phase, and copy them / upload them somewhere. Github releases? A
Debian repository? Up to you.

### Unless commands
After each phase in your configuration file, you can add an 'unless' clause.
This is of the form:
```
unless: some_command
```
This allows you to skip execution of a phase, based on some test. The test can be
the execution of a command, which returns true or false, or a test for the existence
of some file, using the form '-e some_filename'.

For example, the earlier 'version preparation' case:
```
prepare: echo "version='$VERSION'" > version.py
unless: -e version.py 
```

This will create 'version.py' containing the project version number, unless that
file already exists. As the prepare phase runs at the start of every lifecycle,
you may only want to do it once.

### The release phase
If you produce a release of your software on every commit, this lifecycle phase
is probably not for you. If you prefer referring to versions of your software
with a git SHA, then this is almost certainly not for you.

Version numbers can be used to reflect meaningful upgrades and progress of the
software. 1.0.3 comes after 1.0.2. 1.3.1 is just a small change from 1.3.0.
2.0.0 is a rewrite, watch out! 'rl.py' only supports major.minor.patch versions.

The Maven-inspired version numbering system allows you to declare the product
version as x.y.z, with an optional -SNAPSHOT suffix. While you are working on
your product, you would set the version to say 1.2.3-SNAPSHOT. This is like
an alpha test version. At some point, your work is done, and you want to produce
a release. It's time for 1.2.3 to go out to users.

You could edit the 'build.conf', remove the '-SNAPSHOT' from the version, then
run 'rl.py deep-clean deploy', and if all goes well, your version is then
published. Then you could tag your git repo appropriately, and update the
version to say '1.2.4-SNAPSHOT' and continue...

The release phase automates all this. It will:
* Check for uncommited changes and stop if there are any.
* Clean out a 'release' subdirectory, and clone the repo there.
* Obtain the version from the config, using 0.1.0-SNAPSHOT if you haven't specified one.
* Remove the -SNAPSHOT from it.
* Prompt you with this suggested version to release, giving you the option to change it. (Say you think this is a major change, and requires a 'bigger' number.)
* Sets this version in the configuration file.
* Executes the lifecycle phases, upto and including the integration-test phase.
* If successful, executes the deploy phase. (there's a bug in this ATM)
* If successful, commits the updated version number in the configuration file.
* Creates a git tag containing the updated version number.
* In the main tree, increments the version number to the next -SNAPSHOT, incrementing the patch part of the version.
* Pushes the changes and release tag to the origin.

# AI Declaration
This project was initiated with a prompt to claude.ai, which you will find
in 'initialprompt.txt'.

I (Matt Gumbley) have reviewed this initial code, and refined it as
necessary to achieve the goals listed in initialprompt.txt.

I gratefully acknowledge the work of others that was stolen without consent in the
creation of Large Language Models. I understand the environmental impact of this
technology.

The name was chosen after several iterations of suitable names from the 
`https://namegenhub.com/generator/software-name-generator/` name generator,
with me checking that it wasn't used by anything else.


# License
This code is licensed under the GPL v3 license.
See LICENSE.txt.
