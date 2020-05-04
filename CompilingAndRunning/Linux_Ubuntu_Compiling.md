# Ubuntu 16.04LTS and 18.04LTS Compiling

This page contains only the compilation instructions that are specific to Ubuntu and related Debian-based distributions.  
__Tested working on: Ubuntu 14.04 LTS, Ubuntu 16.04 LTS and Ubuntu 18.04 LTS__

_It should also work with the various Ubuntu flavours, including Ubuntu MATE, Ubuntu Studio, Kubuntu and Xubuntu, etc.
It should work on derivatives such as Linux Mint, and it may also work on the corresponding Debian and Debian-derived distributions like Raspian._

## Install dependencies

Recent distributions have up-to-date versions of most of the necessary packages are in the default repositories.

```bash
sudo apt-get install git cmake g++
sudo apt-get install libasound2-dev portaudio19-dev libmp3lame-dev libsndfile1-dev libportmidi-dev
sudo apt-get install libssl-dev libpulse-dev libfreetype6-dev libfreetype6
```

If you get an error message like this while compiling:

```bash
Failed to find "GL" in "".
```

Then try installing these additional libraries:

```bash
sudo apt-get install libdrm-dev libgl1-mesa-dev libegl1-mesa-dev
```

__Important Note: It is now necessary to install Qt 5.8, or later__  
MuseScore uses Qt to achieve a consistent look and feel across different platforms (Mac, Windows & Linux). Qt is updated more frequently than any other dependency. __Having an out-of-date version of Qt is the most common cause of problems__ as far reaching as strange window behavior, keyboard shortcuts not working, or the code failing to compile outright.

If your repository has Qt version 5.8, or later, you can get Qt from your repository:

```bash
sudo apt-get install qtbase5-dev qttools5-dev qttools5-dev-tools qtwebengine5-dev\
qtscript5-dev libqt5xmlpatterns5-dev libqt5svg5-dev libqt5webkit5-dev
```

If your repository does not have 5.8, or later then follow these steps to install it:  
_Note that having two different versions of Qt installed can cause difficulties. If you installed using "apt-get install" above, then you may want to remove that before following these steps._

