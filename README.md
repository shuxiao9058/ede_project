## A simple project create with Ede


3 Quick Start
*************

Once you have EDE enabled, you can create a project.  This chapter
provides an example C++ project that will create Automake files for
compilation.

3.1 Step 1: Create root directory
=================================

First, lets create a directory for our project.  For this example,
we'll start with something in `/tmp`.

     C-x C-f /tmp/myproject/README RET
     M-x make-directory RET RET

   Now put some plain text in your README file to start.

   Now, lets create the project:

     M-x ede-new RET Automake RET myproject RET

   Nothing visible happened, but if you use `dired` to look at the
directory, you should see this:

       /tmp/myproject:
       total used in directory 32 available 166643476
       drwxr-xr-x  2 zappo users  4096 2012-02-23 22:10 .
       drwxrwxrwt 73 root  root  20480 2012-02-23 22:10 ..
       -rw-r--r--  1 zappo users   195 2012-02-23 22:10 Project.ede
       -rw-r--r--  1 zappo users    10 2012-02-23 22:09 README

3.2 Step 2: Create Subdirectories and Files
===========================================

We'll make a more complex project, so use dired to create some more
directories using the `+` key, and typing in new directories:

     + include RET
     + src RET

   Now I'll short-cut in this tutorial.  Create the following files:

   'include/myproj.hh'
     
     /** myproj.hh ---
      */

     #ifndef myproj_hh
     #define myproj_hh 1

     #define IMPORTANT_MACRO 1

     int my_lib_function();

     #endif // myproj_hh

   'src/main.cpp'
    
     /** main.cpp ---
      */

     #include <iostream>
     #include "myproj.hh"

     int main() {

     }

     #ifdef IMPORTANT_MACRO
     int my_fcn() {

     }
     #endif

   'src/mylib.cpp'   
    
     /** mylib.cpp ---
      *
      * Shared Library to build
      */

     int my_lib_function() {

     }     

3.3 Step 3: Create subprojects
==============================

EDE needs subdirectories to also have projects in them.  You can now
create those projects.

   With `main.cpp` as your current buffer, type:

     M-x ede-new RET Automake RET src RET

   and in `myproj.hh` as your current buffer, type:

     M-x ede-new RET Automake RET include RET

   These steps effectively only create the Project.ede file in which you
will start adding targets.

3.4 Step 4: Create targets
==========================

In order to build a program, you must have targets in your EDE
Projects.  You can create targets either from a buffer, or from a
`dired` directory buffer.

   Note: If for some reason a directory list buffer, or file does not
have the `Project` menu item, or if EDE keybindings don't work, just
use `M-x revert-buffer RET` to force a refresh.  Sometimes creating a
new project doesn't restart buffers correctly.

   Lets start with the header file.  In `include/myproj.hh`, you could
use the menu, but we will now start using the EDE command prefix which
is `C-c .`.

     C-c . t includes RET` miscellaneous RET y

   This creates a misc target for holding your includes, and then adds
myproj.hh to the target.  Automake (the tool) has better ways to do
this, but for this project, it is sufficient.

   Next, visit the `src` directory using dired.  There should be a
`Project` menu.   You can create a new target with

     . t myprogram RET program RET

   Note that `. t` is a command for creating a target.  This command is
also in the menu.  This will create a target that will build a program.
If you want, visit `Project.ede` to see the structure built so far.

   Next, place the cursor on `main.cpp`, and use `. a` to add that file
to your target.

     . a myprogram RET

   Note that these prompts often have completion, so you can just press
`TAB` to complete the name `myprogram`.

   If you had many files to add to the same target, you could mark them
all in your dired buffer, and add them all at the same time.

   Next, do the same for the library by placing the cursor on
`mylib.cpp`.

     . t mylib RET sharedobject RET
     . a mylib RET

3.5 Step 5: Compile, and fail
=============================

Next, we'll try to compile the project, but we aren't done yet, so it
won't work right away.

   Visit `/tmp/myproject/Project.ede`.  We're starting here because we
don't have any program files in this directory yet.  Now we can use the
compile command:

     C-c . C

   Because this is the very first time, it will create a bunch of files
for you that are required by Automake.  It will then use automake to
build the support infrastructure it needs.  This step is skipped if you
choose just a `Makefile` build system.

   After the Automake init, it runs compile.  You will immediately
discover the error in main.cpp can't find `myproj.hh`.  We need to go
fix this.

3.6 Step 6: Customizing your project
====================================

To fix the failed compile, we need to add `/tmp/myproject/include` to
the include path.

   Visit `main.cpp`.

     M-x customize-project RET

   Select the `[Settings]` subgroup of options.  Under `Variable :`
click `[INS]`.  At this point, you need to be somewhat savvy with
Automake.  Add a variable named `CPPFLAGS`, and set the value to
`../include`.

   You should see something like this:

     Variables :
     [INS] [DEL] Cons-cell:
                 Name: AM_CPPFLAGS
                 Value: -I../include
     [INS]
     Variables to set in this Makefile.

   Click `[Apply]`.  Feel free to visit `Project.ede` to see how it
changed the config file.

   Compile the whole project again with `C-c . C` from `main.cpp`.  It
should now compile.

3.7 Step 7: Shared library dependency
=====================================

Note: Supporting shared libraries for Automake in this way is easy, but
doing so from a project of type Makefile is a bit tricky.  If you are
creating shared libraries too, stick to Automake projects.

   Next, lets add a dependency from `main.cpp` on our shared library.
To do that, update main like this:

     int main() {

       my_lib_function();

     }

   Now compile with:

     C-c . c

   where the lower case `c` compiles just that target.  You should see
an error.

   This time, we need to add a dependency from `main.cpp` on our shared
library.  To do that, we need to customize our target instead of the
project.  This is because variables such as the include path are
treated globally, whereas dependencies for a target are target specific.

     M-x customize-target RET

   On the first page, you will see an Ldlibs-local section.  Add mylib
to it by first clicking `[INS]`, and they adding the library.  It
should look like this:

     Ldlibs-Local :
     [INS] [DEL] Local Library: libmylib.la
     [INS]
     Libraries that are part of this project. [Hide Rest]
     The full path to these libraries should be specified, such as:
     ../lib/libMylib.la  or ../ar/myArchive.a

   You will also see other variables for library related flags and
system libraries if you need them.  Click `[Accept]`, and from
`main.cpp`, again compile the whole project to force all dependent
elements to compile:

     C-c . C

3.8 Step 8: Run your program
============================

You can run your program directly from EDE.

     C-c . R RET RET

   If your program takes command line arguments, you can type them in
when it offers the command line you want to use to run your program.
