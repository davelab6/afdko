# FDK Build Notes v2.5

v1.05

## FDK build layout

The FDK build directory tree layout is pretty straightforward.
Its basic structure is:

```
FDK/Tools/Programs/
	<component>/
			build/
				<platform>/<compiler>/<debug|release>
			exe/
				<platform>/<debug|release>
			source/
```
When a tool uses a library, then the project for the main tool contains the projects for all the libraries.
The libraries are grouped under the main directory:
```
	<component>/
		<library group name>/
			build/
			lib/
			api/
			source/
```
The sub-directories for `build/` and `lib/` being the same as for the program `build/` and `exe/` directories.

### Special cases

`tx`, `mergeFonts`, and `rotateFonts` share a common set of libraries and resource files.
These libraries are grouped under the public directory in:
```
FDK/Tools/Programs/public/lib/
			build/
			lib/
			api/
			resource/
			source/
```
All programs are built with different build systems on different platforms;

| Platform | Build System |
|---|---|
| Mac OS X | Xcode 4.5 |
| Windows | Visual C++ 2010 |
| Ubuntu 12.4 | GCC |

## FDK distribution package.

The FDK distribution directory tree is built by:

1. building all programs with a platform specific script:

| Platform | Build System |
|---|---|
| Mac OS X | `FDK/Tools/Programs/BuildAll.sh` |
| Windows | `FDK/Tools/Programs/BuildAll.cmd` |
| Ubuntu 12.4 | `FDK/Tools/Programs/BuildAllLinux.sh` |

2. deleting the Programs directory and any unused programs and files (e.g under Windows Tools/osx and Tools/Linux.)

3. building the Python interpreter for the FDK, and copying it into the correct location in the directory tree.

4. install the various Python modules that the FDK depends on

## Building the Python Interpreter

The FDK will use the Python Interpreter whose path is specified in line 4 of the file `setFDKPaths.py`. 
This line can be changed to point to any other Python interpreter program on the system. 
However, if you choose to use another Python interpreter, you must add several third party packages.

The FDK contains a copy of the Python interpreter.
This is used to run the FDK Python scripts.
To build this:
- download the latest Python 2.7 source from www.python.org to a `<local directory>`
- unzip it
- Open a Terminal window, and change the current directory to the directory `<local directory>/Python-2.7/`
- cd to Modules, and copy `Setup.dist` to Setup
- edit Setup to enable `zlib` (needed by `ttx`) and if on OSX or Windows then enable `readlinez`
- run `./configure --prefix=AFDKOPythonBuild`
- run `make install`
- run `mv AFDKOPythonBuild FDK/Tools/osx|win/Python/AFDKOPython27`

For building the 32-bit Linux python:
- run `sudo apt-get install build-essential`
- run `sudo apt-get install zlib1g-dev`
- use the same steps as above in a 32 bit Linux virtual machine, but in the last command use an absolute path to `AFDKOPythonBuild`
- run `cp /lib/i386-linux-gnu/libz.so.1.2.3.4 Python/lib/python2.7/lib-dynload/`
- run `cd Python/lib/python2.7/lib-dynload/; ln -s libz.so.1.2.3 libz.so.1`

## Dependencies

### fontTools

This module is needed for the `ttx` and `ttxn` tools to work. 
Use Behdad Esfahbod's GitHub v2.5 branch as it fixes a number of bugs and is not dependent on the numPy module.
To allow the robofab libraries to work with this branch of fontTools:

```sh
git clone https://github.com/behdad/fonttools.git;
cd fonttools;
AFDKOPython setup.py install;
cp site-packages/FontTools/fontTools/misc/xmlWriter.py site-packages/FontTools/xmlWriter.py;
```

### mutatorMath

First install the ufo3 branch of fontMath:

```sh
git clone -b ufo3 https://github.com/typesupply/fontMath.git;
cd fontMath; 
AFDKOPython setup.py install;
```

Next, the ufo3k branch of robofab
```sh
git clone -b ufo3k https://github.com/robofab-developers/robofab.git;
cp -ra robofab FDK/Tools/osx/Python/AFDKOPython27/lib/python2.7/site-packages;
cd FDK/Tools/osx/Python/AFDKOPython27/lib/python2.7/site-packages/robofab;
AFDKOPython install.py;
mv LIB /tmp;
mv LICENSE.txt /tmp;
rm -r *;
mv /tmp/LIB .;
mv /tmp/LICENSE.txt .;
```
Next, the ufo3 branch of the defcon library:
```sh
git clone -b ufo3 https://github.com/typesupply/defcon.git;
cd defcon;
AFDKOPython setup.py install;
```

The install script has a bug, in that it does not copy any of the subdirectories of defcon. Copy these manually.

Next, master branch of mutatorMath:
```
git clone https://github.com/LettError/MutatorMath.git;
cp -r MutatorMath FDK/Tools/osx/Python/AFDKOPython27/lib/python2.7/site-packages/;
cd FDK/Tools/osx/Python/AFDKOPython27/lib/python2.7/site-packages/MutatorMath;
mv LIB /tmp;
mv LICENSE.txt /tmp;
rm -r *;
mv /tmp/LIB .;
mv /tmp/LICENSE.txt .;
```
Manually create the file "mutatorMath-master.pth", and put in it the relative path to the mutatorMath-master directory, e.g just  the line "mutatorMath-master".

In the AFDKOPython directory, check the path references in all the *.pth files in `FDK/Tools/osx/Python/AFDKOPython27/lib/python2.7/site-packages` are all relative to the site-packages directory.

## Testing/building systems.

The command-line tools are all implemented as either binary C programs or shell scripts. Each shell script finds the Python included in the FDK and then uses that Python to run a specific Python script with appropriate options.

The path `FDK/Tools/$platform` must be added to the current user's `PATH` environment variable for the commands to work.

If your primary system is Mac OSX, then a useful approach is to use MVWare Fusion to make virtual systems. For Linux:

- download the Ubuntu Linux 32 bit iso file (x86)
- make a new VM machine with VMWare Fusion, select the iso file as the source
- install all updates
- use the Settings, Share option to share the FDK source tree on your Mac OSX file system
- choose the VMWare Fusion menu option "VM Machine, Install VMWare Tools"

This mounts an iso file in the Linux VM. Copy the contents to a local folder in the Ubuntu Linux system. Decompress the tar file, look inside for vmware-install.pl. Run this with sudo, take default answer for all.

Do not /usr/bin/vmware-user at the end - you don't need additional X Windows support.