* Download the latest version of Qt (currently 5.8) from "http://qt-project.org/">[http://qt-project.org](http://qt-project.org/). The file you download is actually an installation script called something like "qt-unified-linux-x64-2.0.3-online.run". (It will be called something different for 32-bit machines).
* Move the installer (file you downloaded) to your Home directory and open a terminal window (Ctrl+Alt+T on Ubuntu).
* Give the installer execute permissions:

```bash
sudo chmod +x qt-unified-linux-x64-2.0.3-online.run
```

* Run the installer ("sudo" is not required if you choose to install to your Home directory in Step 5):

```bash
sudo ./qt-unified-linux-x64-2.0.3-online.run
```

* Follow the installation wizard and write down the installation directory (default: `/opt/Qt`. You can choose somewhere else if you want but make sure it doesn't have spaces anywhere in the path). Finish the install.
* In your file browser, navigate to the installation directory and find the path to the Qt `bin` directory. (Mine is `/opt/Qt/5.8/gcc_64/bin`. This will be different on different machines.)
* Add the `bin` directory to your `$PATH` environment variable so that MuseScore knows where it is. (Modify the following command with the correct path as appropriate.):

```bash
echo 'export PATH=/opt/Qt/5.8/gcc_64/bin:$PATH' >> ~/.bashrc
```

* Load your new $PATH variable.

```bash
source  ~/.bashrc
```

You can check the Qt installation and version by typing "`qmake -version`" in a terminal.

__If you experience any problems with MuseScore__, first check you have the latest copy of the MuseScore source code, and then __check you have the latest copy of Qt__. Only once you have confirmed this (and done the same for the other dependencies) should you consider creating a bug report in the issue tracker.

## Compiling the code

Now follow the [generic instructions for compiling on Linux](Linux_Compiling.md).

_The remaining steps on this page are optional. Read them if you experience problems or wish to completely uninstall MuseScore and it's dependencies._

### Uninstall dependencies

Remember all the packages installed previously in order to compile MuseScore? Actually, even more were installed because each installed package has its own dependencies. If you don't want them anymore, here's how to remove them.

When you proceed with the installation, `apt-get` outputs in the terminal the complete list of packages that are to be installed. If you copy this list, the packages can be easily removed later with this command:

```bash
sudo apt-get remove --purge LIST
```

If you did not copy this output and still want to remove all the packages, it's a little more complicated but still feasible. For each command entered, for example:

```bash
sudo apt-get install git cmake g++
```

an entry is added in the following log file: `/var/log/apt/history.log`. Open it with a text editor and find the relevant entry. Example:

```bash
<strong>Start-Date:</strong> START DATE OF INSTALLATION
<strong>Commandline:</strong> apt-get install git cmake g++
<strong>Install:</strong> libstdc++-4.9-dev:amd64 (4.9.1-15ubuntu1, automatic), libc-dev-bin:amd64 (2.19-10ubuntu1, automatic), g++:amd64 (4.9.1-4ubuntu2), g++-4.9:amd64 (4.9.1-15ubuntu1, automatic), liberror-perl:amd64 (0.17-1.1, automatic), git-man:amd64 (2.1.0-1, automatic), git:amd64 (2.1.0-1), cmake:amd64 (2.8.12.2-0ubuntu5), cmake-data:amd64 (2.8.12.2-0ubuntu5, automatic), linux-libc-dev:amd64 (3.16.0-16.22, automatic), libc6-dev:amd64 (2.19-10ubuntu1, automatic)
<strong>Upgrade:</strong> MAY BE THERE BUT IT'S NOT RELEVANT
<strong>End-Date:</strong> END DATE OF INSTALLATION
```

Text that must be copied follows the key _Install_:

```bash
libstdc++-4.9-dev:amd64 (4.9.1-15ubuntu1, automatic), libc-dev-bin:amd64 (2.19-10ubuntu1, automatic), g++:amd64 (4.9.1-4ubuntu2), g++-4.9:amd64 (4.9.1-15ubuntu1, automatic), liberror-perl:amd64 (0.17-1.1, automatic), git-man:amd64 (2.1.0-1, automatic), git:amd64 (2.1.0-1), cmake:amd64 (2.8.12.2-0ubuntu5), cmake-data:amd64 (2.8.12.2-0ubuntu5, automatic), linux-libc-dev:amd64 (3.16.0-16.22, automatic), libc6-dev:amd64 (2.19-10ubuntu1, automatic)
```

To remove these packages:

```bash
sudo apt-get remove --purge $(echo "TEXT COPIED ABOVE" | sed -r 's/\s\([^\)]+\),?//g')
```

__IMPORTANT:__ by doing so, you may remove some Qt packages needed to run MuseScore. The following command should ensure that you have all that is required:

```bash
sudo apt-get install libqt5core5a libqt5gui5 libqt5network5 libqt5xml5 libqt5xmlpatterns5 \
libqt5svg5 libqt5printsupport5 libqt5webkit5
```

### Note about `lrelease`

When we invoke `make`, a call to the executable `lrelease` is done during the process. However, it's not the command `/usr/bin/lrelease` from the package `qtchooser` but the one from `qttools5-dev-tools`. That's why the package `qttools5-dev-tools` is added in the list of dependencies to install. If it was not installed, the following error would occur during the `make` invocation:

```bash
make[4]: Entering directory '/path/to/MuseScore/build.release'
/bin/sh: 1: /usr/lib/x86_64-linux-gnu/qt5/bin/lrelease: not found
CMakeFiles/lrelease.dir/build.make:49: recipe for target 'CMakeFiles/lrelease' failed
make[4]: *** [CMakeFiles/lrelease] Error 127
make[4]: Leaving directory '/path/to/MuseScore/build.release'
```

and the following during the `make install` invocation:

```bash
CMake Error at share/locale/cmake_install.cmake:36 (FILE):
  file INSTALL cannot find
  "/path/to/MuseScore/share/locale/mscore_af.qm".
Call Stack (most recent call first):
  share/cmake_install.cmake:43 (INCLUDE)
  cmake_install.cmake:45 (INCLUDE)


Makefile:62: recipe for target 'install' failed
make[1]: *** [install] Error 1
make[1]: Leaving directory '/path/to/MuseScore/build.release'
Makefile:92: recipe for target 'install' failed
make: *** [install] Error 2
```
