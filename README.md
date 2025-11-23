# releaselite

'releaselite' or 'rl' (from the rl.py file found in this repository)
is a simple, cross-platform build orchestration/lifecycle management
tool. It has similar goals to Maven's project lifecycle management, but
is cross-language/ecosystem, and does not have a preference for
Java/JVM-based projects. It can be used for projects
in any language, and handles the top-level phases of working with a
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

rl.py is a Python 3 script that requires no external dependencies - it
only uses the Python standard library. You may commit this script to your
project repository as necessary.

(more info to follow)



# AI Declaration
This project was initiated with a prompt to claude.ai, which you will find
in 'initialprompt.txt'.

I (Matt Gumbley) have reviewed this initial code, and refined it as
necessary to achieve the goals listed in initialprompt.txt.

I gratefully acknowledge the work of others that was stolen without consent in the
creation of Large Language Models. I understand the environmental impact of this
technology.

# License
This code is licensed under the GPL v3 license.
See LICENSE.txt.
