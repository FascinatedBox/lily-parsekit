lily-parsekit
=============

This is a toolkit for building scripts that need to understand the syntax of
Lily itself. The base of this toolkit is an engine that can parse Lily code.
This tool consists of a base (`engine.lily`) which is used by 

Right now, this kit contains the following scripts:

* `bindgen.lily`: This script helps create extension libraries. You'll sprinkle
  the source `.c` file with special `/** ... */` comments that this picks up.
  From those, this tool will generate helper macros and the loading table that
  the interpreter needs to be able to load an extension library.

Future tool ideas:

* Documentation: A tool that begins at either a `.c` source or `.lily` project
  root and reads through to grab all the docblocks within. From them, generating
  documentation in `.html` or perhaps some other kind of output.

* Hosting: A tool that will walk through a `.lily` project root and from it
  generating a self-hosted executable.

* Diffing: Providing a tool that takes the source of version A of a library, and
  the pending version B. The tool extracts info from both sources, then runs a
  "diff" between them to see if B should be a minor version bump 
