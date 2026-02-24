# handling-interrupts
### Steps to Install and Launch MCU8051IDE in Docker on macOS:

1. Install XQuartz:
- Download and install XQuartz from [XQuartz's official page](http://www.xquartz.org/).

2. Configure XQuartz:
- Open XQuartz and go to preferences to allow connections from network clients.

3. Start XQuartz:
- Make sure XQuartz is running.

4. Run the Docker Container:
- Use the following command to launch your Docker container with display forwarding:
   docker run -e DISPLAY=host.docker.internal:0 -it 
               
5. Install MCU8051IDE Inside the Container:
- In the terminal of the Docker container, run:
   apt update
   apt install -y mcu8051ide

6. Launch MCU8051IDE:
- After installation, start the MCU8051IDE by executing:
   mcu8051ide
  
### Using SSH with X11 Forwarding
Alternatively, if you are using SSH to connect to the Docker container, you can enable X11 forwarding:
1. SSH into Docker Container:
- Make sure to do:
   docker exec -it  /bin/bash

2. Run with X11 Forwarding:
- When you SSH, add the `-X` option to enable X11 forwarding.


Fixing MCU8051IDE 

Gemini
Fixing MCU8051IDE Tclsh Dependency
Conversation with Gemini
You said
ls /usr/lib/tcltk/x86_64-linux-gnu/Img1.4.7 shows no files.

ls: cannot access '/usr/lib/tcltk/x86_64-linux-gnu/Img1.4.7': No such file or directory

Gemini said
That explains it! If that directory doesn't exist, Tcl has absolutely no way to load the PNG library, even if apt says it's "installed." This often happens in Docker if a previous install was interrupted or if the package paths differ on your specific build.

Let's find where the files actually landed and fix the path.

1. Find where libtk-img put the files
Run this command to see the list of files the package actually installed:

Bash
dpkg -L libtk-img | grep Img
Look for a directory that contains a file named pkgIndex.tcl.

2. Common Alternative Paths
On some Ubuntu 18.04 configurations, the path might be slightly different. Try checking these:

/usr/lib/tcltk/img1.4.7

/usr/lib/tcltk/tkimg1.4.7

/usr/lib/x86_64-linux-gnu/tcltk/Img1.4.7

Run this "catch-all" search if you're unsure:

Bash
find /usr/lib -name "pkgIndex.tcl" -exec grep -l "package ifneeded Img" {} +
3. The "Reset" Fix
If the find command returns nothing, the package installation is corrupted. Since you are in Docker, try a clean reinstall of the image library:

Bash
apt-get purge -y libtk-img
apt-get autoremove -y
apt-get update
apt-get install -y libtk-img
4. Apply the Path
Once you find the real path (let's say it's /usr/lib/tcltk/Img1.4.7), run the IDE with that specific folder:

Bash
export TCLLIBPATH="/usr/lib/tcltk/Img1.4.7 $TCLLIBPATH"
/usr/bin/mcu8051ide
One Final Check: The DISPLAY Variable
Since you've moved past the "tclsh not found" error and are now hitting library errors, the IDE is actually trying to initialize. In Docker, if you haven't set up a virtual display, tk (the graphics engine) sometimes throws generic "library" errors because it can't initialize the screen.

Do you have an X11 server running and have you set your DISPLAY variable? (e.g., export DISPLAY=host.docker.internal:0).

What was the output of the dpkg -L libtk-img | grep Img command? Knowing that path will let us solve this once and for all.

You said
the real path is here:

# find /usr/lib -name "pkgIndex.tcl" -exec grep -l "package ifneeded Img" {} +

/usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7/pkgIndex.tcl

Gemini said
Aha! The aarch64 in the path tells me you are running Ubuntu on an ARM64 architecture (like an Apple Silicon Mac, a Raspberry Pi, or an ARM-based cloud server). Most standard guides assume x86_64, which is why the previous paths didn't work.

Now that we have the exact location, we can bridge the gap.

1. The Immediate Fix (Environment Variable)
Run this command to tell Tcl exactly where that Img folder is. Note that we point to the parent directory of the library folder:

Bash
export TCLLIBPATH="/usr/lib/tcltk/aarch64-linux-gnu $TCLLIBPATH"
/usr/bin/mcu8051ide
2. The Permanent Fix (Symlink)
To avoid having to export that variable every time you open a new terminal, create a symbolic link in the standard Tcl directory:

Bash
ln -s /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7 /usr/lib/tcltk/Img1.4.7
3. Verify Other Dependencies
Since you are on an ARM architecture, let's make sure the other "Missing" items from your initial check (BWidget and Itcl) are also linked properly. Run the check one more time:

Bash
/usr/bin/mcu8051ide --check-libraries
If BWidget or Itcl still say "NO," repeat the search for them:

Bash
find /usr/share/tcltk -name "pkgIndex.tcl" | xargs grep -E "BWidget|Itcl"
Potential Next Hurdle: The Display
Now that Tcl can find the PNG library, the IDE will try to draw its window. Since you are in a Docker container, you will likely hit a no display name and no $DISPLAY environment variable error next.

To fix this, you'll need an X11 server on your host machine. * Windows: Use VcXsrv or Xming.

Mac: Use XQuartz.

Linux: You're likely already running one.

Does the --check-libraries command show all "YES" now? If so, try running the IDE and let me know if you get a "no display" error.

You said
I tried the second option....

# ln -s /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7 /usr/lib/tcltk/Img1.4.7

# usr/bin/mcu8051ide

IMPORTANT INFORMATION :

        This program is distributed in the hope that it will be useful, but with ABSOLUTELY NO WARRANTY !

        This is free software, so you are free to change and redistribute it.

        License: GNU General Public License version 2 or later

        Please report bugs at http://mcu8051ide.sf.net

Authors:

        Martin Osmera <martin.osmera@moravia-microsystems.com>





ERROR: Unable to load library img::png



Tip: try to run mcu8051ide with --check-libraries to see what's wrong.

# find /usr/share/tcltk -name "pkgIndex.tcl" | xargs grep -E "BWidget|Itcl"^C

# 

# 

# /usr/bin/mcu8051ide --check-libraries

MCU 8051 IDE v1.4.9



        Checking libraries...

                1/9 Checking for library BWidget

                        Library present ... NO !

                        Version 1.8     ... NO !

                2/9 Checking for library Itcl

                        Library present ... YES

                        Version 3.4     ... YES

                3/9 Checking for library md5

                        Library present ... YES

                        Version 2.0     ... YES

                4/9 Checking for library Tk

                        Library present ... NO !

                        Version 8.5     ... NO !

                5/9 Checking for library img::png

                        Library present ... NO !

                        Version 1.3     ... NO !

                6/9 Checking for library tdom

                        Library present ... YES

                        Version 0.8     ... YES

                9/9 Checking for library Tcl

                        Library present ... YES

                        Version 8.2     ... YES



        RESULTS:

                Number of fails: 3

                PROGRAM WILL NOT RUN, please install the missing libraries



# find /usr/share/tcltk -name "pkgIndex.tcl" | xargs grep -E "BWidget|Itcl"

/usr/share/tcltk/bwidget1.9.12/pkgIndex.tcl:package ifneeded BWidget 1.9.12 "\

/usr/share/tcltk/bwidget1.9.12/pkgIndex.tcl:    [list tclPkgSetup $dir BWidget 1.9.12 {

# 

Gemini said
We’re making progress—Itcl is now marked as YES. The remaining "NO" for Tk, BWidget, and img::png is happening because Tcl 8.6 expects its libraries to be in specific places, and the ARM-based paths are confusing it.

Since the symlink didn't immediately work, we are going to use the "Global Path" method. This is the most reliable way to force Tcl to look exactly where we know the files are.

1. Identify the missing paths
We need to gather the parent directories for the three missing pieces:

BWidget: /usr/share/tcltk (Based on your find result).

img::png: /usr/lib/tcltk/aarch64-linux-gnu (Based on your find result).

Tk: This is usually in /usr/lib/tcltk/aarch64-linux-gnu/tk8.6.

2. The "Nuclear" Export Command
Run this exact command. It combines all the paths Tcl needs into one environment variable.

Bash
export TCLLIBPATH="/usr/share/tcltk /usr/lib/tcltk/aarch64-linux-gnu /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7 $TCLLIBPATH"
Now, try the check again:

Bash
/usr/bin/mcu8051ide --check-libraries
3. If BWidget or Tk still fail
If BWidget still shows "NO", it might be because the IDE specifically wants an older version or a specific path. Let's try to link it directly where Tcl 8.6 looks:

Bash
# Link BWidget
ln -s /usr/share/tcltk/bwidget1.9.12 /usr/lib/tcltk/bwidget

# Link Tk (since the check says Version 8.5 NO, but you have 8.6)
ln -s /usr/lib/tcltk/aarch64-linux-gnu/tk8.6 /usr/lib/tcltk/tk8.6
4. Important: The "Tk" Version Mismatch
Your check says: Version 8.5 ... NO !. This is because you changed the script to use tclsh8.6, but the IDE script might still be specifically asking for package require Tk 8.5.

Tcl 8.6 is backwards compatible, but we need to make sure the tk8.6 package is actually installed. Run this to be sure:

Bash
apt-get install -y tk8.6
Why is img::png still failing?
If the TCLLIBPATH export above doesn't fix img::png, it's almost certainly because a system library (like libpng) is missing for the ARM architecture in your container.

Run this to check for broken dependencies in the PNG library:

Bash
ldd /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7/libtkimgpng1.4.7.so
If any lines say "not found", please paste them here.

Would you like me to show you how to edit the IDE's main.tcl file to print out exactly where it is looking for these files?

You said
# ln -s /usr/share/tcltk/bwidget1.9.12 /usr/lib/tcltk/bwidget

# ln -s /usr/lib/tcltk/aarch64-linux-gnu/tk8.6 /usr/lib/tcltk/tk8.6

# /usr/bin/mcu8051ide --check-libraries

MCU 8051 IDE v1.4.9



        Checking libraries...

                1/9 Checking for library BWidget

                        Library present ... NO !

                        Version 1.8     ... NO !

                2/9 Checking for library Itcl

                        Library present ... YES

                        Version 3.4     ... YES

                3/9 Checking for library md5

                        Library present ... YES

                        Version 2.0     ... YES

                4/9 Checking for library Tk

                        Library present ... NO !

                        Version 8.5     ... NO !

                5/9 Checking for library img::png

                        Library present ... NO !

                        Version 1.3     ... NO !

                6/9 Checking for library tdom

                        Library present ... YES

                        Version 0.8     ... YES

                9/9 Checking for library Tcl

                        Library present ... YES

                        Version 8.2     ... YES



        RESULTS:

                Number of fails: 3

                PROGRAM WILL NOT RUN, please install the missing libraries



# apt-get install -y tk8.6

Reading package lists... Done

Building dependency tree       

Reading state information... Done

tk8.6 is already the newest version (8.6.8-4).

tk8.6 set to manually installed.

The following package was automatically installed and is no longer required:

  tcl-signal

Use 'apt autoremove' to remove it.

0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.

# ldd /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7/libtkimgpng1.4.7.so

        linux-vdso.so.1 (0x0000ffff87246000)

        libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000ffff870a2000)

        /lib/ld-linux-aarch64.so.1 (0x0000ffff87210000)

# 

Gemini said
The ldd output looks healthy (no "not found" errors), which confirms the library is physically fine. The issue is that the Tcl interpreter is ignoring these specific ARM-based directories when it boots up.

Since the symlinks didn't do the trick, we need to be more aggressive by modifying the auto_path globally within the script environment.

1. The "Path Injector" Method
Instead of relying on symlinks, we will modify the IDE's execution script to force-inject these paths into Tcl's search list.

Open /usr/bin/mcu8051ide and modify it to look exactly like this:

Bash
#!/bin/sh
# Add the ARM library paths to the TCLLIBPATH environment variable
export TCLLIBPATH="/usr/lib/tcltk/aarch64-linux-gnu /usr/share/tcltk /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7 $TCLLIBPATH"

exec tclsh8.6 /usr/share/mcu8051ide/lib//main.tcl "$@" || \
    printf "Unable to execute, TCL interpreter is not reachable.\n" > /dev/stderr
2. Verify "BWidget" and "Tk" Specifically
If the check still fails after Step 1, it’s likely because the IDE is checking for a version it thinks is "too new" or it's looking for a specific folder name. Let's create these specific "version-less" aliases:

Bash
# Force BWidget to be findable in a standard path
ln -sf /usr/share/tcltk/bwidget1.9.12 /usr/share/tcltk/bwidget

# Force the Img package to be findable where Tcl usually expects it
ln -sf /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7 /usr/lib/tcltk/Img
3. The "Package Index" Verification
Tcl only recognizes a folder if it contains a pkgIndex.tcl file. Let's make sure Tcl can actually read the one for BWidget:

Bash
head -n 3 /usr/share/tcltk/bwidget1.9.12/pkgIndex.tcl
Note: If this file says package ifneeded BWidget 1.9.12, but the IDE is hardcoded to require BWidget 1.8, it might fail the check even if the library is present.

4. Final Workaround: The "No Check" Launch
Sometimes the --check-libraries routine in this specific IDE is bugnier than the IDE itself. If you have exported the paths in Step 1, try bypassing the check and launching the IDE directly:

Bash
# Make sure you're using the absolute path with the export
/usr/bin/mcu8051ide
What if it still says "ERROR: Unable to load library img::png"?
If it still fails, it means tclsh8.6 is looking for the "Img" package but doesn't realize it provides img::png. Try this command to see what Tcl thinks is available:

Bash
echo "puts [package names]" | tclsh8.6
Does "Img" appear in that list when you run that command? If it does, we just need to change one word in the IDE source code.

You said
Here the output:

Content of usr/bin/mcu8051ide

#!/bin/sh

# Add the ARM library paths to the TCLLIBPATH environment variable

export TCLLIBPATH="/usr/lib/tcltk/aarch64-linux-gnu /usr/share/tcltk /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7 $TCLLIBPATH"

exec tclsh8.6 /usr/share/mcu8051ide/lib//main.tcl "$@" || \n printf "Unable to execute, TCL interpreter is not reachable.

" > /dev/stderr



# ln -sf /usr/share/tcltk/bwidget1.9.12 /usr/share/tcltk/bwidget

# ln -sf /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7 /usr/lib/tcltk/Img

# usr/bin/mcu8051ide

IMPORTANT INFORMATION :

        This program is distributed in the hope that it will be useful, but with ABSOLUTELY NO WARRANTY !

        This is free software, so you are free to change and redistribute it.

        License: GNU General Public License version 2 or later

        Please report bugs at http://mcu8051ide.sf.net

Authors:

        Martin Osmera <martin.osmera@moravia-microsystems.com>





ERROR: Unable to load library img::png



Tip: try to run mcu8051ide with --check-libraries to see what's wrong.

# usr/bin/mcu8051ide --check-libraries

MCU 8051 IDE v1.4.9



        Checking libraries...

                1/9 Checking for library BWidget

                        Library present ... NO !

                        Version 1.8     ... NO !

                2/9 Checking for library Itcl

                        Library present ... YES

                        Version 3.4     ... YES

                3/9 Checking for library md5

                        Library present ... YES

                        Version 2.0     ... YES

                4/9 Checking for library Tk

                        Library present ... NO !

                        Version 8.5     ... NO !

                5/9 Checking for library img::png

                        Library present ... NO !

                        Version 1.3     ... NO !

                6/9 Checking for library tdom

                        Library present ... YES

                        Version 0.8     ... YES

                9/9 Checking for library Tcl

                        Library present ... YES

                        Version 8.2     ... YES



        RESULTS:

                Number of fails: 3

                PROGRAM WILL NOT RUN, please install the missing libraries



# head -n 3 /usr/share/tcltk/bwidget1.9.12/pkgIndex.tcl

if {[catch {package require Tcl}]} return

# NOTE: auto_loaded top-level commands shall not be qualified (no leading ::)

# but all others should.  See auto_qualify for details.

# head -n 3 /usr/share/tcltk/bwidget1.9.12/pkgIndex.tcl^C

# ^C

# ^C

# 

# 

# echo "puts [package names]" | tclsh8.6

zlib TclOO tcl::tommath Tcl

# 

Gemini said
This is the "smoking gun." The command echo "puts [package names]" | tclsh8.6 output only shows the bare-bones internal packages: zlib TclOO tcl::tommath Tcl.

This means tclsh8.6 is not loading any external libraries at all. Even though you have them on disk, the interpreter isn't looking in the right folders during its initialization.

We need to fix the auto_path directly in the IDE's source code, as the environment variable approach is being ignored by your specific Tcl build.

1. Force the paths inside the IDE
Since tclsh8.6 is stubborn, we will manually tell the IDE where to find its components by editing the main entry file.

Open /usr/share/mcu8051ide/lib/main.tcl and add these lines at the very top (right after the first few comments):

Tcl
lappend auto_path "/usr/share/tcltk"
lappend auto_path "/usr/lib/tcltk"
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu"
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7"
2. Verify with the "Names" test again
Let's see if we can get tclsh8.6 to recognize the packages manually. Run this:

Bash
tclsh8.6 << 'EOF'
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu"
lappend auto_path "/usr/share/tcltk"
puts [package names]
EOF
If Img, Tk, or BWidget appear in that output, the manual lappend in Step 1 will work.

3. Missing Tk Library
The fact that Tk doesn't even appear in your package names list is a major red flag. On Ubuntu, tclsh and wish (the windowing shell) are separate.

Try running the check using wish instead of tclsh in your /usr/bin/mcu8051ide file:

Change: exec tclsh8.6 ...

To: exec wish8.6 /usr/share/mcu8051ide/lib//main.tcl "$@"

4. Why BWidget is failing version check
Your pkgIndex.tcl for BWidget shows version 1.9.12, but the IDE check expects 1.8. Tcl usually handles this, but if the IDE uses a strict package require BWidget 1.8, it might reject 1.9.

If you get past the PNG error and still hit a BWidget error, we will simply edit the IDE's version requirements.

Next Step
Apply the lappend lines to /usr/share/mcu8051ide/lib/main.tcl.

Try running /usr/bin/mcu8051ide again.

If you still get the img::png error, run the tclsh8.6 << 'EOF' block I wrote in Step 2 and show me the output—it will tell us exactly which folder Tcl is willing to accept.

You said
This main.tcl...

Gemini said
It is now clear why your previous attempts failed: the main.tcl file has a hardcoded line that forces Tcl to look for the PNG library in an x86_64 (Intel) directory, which doesn't exist on your aarch64 (ARM) system.

Even when you set environment variables, this script overrules them during startup. To fix this, you need to modify main.tcl to point to the correct ARM path you discovered earlier.

1. Modify main.tcl
Open /usr/share/mcu8051ide/lib/main.tcl and locate line 161. You will see this specific line:

Tcl
lappend auto_path "/usr/lib/tcltk/x86_64-linux-gnu/Img1.4.7"
Change it to your actual ARM path:

Tcl
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7"
2. Fix the BWidget and Tk Paths
Since your package names test showed that Tcl is not loading any external directories, you should add the global library paths to the very top of main.tcl (right after the file inclusion guard at line 46) to ensure BWidget and Tk are found:

Tcl
# Add these right after "set _MAIN_TCL _"
lappend auto_path "/usr/share/tcltk"
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu"
3. Update the execution script
Ensure your /usr/bin/mcu8051ide is using wish8.6 instead of tclsh8.6. wish is the interpreter specifically designed for windowed (GUI) applications, which will solve the "Tk NO" error:

Bash
#!/bin/sh
exec wish8.6 /usr/share/mcu8051ide/lib/main.tcl "$@"
4. Verify the Architecture
The main.tcl file contains a specific architecture check (around line 125) that only expects x86 or x86_64. While this usually triggers for Itcl failures, it confirms the software was not originally designed for ARM. By manually setting the auto_path as shown in Step 1, you are bypassing these hardcoded limitations.

Summary of Changes:
File: /usr/share/mcu8051ide/lib/main.tcl

Changed x86_64-linux-gnu to aarch64-linux-gnu.

Added lappend auto_path for the general library folders.

File: /usr/bin/mcu8051ide

Switched to wish8.6 to provide the Tk environment.

Try running /usr/bin/mcu8051ide now. It should finally bypass the library errors and attempt to open the graphical interface.

You said
Here is the results :



 /usr/bin/mcu8051ide

/bin/sh: 40: /usr/bin/mcu8051ide: Permission denied

# chmod +x /usr/bin/mcu8051ide

# /usr/bin/mcu8051ide

application-specific initialization failed: no display name and no $DISPLAY environment variable

IMPORTANT INFORMATION :

        This program is distributed in the hope that it will be useful, but with ABSOLUTELY NO WARRANTY !

        This is free software, so you are free to change and redistribute it.

        License: GNU General Public License version 2 or later

        Please report bugs at http://mcu8051ide.sf.net

Authors:

        Martin Osmera <martin.osmera@moravia-microsystems.com>





ERROR: Unable to load library img::png



Tip: try to run mcu8051ide with --check-libraries to see what's wrong.

# /usr/bin/mcu8051ide --check-libraries

application-specific initialization failed: no display name and no $DISPLAY environment variable

MCU 8051 IDE v1.4.9



        Checking libraries...

                1/9 Checking for library BWidget

                        Library present ... NO !

                        Version 1.8     ... NO !

                2/9 Checking for library Itcl

                        Library present ... YES

                        Version 3.4     ... YES

                3/9 Checking for library md5

                        Library present ... YES

                        Version 2.0     ... YES

                4/9 Checking for library Tk

                        Library present ... NO !

                        Version 8.5     ... NO !

                5/9 Checking for library img::png

                        Library present ... NO !

                        Version 1.3     ... NO !

                6/9 Checking for library tdom

                        Library present ... YES

                        Version 0.8     ... YES

                9/9 Checking for library Tcl

                        Library present ... YES

                        Version 8.2     ... YES



        RESULTS:

                Number of fails: 3

                PROGRAM WILL NOT RUN, please install the missing libraries



# 

Gemini said
It appears you have reached a critical point where the libraries are still not being detected because the graphical environment is missing. The error message "no display name and no $DISPLAY environment variable" indicates that the Tk library cannot initialize because it has no screen to draw on. In many Tcl/Tk applications, if Tk fails to initialize, the subsequent library checks (like img::png and BWidget) will also fail by default.

To resolve this, you must set up an X11 display for your Docker container and force the library paths one last time.

1. Fix the "No Display" Error
Since you are in a Docker container, you need to link it to an X11 server on your host machine.

On your Host Machine (Mac/Windows/Linux): Ensure an X11 server (like XQuartz or VcXsrv) is running and configured to "Allow connections from network clients."

In your Docker Terminal: Run the following to point the container to your host's display:

Bash
export DISPLAY=host.docker.internal:0
# OR, if on Linux host:
export DISPLAY=:0
2. Final Path Correction in main.tcl
The provided main.tcl file shows a hardcoded search for the PNG library in an x86_64 directory:
lappend auto_path "/usr/lib/tcltk/x86_64-linux-gnu/Img1.4.7".

Because your system is ARM-based (aarch64), you must change this line in /usr/share/mcu8051ide/lib/main.tcl to the correct path you found earlier:

Change line 161 to:
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7"

Additionally, to ensure BWidget and Tk are found, add these lines at the very top of the file, immediately after set _MAIN_TCL _ (around line 47):

Tcl
lappend auto_path "/usr/share/tcltk"
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu"
3. Update the Execution Wrapper
Ensure your /usr/bin/mcu8051ide file is explicitly using wish8.6 to provide the necessary windowing environment:

Bash
#!/bin/sh
export TCLLIBPATH="/usr/lib/tcltk/aarch64-linux-gnu /usr/share/tcltk $TCLLIBPATH"
exec wish8.6 /usr/share/mcu8051ide/lib/main.tcl "$@"
4. Bypassing Version Restrictions
The main.tcl file defines strict library versions:

BWidget 1.8

Tk 8.5

img::png 1.3

Since you have BWidget 1.9 and Tk 8.6, the IDE's internal check may fail even if they are present. Once you have fixed the $DISPLAY and the auto_path above, try running the IDE directly. If it still complains about versions, we can edit the LIBRARIES_TO_LOAD list in main.tcl to match your installed versions.

Try setting the DISPLAY variable and updating the paths in main.tcl now. Does the IDE attempt to open a window?

You said
this is the content of /usr/bin/mcu8051ide:

#!/bin/sh

# Add the ARM library paths to the TCLLIBPATH environment variable

export TCLLIBPATH="/usr/lib/tcltk/aarch64-linux-gnu /usr/share/tcltk /usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7 $TCLLIBPATH"

# exec tclsh8.6 /usr/share/mcu8051ide/lib//main.tcl "$@" 

exec wish8.6 /usr/share/mcu8051ide/lib/main.tcl "$@" || \n printf "Unable to execute, TCL interpreter is not reachable.

" > /dev/stderr



Here is the content of main.tcl:



#!/bin/sh

# the next line restarts using wish \

exec tclsh "$0" "$@"



# Part of MCU 8051 IDE ( http://http://www.moravia-microsystems.com/mcu8051ide )



############################################################################

#    Copyright (C) 2007, 2008, 2009, 2010, 2011, 2012 by Martin Ošmera     #

#    martin.osmera@gmail.com                                               #

#                                                                          #

#    Copyright (C) 2014 by Moravia Microsystems, s.r.o.                    #

#    martin.osmera@moravia-microsystems.com                                #

#                                                                          #

#    This program is free software; you can redistribute it and#or modify  #

#    it under the terms of the GNU General Public License as published by  #

#    the Free Software Foundation; either version 2 of the License, or     #

#    (at your option) any later version.                                   #

#                                                                          #

#    This program is distributed in the hope that it will be useful,       #

#    but WITHOUT ANY WARRANTY; without even the implied warranty of        #

#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the         #

#    GNU General Public License for more details.                          #

#                                                                          #

#    You should have received a copy of the GNU General Public License     #

#    along with this program; if not, write to the                         #

#    Free Software Foundation, Inc.,                                       #

#    59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.             #

############################################################################



# >>> File inclusion guard

if { ! [ info exists _MAIN_TCL ] } {

set _MAIN_TCL _

# Add these right after "set _MAIN_TCL _"

lappend auto_path "/usr/share/tcltk"

lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu"

# <<< File inclusion guard





# --------------------------------------------------------------------------

# DESCRIPTION

# Initilizator of the program

# --------------------------------------------------------------------------



# GENERAL CONSTANTS

# -----------------------------

set DEBUG 0 ;# Turn on debugging

encoding system {utf-8} ;# System encoding

set LIB_DIRNAME [file normalize [file dirname $argv0]] ;# Path to directory where the *.tcl file are located

set INSTALLATION_DIR [file normalize "$LIB_DIRNAME/.."] ;# Path to the main executable (this file)

set ROOT_DIRNAME [file normalize "$LIB_DIRNAME/.."] ;# On Linux it's the same as INSTALLATION_DIR but it's different on Windows

set VERSION "1.4.9" ;# Version of this IDE

set SHORTNAME "MCU8051IDE" ;# Program short name (without white space)

set APPNAME "MCU 8051 IDE v$VERSION" ;# Full program name

set MIN_TCL_VER "8.5" ;# Minimum required Tcl version

set APPLICATION_LOADED 0 ;# True if program is loaded

set TRANSLATION_LOADED 0 ;# Bool: Translation loaded

set MICROSOFT_WINDOWS 0 ;# Bool: Windows OS running on the host computer

set CONFIG_DIR {} ;# Directory containing configuration files



# Check for correct Tcl version

if {[package vcompare $::tcl_version $::MIN_TCL_VER] < 0} {

puts stderr "ERROR: This program requires Tcl version $::MIN_TCL_VER or higher but you have Tcl $::tcl_version ."

puts stderr "       Please install Tcl $::MIN_TCL_VER in order to run this program"



exit 1

}



# Make sure that there is "ttk::spinbox" even when Tk version is lower than 8.5.9

namespace eval ttk {

proc spinbox {args} {

eval "set result \[::spinbox $args\]"

return $result

}

}



## Tcl packages used by this software

 # Format: {

 # {pkg_name pkg_verison}

 # ...

 # }

set LIBRARIES_TO_LOAD {

{BWidget 1.8}

{Itcl 3.4}

{md5 2.0}

{Tk 8.5}

{img::png 1.3}

{tdom 0.8}

{Tclx 8.0}

{Signal 1.4}

{Tcl 8.2}

}



set OPTIONAL_LIBRARIES {

Signal

Tclx

}



## Bool:

 #     1 == Normal GUI operation

 #     0 == Program GUI suppressed

set ::GUI_AVAILABLE 1



## Bool:

 #     1 == library TclX is available

 #     0 == library TclX is NOT available

 #

 # TclX is used here only to handle signals (e.g. SIGINT), so the IDE can run

 # without it, that's the reason for this variable. If TCLX_AVAILABLE is 0 then

 # we are not able to handle signals, but everything else works normally.

set ::TCLX_AVAILABLE 1



## Bool:

 #     1 == library Signal is available

 #     0 == library Signal is NOT available

set ::SIGNAL_AVAILABLE 1



## Determinate the host OS

set ::MICROSOFT_WINDOWS 0

if {[string first {Windows} ${tcl_platform(os)}] != -1} {

# Note:

#   Microsoft Windows is NOT a POSIX system, and because of that, we need

#   to do some workarounds here in order to make the IDE functional.

set ::MICROSOFT_WINDOWS 1



set LIB_DIRNAME_SPECIFIC_FOR_MS_WINDOWS "<AIPCS:LIB_DIRNAME_SPECIFIC_FOR_MS_WINDOWS>" ;# <-- The auto. inst. pkg. creation script will fill this in

set AUTO_PATH_FOR_MS_WINDOWS "<AIPCS:AUTO_PATH_FOR_MS_WINDOWS>" ;# <-- The auto. inst. pkg. creation script will fill this in

set INSTALLATION_DIR $LIB_DIRNAME

set LIB_DIRNAME $LIB_DIRNAME_SPECIFIC_FOR_MS_WINDOWS

set ROOT_DIRNAME [regsub {\/\w+\/?$} $LIB_DIRNAME {}]



foreach dir $AUTO_PATH_FOR_MS_WINDOWS {

lappend ::auto_path "${::ROOT_DIRNAME}/${dir}"

}

set env(ITCL_LIBRARY) "${::ROOT_DIRNAME}/libraries/itcl"

}



# Set directory containing configuration files according to the host OS

if {!$::MICROSOFT_WINDOWS} {

set CONFIG_DIR [file join ${::env(HOME)} .[string tolower ${::SHORTNAME}]]

} else {

set CONFIG_DIR [file join ${::env(USERPROFILE)} ".[string tolower ${::SHORTNAME}]"]

}



# Handle CLI options

# -----------------------------

proc mc args {return [eval "format $args"]}

source "${::LIB_DIRNAME}/cli.tcl"



# SHOW WARNING MESSAGE

# -----------------------------



if {!$::CLI_OPTION(quiet)} {

if {$::CLI_OPTION(nocolor)} {

puts "IMPORTANT INFORMATION:"

puts "\tThis program is distributed in the hope that it will be useful, but with ABSOLUTELY NO WARRANTY !"

puts "\tThis is free software, so you are free to change and redistribute it."

puts "\tLicense: GNU General Public License version 2 or later"

puts "\tPlease report bugs at http://mcu8051ide.sf.net"

puts "Authors:"

puts "\tMartin Osmera <martin.osmera@moravia-microsystems.com>"

} else {

puts "IMPORTANT INFORMATION :"

puts "\tThis program is distributed in the hope that it will be useful, but with \033\[31;1mABSOLUTELY NO WARRANTY !\033\[m"

puts "\tThis is free software, so you are free to change and redistribute it."

puts "\tLicense: GNU General Public License version 2 or later"

puts "\tPlease report bugs at \033\[34;1mhttp://mcu8051ide.sf.net\033\[m"

puts "Authors:"

puts "\tMartin Osmera \033\[33;1m<martin.osmera@moravia-microsystems.com>\033\[m"

}

}



## This function should be called when some Tcl library fail to load

 # @parm String library - Name of failed library

 # @return void

proc libraryLoadFailed {library} {



# Itcl workarond for Debian

if {$library == {Itcl}} {

if {[package vcompare $::tcl_version "8.6"] >= 0} {

if {![catch {package require Itcl}]} {

return

}

}



set library_version "3.4"

set libname "libitcl"



set ::env(ITCL_LIBRARY) ${::LIB_DIRNAME}



puts stderr "\nERROR: Unable to load Itcl library compatible with this version of Tcl/Tk!"

puts stderr "Trying to workaround ..."



if {[lsearch {Linux} ${::tcl_platform(os)}] == -1} {

puts stderr "FATAL ERROR: Unsupported operating system. ${::tcl_platform(os)}"

puts stderr "You can contact authors of the project at <martin.osmera@gmail.com> if you want to get you OS supported."

exit 1

}



if {[lsearch {x86_64 i386 i486 i586 i686 x86} ${::tcl_platform(machine)}] == -1} {

puts stderr "FATAL ERROR: Unsupported system architecture. ${::tcl_platform(machine)}"

puts stderr "You can contact authors of the project at <martin.osmera@gmail.com> if you want to get you OS supported."

exit 1

}



puts stderr "Loading library $library for ${::tcl_platform(os)} on ${::tcl_platform(machine)} ... (filename: ${libname}${library_version}.so.${::tcl_platform(os)}.${::tcl_platform(machine)})"

if {[catch {load "${::LIB_DIRNAME}/${libname}${library_version}.so.${::tcl_platform(os)}.${::tcl_platform(machine)}" Itcl} error_info]} {

puts stderr "FAILED !"

puts stderr "Reason: ${error_info}"

puts "\nPlease try to run mcu8051ide with --check-libraries to see what's wrong."



exit 1

} else {

puts stderr "WORKAROUND SUCCESSFUL ... \n(But don't be much happy about this, it is still serious failure. And please don't forget to comply to developers of your Linux distribution. Missing library is: ${library} version ${library_version})"

return

}



# Tclx workarond for Debian (1/2)

} elseif {$library == {Tclx}} {

set ::TCLX_AVAILABLE 0

puts stderr "\nERROR: Unable to load library Tclx, MCU 805 1IDE functionality might be limited."

return



# Tclx workarond for Debian (2/2)

} elseif { $library == {Signal} } {

set ::SIGNAL_AVAILABLE 0

if {!$::TCLX_AVAILABLE} {

puts stderr "\nERROR: Unable to load library Signal, MCU 805 1IDE functionality might be limited."

}

return

}



# Print error message

if {$::CLI_OPTION(nocolor)} {

puts stderr "\n\nERROR: Unable to load library $library"

} else {

puts stderr "\n\n\033\[31mERROR:\033\[m Unable to load library \033\[32m$library\033\[m"

}



# Print tip

puts "\nTip: try to run mcu8051ide with --check-libraries to see what's wrong."



# Terminate the program

exit 1

}



# PRE-INITIALIZATION

# -----------------------------

# Load Tk ToolKit

lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7"

#lappend auto_path "/usr/lib/tcltk/x86_64-linux-gnu/Img1.4.7"

set T [lindex [time {

if {[catch {package require img::png 1.3} e]} {

libraryLoadFailed "img::png"

}

if {[catch {package require Tk $::MIN_TCL_VER} errinfo]} {

puts stderr "Unable to initialize Tk\n$errinfo"

}

}] 0]

# Hide main window

wm withdraw .

update



# Determinate default Fixed font

set ::DEFAULT_FIXED_FONT {Courier}

if {!$::MICROSOFT_WINDOWS} {

if {[lsearch -ascii -exact [font families] {DejaVu Sans Mono}] != -1} {

set ::DEFAULT_FIXED_FONT {DejaVu Sans Mono}

}

}



# Set default background color

set ::COMMON_BG_COLOR [. cget -bg]



# ------------------------------------------------------------------------------

# Microsoft Windows OS specific code

# ------------------------------------------------------------------------------

if {$::MICROSOFT_WINDOWS} {

# Print windows related warning

puts ""

puts "        THE IDE WAS ORIGINALY WRITTEN FOR POSIX, SO IT IS POSSIBLE THAT SOME"

puts "        FUNCTIONALITY WILL BE LIMITED ON MS WINDOWS DUE TO ABSENCE OF CERTAIN"

puts "        POSIX FUNCTIONALITY!"

puts ""

}

# ------------------------------------------------------------------------------





# ------------------------------------------------------------------------------

# Make key shortcuts independent on Caps Lock.

# ------------------------------------------------------------------------------

rename bind original_command_bind

proc mcu8051ide_bind args {

set widget [lindex $args 0]

set event_str [lindex $args 1]

set command [lindex $args 2]



if {[llength $args] > 3} {

error "Too many arguments to bind command."

} elseif {[llength $args] == 2} {

return [original_command_bind $widget $event_str]

}



# MS Windows doesn't recognize ISO and XFree86 codes

if {$::MICROSOFT_WINDOWS} {

if {

[string first {ISO} $event_str] != -1 ||

[string first {XF86} $event_str] != -1

} then {

return

}

}



if {

[string index $event_str end-2] == {-}

&&

[string is alpha [string index $event_str end-1]]

&&

[string index $event_str end] == {>}

} then {

set letter [string index $event_str end-1]

set event_str [string range $event_str 1 end-2]

if {[string is upper $letter]} {

set event_str "Shift-${event_str}"

}



original_command_bind $widget "<${event_str}[string toupper $letter]>" $command

original_command_bind $widget "<${event_str}[string tolower $letter]>" $command

} else {

original_command_bind $widget $event_str $command

}

}

rename mcu8051ide_bind bind

# ------------------------------------------------------------------------------



# Load base config file

# -----------------------------



# Initialize variable containing the font size factor

set font_size_factor 1.0



# Load i18n library

# (It must be loaded here because ::msgcat::mclocale must be available when

# base config file is being loaded)

incr T [lindex [time {

if {[catch {package require msgcat 1.3.4}]} {

libraryLoadFailed "msgcat"

} else {

namespace import -force ::msgcat::mc

namespace import -force ::msgcat::mcset

namespace import -force ::msgcat::*

}

}] 0]

# Check if the file exits

if {![file exists ${::CONFIG_DIR}]} {

file mkdir ${::CONFIG_DIR}

puts "\nCreating program configuration files in directory: \"[file normalize ${::CONFIG_DIR}]\""

if {!$::MICROSOFT_WINDOWS} {

puts "Welcome in this IDE, [file tail [file normalize ~]] !"

} else {

catch { ;# Make the configuration directory in Microsoft Windows hidden

file attributes $::CONFIG_DIR -hidden 1

}

puts "Welcome in this IDE, ${::env(USERNAME)} !"

}

}

## Open and read the file

if {[catch {

set conf_file [open "${::CONFIG_DIR}/base.conf" r]

 # File doesn't exits -> create it with default configuration

}]} then {

# Default settings

set detected_lang [string tolower [lindex [split [::msgcat::mclocale] {_}] 0]]

if {{} != [lindex [split [::msgcat::mclocale] {_}] 1]} {

append detected_lang {_} [string toupper [lindex [split [::msgcat::mclocale] {_}] 1]]

}

array set GLOBAL_CONFIG [list \

splash 1 \

tips 1 \

language $detected_lang \

background {Default} \

wstyle {clam} \

]



# Create the file

if {[catch {

set conf_file [open "${::CONFIG_DIR}/base.conf" w]

puts -nonewline $conf_file [list \

$GLOBAL_CONFIG(splash) \

$GLOBAL_CONFIG(tips) \

$GLOBAL_CONFIG(language) \

${::font_size_factor} \

$GLOBAL_CONFIG(background) \

$GLOBAL_CONFIG(wstyle) \

]

close $conf_file

}]} then {

puts stderr "Unable to create base configuration file"

}

 # File exits -> read configuration from it

} else {

# Read file contents

set data [read $conf_file]

close $conf_file



# Set configuration according to the file contents

set GLOBAL_CONFIG(splash) [lindex $data 0]

set GLOBAL_CONFIG(tips) [lindex $data 1]

set GLOBAL_CONFIG(language) [lindex $data 2]

set ::font_size_factor [lindex $data 3]

set GLOBAL_CONFIG(background) [lindex $data 4]

set GLOBAL_CONFIG(wstyle) [lindex $data 5]



## Validate read values

if {![regexp {^1\.\d+$} $::font_size_factor] || $::font_size_factor < 1.0 || $::font_size_factor >= 2.0} {

set ::font_size_factor 1.0

}

if {![string is boolean -strict ${::GLOBAL_CONFIG(splash)}]} {

set GLOBAL_CONFIG(splash) 1

}

if {![string is boolean -strict ${::GLOBAL_CONFIG(tips)}]} {

set GLOBAL_CONFIG(tips) 1

}

# This feature is not supported on MS Windows due to BWidget

if {!$::MICROSOFT_WINDOWS} {

switch -- ${::GLOBAL_CONFIG(background)} {

{Default} {set ::COMMON_BG_COLOR {#eeeeee}}

{Windows} {set ::COMMON_BG_COLOR {#ece9d9}}

{Tk} {}

{Light} {set ::COMMON_BG_COLOR {#f8f8f8}}

{Dark} {set ::COMMON_BG_COLOR {#cccccc}}

default {

set ::GLOBAL_CONFIG(background) {Default}

set ::COMMON_BG_COLOR {#eeeeee}

}

}

}

if {[lsearch -ascii -exact [ttk::style theme names] ${::GLOBAL_CONFIG(wstyle)}] == -1} {

set ::GLOBAL_CONFIG(wstyle) {clam}

}

# Check if the cpecified translation is valid

set tmp [list]

catch { ;# For Microsoft Windows it has to be enclosed by catch

set tmp [glob -nocomplain -types f -tails \

-directory "${::INSTALLATION_DIR}/translations" *.msg \

]

}

set translations {en}

foreach translation $tmp {

lappend translations [file rootname $translation]

}



if {[lsearch $translations ${GLOBAL_CONFIG(language)}] == -1} {

set GLOBAL_CONFIG(language) {en}

}

}





# Load translation

# -----------------------------



# Load list of language names

set ::LANGUAGE_CODES [list]

namespace eval ::configDialogues::global {

if {[catch {

set file [open "${::INSTALLATION_DIR}/translations/languages.txt" r]

set languages [read $file]

close $file



regsub -all -line {\s*#.*$} $languages {} languages

set ::LANGUAGE_CODES $languages

lindex $languages 0

} e]} {

puts stderr "Error while loading list of laguages: $e"

}

}



# Load language specific translation file

if {!${::CLI_OPTION(notranslation)} && ${GLOBAL_CONFIG(language)} != {en}} {

if {[catch {

::msgcat::mclocale ${GLOBAL_CONFIG(language)}

incr T [lindex [time {

source "${::INSTALLATION_DIR}/translations/${GLOBAL_CONFIG(language)}.msg"

}] 0]



} result]} then {

puts stderr "Unable to load translation"

puts stderr "\tFile: '${::INSTALLATION_DIR}/translations/${GLOBAL_CONFIG(language)}.msg'"



} else {

set ::TRANSLATION_LOADED 1

}

}



# CREATE SPLASH SCREEN

# -----------------------------

if {!$::CLI_OPTION(nosplash) && ${::GLOBAL_CONFIG(splash)}} {



# Workaround for multiple monitors

toplevel .splash_aux

wm attributes .splash_aux -fullscreen 1

update

set sw [winfo width .splash_aux]

set sh [winfo height .splash_aux]

destroy .splash_aux

update



# Crete toplevel  window

toplevel .splash -class {Splash creen} -bg ${::COMMON_BG_COLOR}



# Show image of splash creen

place [label .splash.bg \

-bg {#FFFFFF} \

-image [ \

image create photo -format png \

-file "${::ROOT_DIRNAME}/icons/other/splash.png" \

] \

] -x 0 -y 0 -width 400 -height 199



# Show status bar

place [label .splash.status \

-bg {#FFFFFF} -fg {#0000FF} \

-text [mc "Initializing %s" $APPNAME] \

] -x 200 -y 180 -anchor center



# Set window parameters

wm geometry .splash "=400x199+[expr {$sw / 2 - 200}]+[expr {$sh / 2 - 100}]"

wm overrideredirect .splash 1

wm attributes .splash -alpha 0.8



# Click on splash creen destroys it

bind .splash <1> {wm withdraw .splash}



# Done ..

update

}





# BASIC FUNCTIONS

# -----------------------------



## Print content of $T in mili seconds ($T must contain value in [us])

 # @return void

proc time_in_msec {} {

global T ;# Time in microseconds



# Determinate number of miliseconds

set msec [lindex $T 0]

set msec [expr {$msec / 1000}]



# print the message

if {!$::CLI_OPTION(quiet)} {

if {$::CLI_OPTION(nocolor)} {

puts "... $msec ms"

} else {

puts "... \033\[33m$msec ms\033\[m"

}

}

}



## Print some initialization message (splash screen and CLI)

 # @parm String message - text of the message

 # @return void

proc showInitMessage {message} {



# Change content of splash screen status bar

if {!${::CLI_OPTION(nosplash)} && ${::GLOBAL_CONFIG(splash)}} {

if {[winfo exists .splash.status]} {

.splash.status configure -text [string trim $message]

update

}

}



# Print message to console output

if {!${::CLI_OPTION(quiet)}} {

if {${::CLI_OPTION(nocolor)}} {

puts -nonewline $message

} else {

puts -nonewline "\033\[37m$message\033\[m"

}



puts -nonewline [string repeat { } [expr {38 - [string length $message]}]]

flush stdout

}

}



## Set status bar tip for some widget

 # Usage:

 # setStatusTip -widget $some_widget -text "some text"

 #

 # @return void

proc setStatusTip args {



# Local variables

set widgetIsSet 0 ;# True if widget is set

set textIsSet 0 ;# True if text is set

set argsLength [llength $args] ;# Number of arguments

set widget {} ;# ID of widget specified by argument '-widget'

set helpText {} ;# Help text specified by argument '-text'



# Iterate over given arguments and evaluate them

for {set i 0} {$i < $argsLength} {incr i} {

# Currently parsed argument

set arg [lindex $args $i]

# Decide what $arg means

switch -- $arg {

-widget { ;# ID of the widget



# check if that widget wasn't already specified

if {$widgetIsSet} {

error "Widget has been already specified"

}



# Check if widget's ID follow the arument

incr i

if {$i >= $argsLength} {

error "Expected widget name after -widget option"

}



# Set ID of the widget

set widget [lindex $args $i]

if {![winfo exists $widget]} {

error "The specified widget does not exist"

}



# Widget is now set

set widgetIsSet 1

}



-text { ;# The help text



# Check if help text follow the argument

incr i

if {$i >= $argsLength} {

error "Expected text after -text option"

}



# Set the help text

set helpText [lindex $args $i]



# Help text is now set

set textIsSet 1

}



default { ;# Unrecognized opton -> invoke ERROR

error "Invalid argument '$arg', possible options are -widget and -text"

}

}

}



# Ckeck if both aruments are properly specified

if {!$widgetIsSet || !$textIsSet} {

error "You must specify text and widget"

}



# Create binding

bind $widget <Enter> "Sbar -freeze {$helpText}"

bind $widget <Leave> "Sbar {}"

}







# INITIALIZATION

# -----------------------------



# Show "first line message"

if {!$CLI_OPTION(quiet)} {

if {$CLI_OPTION(nocolor)} {

puts [mc "\nInitializing MCU 8051 IDE %s" $VERSION]

} else {

puts [mc "\nInitializing \033\[1mMCU 8051 IDE \033\[32m%s\033\[m" $VERSION]

}

}



## Load libraries

showInitMessage [mc "\tLoading libraries"]

incr T [lindex [time {

# Iterate over list of libraries and lod each of them

foreach library $::LIBRARIES_TO_LOAD {

# Loading successful

if {[catch {package require [lindex $library 0] [lindex $library 1]}]} {

libraryLoadFailed [lindex $library 0]

# Loading failed

} else {

if {!$::CLI_OPTION(nosplash)} update

}

}



if {$::MICROSOFT_WINDOWS} { ;# Load dde - Dynamic Data Exchange on Microsoft Windows

package require dde

}



# Import NS for managing OOP in Tcl

namespace import -force ::itcl::*

}] 0]



# Look for some external programs

foreach program {

urxvt vim emacs kwrite gedit nano dav

le sdcc indent doxygen asl asem doxywizard

as31 sdcc-sdcc gvim hunspell d52

make

} \

{

if {[auto_execok $program] == {}} {

set ::PROGRAM_AVAILABLE($program) 0

} else {

set ::PROGRAM_AVAILABLE($program) 1

}

}

time_in_msec ;# Print time info



## Load program sources

showInitMessage [mc "\tLoading program sources"]

set T [time {

source "${::LIB_DIRNAME}/lib/modern_notebook.tcl" ;# NoteBook widget with a modern look (modern in 2011)

source "${::LIB_DIRNAME}/lib/FSnotifications.tcl" ;# Filesystem watcher

source "${::LIB_DIRNAME}/lib/innerwindow.tcl" ;# Tool for creating inner windows

source "${::LIB_DIRNAME}/dialogues/errorhandler.tcl" ;# Background error handler

source "${::LIB_DIRNAME}/dialogues/my_tk_messageBox.tcl";# A replacement for tk_messageBox

source "${::LIB_DIRNAME}/lib/settings.tcl" ;# Settings management

source "${::LIB_DIRNAME}/project.tcl" ;# Project management

source "${::LIB_DIRNAME}/dialogues/fsd.tcl" ;# File selection dialog

source "${::LIB_DIRNAME}/X.tcl" ;# GUI <==> Implementation Interface

source "${::LIB_DIRNAME}/configdialogues/configdialogues.tcl";# Configuration dialogues

source "${::LIB_DIRNAME}/editor/editor.tcl" ;# Source code editor

source "${::LIB_DIRNAME}/lib/Math.tcl" ;# Special mathematical operations

source "${::LIB_DIRNAME}/compiler/compiler.tcl" ;# 8051 Assemly language compiler

source "${::LIB_DIRNAME}/dialogues/tips.tcl" ;# Tips on start-up

source "${::LIB_DIRNAME}/lib/hexeditor.tcl" ;# Hexadecimal editor

source "${::LIB_DIRNAME}/utilities/hexeditdlg.tcl" ;# Hexadecimal editor dialog

source "${::LIB_DIRNAME}/environment.tcl" ;# Main window "trappings" (menu and such)

source "${::LIB_DIRNAME}/rightpanel/rightpanel.tcl" ;# Right panel

source "${::LIB_DIRNAME}/leftpanel/filelist.tcl" ;# Left and middle panel

source "${::LIB_DIRNAME}/simulator/simulator.tcl" ;# MCU Simulator

source "${::LIB_DIRNAME}/bottompanel/bottomnotebook.tcl";# Bottom panel

source "${::LIB_DIRNAME}/maintab.tcl" ;# Central widget

source "${::LIB_DIRNAME}/lib/ihextools.tcl" ;# Tools for manipulating Intel 8 HEX

source "${::LIB_DIRNAME}/utilities/symbol_viewer.tcl" ;# Assembly symbols viewer

source "${::LIB_DIRNAME}/utilities/eightsegment.tcl" ;# 8-Segment LED display editor

source "${::LIB_DIRNAME}/utilities/asciichart.tcl" ;# ASCII chart

source "${::LIB_DIRNAME}/utilities/table_of_instructions.tcl";# 8051 Instruction Table

source "${::LIB_DIRNAME}/utilities/notes.tcl" ;# Scribble notepad

source "${::LIB_DIRNAME}/utilities/baseconverter.tcl" ;# Base converter

source "${::LIB_DIRNAME}/utilities/speccalc.tcl" ;# Special calculator for x51 MCU's

source "${::LIB_DIRNAME}/utilities/rs232debugger.tcl" ;# UART/RS232 applications debugger

}]

time_in_msec ;# Print time info



# CHECK FOR VALIDITY OF THE MCU DATABASE

if {${::X::available_processors} == {}} {

destroy .splash

bell

tk_messageBox \

-icon error \

-type ok \

-title [mc "FATAL ERROR"] \

-message [mc "MCUs database file is corrupted,\nthis program cannot run without it.\nPlease reinstall MCU 8051 IDE."]

exit 1

}



# Load global configuration

loadApplicationConfiguration



# Start spell checker process and wait until it is started

::Editor::restart_spellchecker_process

::Editor::adjust_spell_checker_config_button



# Initialize GUI environment

mainmenu_redraw ;# Main menu

iconbar_redraw ;# Main toolbar

shortcuts_reevaluate ;# Key shortcuts



## Remove splash screen

if {[winfo exists .splash]} {

destroy .splash

}

if {$CLI_OPTION(minimalized)} {

wm state . iconic

} else {

wm deiconify .

}



# Configure signal handling

if {$::TCLX_AVAILABLE || $::SIGNAL_AVAILABLE} {

proc signal_handler {signal_name} {

global cntrlc_flag

puts stderr [mc "\nExiting on signal %s" $signal_name]

catch {

::X::__exit 1 1

}

puts stderr [mc "\nERROR: Unable to exit the program correctly -- TERMINATING NOW!" $signal_name]

exit 1

}



if {$::TCLX_AVAILABLE} {

signal trap SIGINT {signal_handler SIGINT}

signal trap SIGTERM {signal_handler SIGTERM}

} else {

signal add SIGINT {signal_handler SIGINT}

signal add SIGTERM {signal_handler SIGTERM}

}

}





# ---------------------------------------------------------

# Ugly job ... dirty workarounds and such things ... :(

# ---------------------------------------------------------



catch {

NoteBook .foo

proc NoteBook::_getoption {path page option} {

if {$option == {-background}} {

return ${::COMMON_BG_COLOR}

}



catch {

set value [Widget::cget $path.f$page $option]

if {![string length $value]} {

set value [Widget::cget $path $option]

}



return $value

}

}

destroy .foo

}



# ---------------------------------------------------------



## Open the last session

# Print message

showInitMessage [mc "\tOpening last session"]

flush stdout

# Evaluate new geometry of the main window

update

evaluate_new_window_geometry

if {![winfo viewable .mainMenu]} {

wm geometry . $::CONFIG(WINDOW_GEOMETRY)

update

}

# Open projects of last session

set T [time {

foreach project_file $::CONFIG(OPENED_PROJECTS) {

if {![Project::open_project_file $project_file]} {

tk_messageBox \

-title [mc "File not found"] \

-icon warning \

-type ok \

-message [mc "Unable to open project file:\n\"%s\"" $project_file]

}

}

}]



# Reopen base converters

foreach cfg $::CONFIG(BASE_CONVERTERS) {

if {[catch {

set obj [::X::__base_converter]

::X::$obj set_config $cfg

}]} then {

puts stderr {}

puts stderr $::errorInfo

}

}



time_in_msec ;# Print time info



# Without this "help windows" won't work properly on MS Windows

if {$::MICROSOFT_WINDOWS} {

if {[wm state .] == {zoomed}} {

set ::LATS_KNOWN_WM_STATE_IS_ZOOMED 1

} else {

set ::LATS_KNOWN_WM_STATE_IS_ZOOMED 0

}

}

# Create binding for panes management

bind . <Configure> {

X::redraw_panes



# Without this "help windows" won't work properly on MS Windows

if {$::MICROSOFT_WINDOWS} {

if {[wm state .] == {zoomed}} {

set now_zoomed 1

} else {

set now_zoomed 0

}



if {!${::LATS_KNOWN_WM_STATE_IS_ZOOMED} && $now_zoomed} {

after idle {

update

regsub {[\+\-].*$} [wm geometry .] {+0+0} geometry

wm geometry . $geometry

}

}

set ::LATS_KNOWN_WM_STATE_IS_ZOOMED $now_zoomed

}

}

# Print final message

if {!$CLI_OPTION(quiet)} {

puts [mc "%s is now operational\n" $APPNAME]

}



# Program is now operational

set ::Compiler::in_IDE 1

set APPLICATION_LOADED 1

set X::critical_procedure_in_progress 0

update

X::redraw_panes

update

foreach project ${::X::openedProjects} {

$project bottomNB_redraw_pane

$project filelist_adjust_size_of_tabbar

$project ensure_that_both_editors_are_properly_initialized

}

# Focus on the active editor

if {${::X::actualProject} != {}} {

update

focus [${::X::actualProject} editor_procedure {} cget -editor]

focus [${::X::actualProject} editor_procedure {} Configure {}]

focus [${::X::actualProject} editor_procedure {} highlight_visible_area {}]

}



# Correct strange behavior concerning restoration of the last window size and position

if {$::MICROSOFT_WINDOWS} {

update

wm geometry . $::CONFIG(WINDOW_GEOMETRY)

}



# Initialize file change notifications mechanism

FSnotifications::init



# >>> File inclusion guard

}

# <<< File inclusion guard

Gemini said
We are very close. The main.tcl file you provided confirms that the script is manually adding paths to auto_path and then trying to load the libraries.

The "no display" error is currently the primary blocker. Tcl/Tk libraries (especially Tk and img::png) cannot initialize their internal structures if they can't connect to a graphical server. This causes the library load to fail, which the script reports as "Unable to load library."

Follow these steps to fix the remaining issues:

1. The X11 Display Fix (Crucial)
You must give the Docker container a way to show its windows.

If your host is a Mac/Windows machine: You need an X11 server (XQuartz or VcXsrv) running.

In your terminal: Before running the IDE, set your display variable. Replace YOUR_IP with your host machine's IP address (or host.docker.internal on Mac/Windows):

Bash
export DISPLAY=host.docker.internal:0
2. Update main.tcl (Final Version)
You need to make two specific edits to the main.tcl you just shared to handle the ARM (aarch64) architecture.

Edit 1: Add general ARM paths (Around line 47)
Find where you added the first lappend lines and ensure they are exactly like this:

Tcl
# Add these right after "set _MAIN_TCL _"
lappend auto_path "/usr/share/tcltk"
lappend auto_path "/usr/lib/tcltk"
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu"
Edit 2: Fix the PNG path (Around line 161)
Find the line that mentions x86_64 and change it to aarch64.

Change: lappend auto_path "/usr/lib/tcltk/x86_64-linux-gnu/Img1.4.7"

To: lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7"

3. Adjust Library Versions
The IDE is asking for very specific versions (BWidget 1.8, Tk 8.5, img::png 1.3). Since you have 1.9, 8.6, and 1.4 respectively, Tcl might refuse to load them if the script is too strict.

In main.tcl, locate the set LIBRARIES_TO_LOAD block (around line 85) and change them to empty strings or your actual versions to allow the newer libraries to load:

Tcl
set LIBRARIES_TO_LOAD {
	{BWidget	""}
	{Itcl		""}
	{md5		""}
	{Tk		""}
	{img::png	""}
	{tdom		""}
	{Tclx		""}
	{Signal		""}
	{Tcl		""}
}
4. Verify the Execution Script
Make sure /usr/bin/mcu8051ide is using wish8.6 so it starts with the graphical environment already loaded:

Bash
#!/bin/sh
export DISPLAY=host.docker.internal:0  # Ensure this matches your setup
exec wish8.6 /usr/share/mcu8051ide/lib/main.tcl "$@"
Summary of why it's still failing:
Architecture: The script was trying to find libraries in an Intel folder (x86_64) that doesn't exist on your ARM chip.

Display: Tk crashes immediately if $DISPLAY is missing, which prevents the PNG library from even trying to load.

Path: Tclsh8.6 on ARM doesn't automatically look in /usr/lib/tcltk/aarch64-linux-gnu, so we have to "force-feed" it that path.

Once you set the DISPLAY and fix the path in main.tcl, try running /usr/bin/mcu8051ide again. Do you see a window pop up on your host computer?

You said
Almost there:



# /usr/bin/mcu8051ide --check-libraries

Authorization required, but no authorization protocol specified

Authorization required, but no authorization protocol specified

application-specific initialization failed: couldn't connect to display "host.docker.internal:0"

MCU 8051 IDE v1.4.9



        Checking libraries...

                1/9 Checking for library BWidget

                        Library present ... Authorization required, but no authorization protocol specified

Authorization required, but no authorization protocol specified

NO !

                        Version         ... NO !

                2/9 Checking for library Itcl

                        Library present ... YES

                        Version         ... NO !

                3/9 Checking for library md5

                        Library present ... YES

                        Version         ... NO !

                4/9 Checking for library Tk

                        Library present ... Authorization required, but no authorization protocol specified

Authorization required, but no authorization protocol specified

NO !

                        Version         ... NO !

                5/9 Checking for library img::png

                        Library present ... Authorization required, but no authorization protocol specified

Authorization required, but no authorization protocol specified

NO !

                        Version         ... NO !

                6/9 Checking for library tdom

                        Library present ... YES

                        Version         ... NO !

                9/9 Checking for library Tcl

                        Library present ... YES

                        Version         ... NO !



        RESULTS:

                Number of fails: 7

                PROGRAM WILL NOT RUN, please install the missing libraries

Gemini said
This is the final hurdle. You have two distinct problems appearing in your logs:

X11 Authentication Error: "Authorization required, but no authorization protocol specified." This means Docker is finding your X11 server, but the server is rejecting the connection for security reasons.

Version Mismatch: The "Version ... NO!" errors for the libraries that are present (Itcl, Tcl) happen because the IDE is looking for specific version numbers (like 8.5) but finding your 8.6 installation.

Here is the plan to finish the setup:

1. Fix the X11 Authorization (The "Authorization required" error)
You need to tell your X11 server (XQuartz on Mac or VcXsrv on Windows) to allow the connection from the Docker container.

On Mac (XQuartz):

Open XQuartz Preferences.

Go to the Security tab.

Check "Allow connections from network clients".

Restart XQuartz.

In your Mac terminal (not the Docker one), run: xhost +local:root or xhost +hostname.

On Windows (VcXsrv):

When starting VcXsrv (XLaunch), make sure you check the box "Disable access control".

On Linux:

Run xhost + in your host terminal.

2. Bypass the Version Checks in main.tcl
Since you are using Tcl 8.6, the IDE’s strict version checks are failing even though the libraries are there. You need to relax these requirements in /usr/share/mcu8051ide/lib/main.tcl.

Find the set LIBRARIES_TO_LOAD block (around line 85) and remove the version numbers so it looks like this:

Tcl
set LIBRARIES_TO_LOAD {
    {BWidget    ""}
    {Itcl       ""}
    {md5        ""}
    {Tk         ""}
    {img::png   ""}
    {tdom       ""}
    {Tclx       ""}
    {Signal     ""}
    {Tcl        ""}
}
3. Final Search Path Verification
Ensure you have the lappend lines we discussed at the top of main.tcl (around line 47) so Tcl 8.6 knows to look in the ARM directories:

Tcl
lappend auto_path "/usr/share/tcltk"
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu"
lappend auto_path "/usr/lib/tcltk/aarch64-linux-gnu/Img1.4.7"
4. Run the IDE Directly
The --check-libraries routine is very sensitive to X11 errors. Once you have "Disabled access control" or run xhost + on your host machine, try running the IDE directly without the check:

Bash
/usr/bin/mcu8051ide
If you still get "Authorization required," try running the container with the host's network mode by adding --net=host to your docker run command, or simply use the IP of your host machine instead of host.docker.internal.

You said
~ % xhost +local:root

zsh: command not found: xhost

Gemini said
The zsh: command not found: xhost error means the X11 support tools aren't in your Mac's system path. This usually happens because XQuartz isn't installed or hasn't been initialized correctly.

Follow these steps to fix the command and the authorization issue:

1. Install/Verify XQuartz
The xhost command is part of the XQuartz package. Even if you have it installed, you might need to link it.

If you don't have XQuartz: Install it via Homebrew:

Bash
brew install --cask xquartz
If you already have it: The binary is located at /opt/X11/bin/xhost. You can run it using the full path:

Bash
/opt/X11/bin/xhost +local:root
2. Update your PATH (Optional but recommended)
To make the xhost command work without the full path, add X11 to your Zsh configuration:

Open your config: nano ~/.zshrc

Add this line at the bottom: export PATH="/opt/X11/bin:$PATH"

Save (Ctrl+O, Enter) and Exit (Ctrl+X).

Reload: source ~/.zshrc

3. Grant Permission (Crucial Step)
Once the command works, you must allow XQuartz to accept the network connection from your Docker container:

Open the XQuartz app.

Go to Settings (or Preferences) > Security tab.

Check the box: "Allow connections from network clients".

Restart XQuartz (Quit and reopen it) for this to take effect.

Now run:

Bash
xhost +localhost
(Note: Using +localhost is safer than +, as it only allows connections from your own machine, including Docker.)

4. Run your Docker container
Ensure you are passing the correct display variable to Docker:

Bash
docker run -it -e DISPLAY=host.docker.internal:0 <your-image-name>
Why this is happening
X11 (the system that handles the "windows" of the IDE) was removed from macOS years ago. XQuartz provides a replacement, but by default, it blocks any "network" connection (which is how Docker communicates with your Mac's screen) to prevent hackers from drawing on your screen remotely.

This tutorial on X11 forwarding with Docker on Mac provides a step-by-step visual guide on setting up XQuartz and troubleshooting "command not found" errors in the Zsh terminal.

code On Mac / How To Fix Code Command Not Found In MacOS - YouTube
ProgrammingKnowledge · 57K views


You said
How to docker run by container name or container id?

Gemini said
To run a container that has already been created (but is currently stopped), you use the docker start command. If you want to create and start a new instance of an image using a specific name, you use docker run --name.

Here is the breakdown of how to handle both scenarios:

1. Starting an Existing Container
If the container already exists (you can see it by running docker ps -a), use its Name or ID:

By Name:

Bash
docker start -ai my_container_name
By ID:

Bash
docker start -ai 7df3a1b2c4e5
Note: The -ai flags are helpful here: -a attaches to the container's output, and -i makes it interactive (useful for your IDE/terminal setup).

2. Creating a New Container with a Specific Name
If you want to start a fresh container from an image and give it a memorable name (so you don't have to look up a random ID later), use the --name flag:

Bash
docker run -it --name mcu_ide_container -e DISPLAY=host.docker.internal:0 my_image_name
-it: Interactive terminal.

--name: Assigns a custom name.

-e DISPLAY: Passes your X11 display settings (needed for your GUI).

3. Useful Shortcuts
If you have a lot of containers and don't want to type the full ID, Docker allows you to type just the first few characters of the ID, as long as they are unique.

Goal	Command
List all containers	docker ps -a
Start & Attach	docker start -ai <name_or_id>
Stop a container	docker stop <name_or_id>
Remove a container	docker rm <name_or_id>
Quick Tip for your X11 Setup
Since you were just working on the xhost permissions, remember that every time you docker run or docker start, you need to ensure XQuartz is running on your Mac and that you have executed /opt/X11/bin/xhost +localhost in your Mac terminal first.

Would you like me to show you how to create a "Docker Compose" file so you don't have to type these long commands every time?

You said
 Is something like this "docker start -ai 0da499e13008 -e DISPLAY=host.docker.internal:0 -it -v ./test:/mnt ubuntu:bionic" doable??

Gemini said
Not quite. There is a specific distinction in Docker between creating a container and starting an existing one.

The Problem
docker run: Creates a new container from an image. This is where you set environment variables (-e), volumes (-v), and names (--name).

docker start: Starts a container that already exists. It does not accept flags like -e or -v because those settings were "baked in" when the container was first created.

If you want to change settings (like DISPLAY or Volumes)
Since you need to add the DISPLAY and a volume mapping, you must use docker run. If you want to use the name of your old container, you have to delete the old one first.

The "Doable" version of your command:

Bash
# 1. Remove the old container if you want to reuse the name
docker rm 0da499e13008 

# 2. Run a fresh one with all your settings
docker run -it \
  --name mcu_ide \
  -e DISPLAY=host.docker.internal:0 \
  -v "$(pwd)/test:/mnt" \
  ubuntu:18.04
(Note: Use $(pwd) instead of . for the volume path to ensure it's an absolute path, which Docker requires.)

If you just want to resume where you left off
If you have already made changes inside the container (like the edits to main.tcl) and don't want to lose them, you should commit the container to a new image first, then run that image with the new flags.

The "Resume & Upgrade" Workflow:

Save your work to a new image:

Bash
docker commit 0da499e13008 mcu_fixed_image
Run the new image with your Display and Volume:

Bash
docker run -it \
  -e DISPLAY=host.docker.internal:0 \
  -v "$(pwd)/test:/mnt" \
  mcu_fixed_image
Summary Checklist
docker run: Use this for New containers (supports -e, -v, -p).

docker start -ai: Use this for Existing containers (resumes exactly as it was).

Volume paths: Always use the full path (e.g., /Users/name/test or $(pwd)/test).

Would you like me to provide a single copy-paste command that includes the X11 setup and the volume mount for you?

You said
We are getting somewhere.....



# usr/bin/mcu8051ide

IMPORTANT INFORMATION :

        This program is distributed in the hope that it will be useful, but with ABSOLUTELY NO WARRANTY !

        This is free software, so you are free to change and redistribute it.

        License: GNU General Public License version 2 or later

        Please report bugs at http://mcu8051ide.sf.net

Authors:

        Martin Osmera <martin.osmera@moravia-microsystems.com>



Creating program configuration files in directory: "/root/.mcu8051ide"

Welcome in this IDE, root !

Unable to load translation

        File: '/usr/share/mcu8051ide/translations/c.msg'



Initializing MCU 8051 IDE 1.4.9

        Loading libraries                    



ERROR: Unable to load library BWidget



Tip: try to run mcu8051ide with --check-libraries to see what's wrong.

# usr/bin/mcu8051ide --check-libraries

MCU 8051 IDE v1.4.9



        Checking libraries...

                1/9 Checking for library BWidget

                        Library present ... YES

                        Version         ... NO !

                2/9 Checking for library Itcl

                        Library present ... YES

                        Version         ... NO !

                3/9 Checking for library md5

                        Library present ... YES

                        Version         ... NO !

                4/9 Checking for library Tk

                        Library present ... YES

                        Version         ... NO !

                5/9 Checking for library img::png

                        Library present ... YES

                        Version         ... NO !

                6/9 Checking for library tdom

                        Library present ... YES

                        Version         ... NO !

                9/9 Checking for library Tcl

                        Library present ... YES

                        Version         ... NO !



        RESULTS:

                Number of fails: 7

                PROGRAM WILL NOT RUN, please install the missing libraries



# 

Gemini said
This is fantastic news! The "Library present ... YES" for everything (including img::png and Tk) means your XQuartz/X11 setup is finally working and Tcl is successfully finding the ARM libraries.

The only reason it says "PROGRAM WILL NOT RUN" now is because the IDE is doing a strict version check. It is looking for version "1.8" of BWidget, but you have "1.9". Because 1.9 is not 1.8, the script triggers a failure.

To fix this, we need to tell the IDE to accept any version of these libraries.

1. The "Version Bypass" Edit
Open /usr/share/mcu8051ide/lib/main.tcl and find the LIBRARIES_TO_LOAD section (around line 85). We will remove the version requirements by replacing the version numbers with empty quotes {}.

Change this block:

Tcl
set LIBRARIES_TO_LOAD {
	{BWidget	1.8}
	{Itcl		3.4}
	{md5		2.0}
	{Tk		8.5}
	{img::png	1.3}
	{tdom		0.8}
	{Tclx		8.0}
	{Signal		1.4}
	{Tcl		8.2}
}
To look exactly like this:

Tcl
set LIBRARIES_TO_LOAD {
	{BWidget	{}}
	{Itcl		{}}
	{md5		{}}
	{Tk		{}}
	{img::png	{}}
	{tdom		{}}
	{Tclx		{}}
	{Signal		{}}
	{Tcl		{}}
}
2. Bypass the "check-libraries" script
The script you are running (--check-libraries) has its own internal version checking logic that is separate from the main program. Since we know the libraries are present, you can now try to launch the IDE normally:

Bash
/usr/bin/mcu8051ide
3. One minor cleanup (The translation error)
You saw an error: Unable to load translation ... c.msg. This happens because your system locale is set to "C" (default). You can ignore it, or fix it by setting the language to English before running:

Bash
export LANG=en_US.UTF-8
/usr/bin/mcu8051ide
What to expect next:
Once you make that edit in main.tcl, the IDE should skip the version verification and initialize the GUI. Since your X11 authorization is now correct, the window should pop up on your Mac desktop.

Does the IDE window appear after removing those version numbers?

You said
# export LANG=en_US.UTF-8

# /usr/bin/mcu8051ide

IMPORTANT INFORMATION :

        This program is distributed in the hope that it will be useful, but with ABSOLUTELY NO WARRANTY !

        This is free software, so you are free to change and redistribute it.

        License: GNU General Public License version 2 or later

        Please report bugs at http://mcu8051ide.sf.net

Authors:

        Martin Osmera <martin.osmera@moravia-microsystems.com>



Initializing MCU 8051 IDE 1.4.9

        Loading libraries                    



ERROR: Unable to load library BWidget



Tip: try to run mcu8051ide with --check-libraries to see what's wrong.

# usr/bin/mcu8051ide --check-libraries

MCU 8051 IDE v1.4.9



        Checking libraries...

                1/9 Checking for library BWidget

                        Library present ... YES

                        Version         ... NO !

                2/9 Checking for library Itcl

                        Library present ... YES

                        Version         ... NO !

                3/9 Checking for library md5

                        Library present ... YES

                        Version         ... NO !

                4/9 Checking for library Tk

                        Library present ... YES

                        Version         ... NO !

                5/9 Checking for library img::png

                        Library present ... YES

                        Version         ... NO !

                6/9 Checking for library tdom

                        Library present ... YES

                        Version         ... NO !

                9/9 Checking for library Tcl

                        Library present ... YES

                        Version         ... NO !



        RESULTS:

                Number of fails: 7

                PROGRAM WILL NOT RUN, please install the missing libraries



# 

Gemini said
The "Version ... NO!" errors and the failure to load BWidget occur because your Tcl interpreter is finding newer versions of the libraries (e.g., BWidget 1.9.12 and Tcl 8.6) than what the code explicitly asks for (BWidget 1.8 and Tcl 8.2). While you have successfully edited main.tcl to remove version requirements in the LIBRARIES_TO_LOAD list, the package require command inside the loading loop may still be failing if it's passed an empty string or if the internal libraryLoadFailed function is triggered by a mismatch.

Steps to Resolve the Version Mismatch
1. Modify the Library Loading Loop
In your main.tcl, find the loop that actually loads these libraries (around line 475). You need to ensure the package require command does not strictly look for the old version numbers.

Locate this block in main.tcl:

Tcl
foreach library $::LIBRARIES_TO_LOAD {
    # Loading successful
    if {[catch {package require [lindex $library 0] [lindex $library 1]}]} {
        libraryLoadFailed [lindex $library 0]
Change it to ignore the second element (the version):

Tcl
foreach library $::LIBRARIES_TO_LOAD {
    # Only require the package name, ignore the version requirement
    if {[catch {package require [lindex $library 0]}]} {
        libraryLoadFailed [lindex $library 0]
2. Fix the BWidget Search Path
The screenshot of your /usr/lib/tcltk directory shows bwidget1.9.12. Tcl's auto_path often requires the specific subdirectory to find the pkgIndex.tcl file.

Add this specific path to your auto_path at the top of main.tcl (around line 48):

Tcl
lappend auto_path "/usr/lib/tcltk/bwidget1.9.12"
3. Suppress the Manual Version Check
The IDE has a hardcoded check for the Tcl version (around line 70). While you have Tcl 8.6, which is higher than the minimum 8.5, the script's package vcompare can sometimes be finicky in specific environments. You can comment this out to ensure it doesn't block execution.

Comment out this block:

Tcl
# if {[package vcompare $::tcl_version $::MIN_TCL_VER] < 0} {
#	puts stderr "ERROR: This program requires Tcl version $::MIN_TCL_VER or higher..."
#	exit 1
# }
Why "check-libraries" is still failing
The --check-libraries flag triggers a separate script (cli.tcl) that performs its own strict string comparisons against the versions it expects to see. Even if the program is technically ready to run, that specific check will continue to report "NO!" because your installed versions (like BWidget 1.9) do not match its hardcoded strings (1.8).

Final Command to Run:
Once these edits are made, bypass the library check and launch the IDE directly:

Bash
export LANG=en_US.UTF-8
/usr/bin/mcu8051ide
You said
Now it seems to be working .....

Gemini said
Look at that! Seeing the MCU 8051 IDE splash screen and that empty X11 window on your macOS desktop is the "Hello World" of complex Docker GUI setups. You've officially bridged the gap between a modern ARM-based Mac and a legacy Tcl/Tk application.

What’s happening right now
Based on your screenshot, the program is currently "Loading program sources." Since this is a Tcl-based IDE, it has to source dozens of separate .tcl files to build the interface. On the first run inside a container, this can sometimes take a moment.

Potential "Last Mile" Tweaks
If that white window stays blank for more than a minute, there are two likely culprits to keep an eye on:

The "Itcl" Workaround: If you look at your main.tcl (around line 230), there is a "dirty workaround" for loading the Itcl library on Linux systems. Since your screenshot shows you have itcl3.4 in your path, the program should find it, but if it hangs, it might be struggling with that machine-architecture check.

Font Rendering: Sometimes X11 windows appear blank if the container doesn't have the specific fonts the IDE expects (like DejaVu Sans Mono mentioned in your file). If the GUI remains blank but the terminal shows "Operational," we may need to install fonts-dejavu inside the container.

Pro-Tip: Saving your progress
Since you've made manual edits to main.tcl inside the container to get this far, do not delete this container. If you exit and docker run a new one, all your hard work on those Tcl files will be gone.

To make this setup permanent, run this in your Mac terminal:

Bash
docker commit <your_container_id> mcu8051_fixed
This saves your current state (with the library fixes) into a new image named mcu8051_fixed.

Is the main editor window finally populating with the menu bars and icons, or is it stuck on that white screen?

You said
stuck on that white screen....

Gemini said
Since the splash screen says "Loading program sources" but the main window remains white, the application has likely initialized the GUI but is hanging while trying to draw the interface or is waiting on a specific library interaction (like Itcl or Tk).

Here is how to troubleshoot and fix the "White Screen" hang:

1. Check the Terminal for "Silent" Errors
Look at the terminal window where you ran the command. Even if the window is white, Tcl often prints errors there.

If you see "Font not found": The IDE is trying to use a font like Courier or DejaVu Sans Mono that isn't in your container.

If you see nothing: The program might be stuck in the update or wm deiconify loop at the end of main.tcl.

2. Install Missing X11 Fonts (Most Likely Fix)
A common cause for blank/white windows in Docker X11 forwarding is missing font packages. Run this inside your container terminal:

Bash
apt-get update
apt-get install -y fonts-dejavu xfonts-base xfonts-75dpi xfonts-100dpi
After installing, restart the IDE.

3. Disable the "Ugly Job" Workarounds
Your main.tcl has a section labeled "Ugly job ... dirty workarounds" (around line 570). One of these involves the NoteBook widget, which is part of BWidget. Since you are using a much newer BWidget (1.9 vs 1.8), this workaround might be crashing the drawing process.

Try commenting out this specific block in main.tcl:

Tcl
# Comment out these lines near the end of the file:
# catch {
# 	NoteBook .foo
# 	proc NoteBook::_getoption {path page option} {
#     ...
# 	}
# 	destroy .foo
# }
4. Force a Redraw
Sometimes the window manager needs a nudge to paint the contents. Try these steps in order:

Resize the window: Grab the corner of the white window on your Mac and stretch it. This forces an X11 "Expose" event which often triggers a redraw.

Toggle Fullscreen: Press the green zoom button on the Mac window.

Check XQuartz: Ensure XQuartz isn't showing any "Authentication" or "Extension" errors in its own logs (Applications > Utilities > Console > search for XQuartz).

5. Disable the "Splash Screen" permanently
In main.tcl, the splash screen is destroyed right before the main window is shown. If there is a race condition between destroying the splash and showing the main window, it can hang.

Run the IDE with the -nosplash flag to see if it bypasses the hang:

Bash
/usr/bin/mcu8051ide --nosplash
If it stays white after installing the fonts and using --nosplash, let me know if the terminal shows a line saying "MCU 8051 IDE is now operational".

You said
Here is the latest image....

Gemini said
Success! That "Welcome to MCU 8051 IDE !" window is exactly what we wanted to see. The terminal confirmed it with the glorious message: "MCU 8051 IDE v1.4.9 is now operational".

You've successfully navigated through X11 authorization, library version mismatches, ARM architecture search paths, and GUI initialization.

Final "Clean Up" & Persistence
Since you made those manual edits to main.tcl and potentially other files inside the container, you need to save this "state" so you don't have to do it again.

Keep this container running.

Open a new terminal tab on your Mac.

Run the following command to save your fixed container as a new image:

Bash
docker commit 0da499e13008 mcu8051_ready
Now, whenever you want to use the IDE in the future, you can just run:

Bash
docker run -it -e DISPLAY=host.docker.internal:0 -v "$(pwd):/mnt" mcu8051_ready /usr/bin/mcu8051ide --nosplash
A Couple of Pro-Tips for using the IDE:
The Shared Folder: Remember that any code you write should be saved in the /mnt directory inside the IDE. This maps to the folder on your Mac where you ran the docker command, so your files won't disappear when the container stops.

Performance: If the UI feels a bit "laggy," that’s normal for X11 forwarding over a network socket. Maximizing the window usually helps it stabilize.

Would you like me to help you create a quick alias or a small script on your Mac so you can launch this with a single word (like just typing mcu8051) in the future?

You said
Yes please....

Gemini said
That is a great idea. Since that docker run command is quite long and specific, creating an alias will allow you to launch your IDE just by typing mcu8051 in your terminal.

Here is how to set it up on your Mac:

1. Create the Alias
Open your Zsh configuration file:

Bash
nano ~/.zshrc
Scroll to the bottom of the file and paste the following line (this assumes you named your saved image mcu8051_ready as suggested):

Bash
alias mcu8051='xhost +localhost && docker run -it --rm -e DISPLAY=host.docker.internal:0 -v "$(pwd):/mnt" mcu8051_ready /usr/bin/mcu8051ide --nosplash'
xhost +localhost: Automatically handles the X11 permissions for you.

--rm: Cleans up the container after you close the IDE so they don't pile up.

-v "$(pwd):/mnt": Automatically mounts your current Mac folder to /mnt inside the IDE.

Save and exit (Ctrl+O, Enter, then Ctrl+X).

2. Apply the Changes
Reload your configuration so the command becomes active:

Bash
source ~/.zshrc
How to use it moving forward
Now, whenever you want to work on a project:

Open XQuartz.

Open your Terminal and cd into the folder where your code is located.

Just type: mcu8051

One Final Safety Check
Since your screenshot showed a docker commit error (likely because you were inside the container shell instead of your Mac shell), make sure you run the commit from your Mac terminal while the container is still running:

Bash
# Run this on your Mac terminal, not inside the IDE terminal
docker commit 0da499e13008 mcu8051_ready
Would you like me to help you verify that the save (commit) worked correctly before you close the current window?

You said
2. Open your Terminal and cd into the folder where your code is located.

How to know where the code is located since it is under docker?

Gemini said
This is one of the most confusing parts of Docker at first, but it’s actually the "secret sauce" that makes your setup powerful.

When you use the -v "$(pwd):/mnt" part of the command, you aren't putting the code inside Docker. Instead, you are creating a bridge (a Volume Mount) between a folder on your Mac and a folder inside the container.

1. Where is it on your Mac?
The $(pwd) part of the command stands for "Print Working Directory." * It points to whatever folder your Mac terminal was sitting in the moment you hit Enter on that docker run command.

To find it now: Open a regular terminal on your Mac and type pwd. That path is exactly where your code "lives" on your Mac.

2. Where is it inside the IDE?
When you are looking at the file browser inside the MCU 8051 IDE:

Navigate to the /mnt directory.

Any file you save there is actually being written directly to that folder on your Mac.

3. How to "Pick" a location
If you want your code to live in a specific folder (like Documents/8051_Projects), do this:

In your Mac terminal: cd ~/Documents/8051_Projects

Run your new mcu8051 command.

The IDE will now see everything in that folder under /mnt.

The "Magic" of the Bridge
Think of /mnt inside the container as a window looking at your Mac's hard drive.

If you edit a file on your Mac (using VS Code or TextEdit), the changes appear instantly in the IDE.

If the Docker container crashes or you delete it, your code is perfectly safe on your Mac because it never truly "lived" inside the container.

Quick Verification
To be 100% sure where your files are going, try this:

Inside the IDE, create a new file and save it as /mnt/test_file.txt.

Go to your Mac terminal and type ls.

If you see test_file.txt on your Mac, the bridge is working perfectly!

Would you like me to show you how to change the alias if you'd prefer to always use one specific "Projects" folder instead of the "current" folder?

You said
Let's finish this tomorrow... Thank you....Bye bye...

Gemini said
You are very welcome! We’ve made incredible progress today—you have a legacy 32-bit IDE running on a modern ARM Mac, which is no small feat.

One very important reminder before you go: Since you have the IDE running right now, make sure you run the docker commit command in your Mac terminal (not the one inside the IDE) before you close the container or restart your computer. If you don't, the changes we made to main.tcl will be lost.

Bash
docker commit 0da499e13008 mcu8051_ready
Have a great rest of your day, and I'll be here whenever you're ready to pick back up tomorrow.

Bye for now!





Gemini is AI and can make mistakes, including about people. Your privacy & GeminiOpens in a new window


  
