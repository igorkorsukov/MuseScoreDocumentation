# Fedora Compiling

This page contains only the compilation instructions that are specific to Fedora and related RPM-based distributions.

Tested working on: Fedora 21 and 22

Install dependencies:  
_NOTE: Use "yum" instead of "dnf" on older systems._

```bash
sudo dnf install -y gcc gcc-c++ qt-devel pulseaudio-libs-devel alsa-lib-devel jack-audio-connection-kit-devel qt5-qtbase-devel qt5-qttools-libs-designercomponents qt5-qttools-devel portaudio-devel poppler-qt5-devel qt5-qtdeclarative-devel qt5-qtscript-devel qtermwidget-qt5-devel qt5-qtwebkit-devel qt5-qtxmlpatterns-devel qt5-qtquick1-devel qt5-qtsvg-devel qt5-qttools-devel qt5-qttools-static lame-devel libsndfile-devel freetype-devel texlive-scheme-basic qt5-qtwebengine qt5-qtwebengine-devel libvorbis-devel
```

Tested working on Fedora 28 :

```bash
sudo dnf install -y gcc gcc-c++ qt-devel pulseaudio-libs-devel alsa-lib-devel jack-audio-connection-kit-devel qt5-qtbase-devel qt5-qttools-libs-designercomponents qt5-qttools-devel portaudio-devel poppler-qt5-devel qt5-qtdeclarative-devel qt5-qtscript-devel qtermwidget-qt5-devel qt5-qtwebkit-devel qt5-qtxmlpatterns-devel qt5-qtsvg-devel qt5-qttools-devel qt5-qttools-static lame-devel libsndfile-devel freetype-devel texlive-scheme-basic qt5-qtwebengine qt5-qtwebengine-devel libvorbis-devel portmidi-devel
```

Now follow the [generic instructions for compiling on Linux](Linux_Compiling.md)