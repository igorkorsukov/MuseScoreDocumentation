# OpenSUSE Compiling

This page contains only the compilation instructions that are specific to openSUSE distributions.

## Install the tools required for building MuseScore

```bash
sudo zypper install -y git gcc7 gcc-c++ make cmake
```

## Install the dependencies required by building MuseScore

```bash
sudo zypper install -y alsa-devel libpulse-devel libjack-devel portaudio-devel libQt5Core-devel \
                  libQt5Gui-devel libQt5Network-devel libQt5Test-devel libQt5QuickControls2-devel \
                  libQt5Xml-devel libqt5-qtsvg-devel libQt5Sql-devel libQt5Widgets-devel \
                  libQt5PrintSupport-devel libQt5Concurrent-devel libQt5OpenGL-devel libqt5-linguist-devel \
                  libQt5Help5 libqt5-qttools-devel libqt5-qtwebengine-devel libmp3lame-devel \
                  libsndfile-devel  portmidi-devel libvorbis-devel
```

## Remark regarding `qmake`

On openSUSE, the tool `qmake` is refering to the Qt4 version of this tool. However, since MuseScore is using Qt5, the tool `qmake-qt5` should be used!
This can be done by editing the file `build/FindQt5.cmake` and change line 38

```text
find_program(QT_QMAKE_EXECUTABLE **qmake**)
```

into

```text
find_program(QT_QMAKE_EXECUTABLE **qmake-qt5**)
```

An alternative is to create a script `qmake` in you path with the contents:

```bash
#!/bin/bash --noprofile
exec qmake-qt5 ${*}
```

## Compiling the code

Now follow the [generic instructions for compiling on Linux](Linux_Compiling.md)

## Tested on

These instructions are tested on

* openSUSE 15.1
* openSUSE Tumbleweed as per September 9th, 2019

## Note on openSUSE Tumbleweed

Please note openSUSE Tumbleweed is a [rolling release](https://en.opensuse.org/Portal:Tumbleweed) with the most recent release of the used packages. At this moment (September 12th, 2019) the installed version of Qt5 is Qt5 5.13.1. This version causes several compile warnings but the resulting executable runs.  
To use the recommended version of Qt5, installed the required Qt packages from another source, e.g. the [openSUSE download site](https://software.opensuse.org).
