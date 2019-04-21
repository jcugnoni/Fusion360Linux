# Fusion360Linux

My notes on how to run Autodesk Fusion 360 under Linux (CAELinux 2018 / Ubuntu 16.04 amd64)
Procedure adapted from https://gist.github.com/probonopd/0fab254aa0b6fc371d8db641822bd530

# OS: CAELinux 2018 / Ubuntu 16.04 amd64

#please note that you need to start with no initial wine config in ~/.wine. If not, create a new "prefix" (=config) and set WINEPREFIX=your_path_to_wine_prefix

# Graphics driver and Vulkan support (not sure if it is really mandatory...)

#NVIDIA install nvidia proprietary drivers from additionnal drivers, test with version 384 for me 
#AMD: probably fairly similar, but see online docs to enable Vulkan on recent AMD GPU

sudo add-apt-repository ppa:graphics-drivers/ppa

sudo apt-get update

#This will open the additional driver installer

software-properties-gtk --open-tab=4

#and finally vulkan

sudo apt-get install vulkan-utils

#for intel GPU, also install mesa vulkna drivers:
sudo apt-get install mesa-vulkan-drivers 

# install winehg-staging (wine 4.6)

sudo dpkg --add-architecture i386 

wget -nc https://dl.winehq.org/wine-builds/winehq.key

sudo apt-key add winehq.key

sudo apt-add-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ xenial main' 

sudo apt-get update

#install winehq staging (4.6 for me)

sudo apt install --install-recommends winehq-staging      

# update winetricks

cd "${HOME}"

wget  https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks

chmod +x winetricks


# DLLs

~/winetricks atmlib gdiplus msxml3 msxml6 vcrun2017 corefonts fontsmooth=rgb winhttp win10

#DirectX 11 via Vulkan

wget https://github.com/doitsujin/dxvk/releases/download/v1.0.2/dxvk-1.0.2.tar.gz

tar xvzf dxvk-1.0.2.tar.gz && cd dxvk-1.0.2

WINEPREFIX=~/.wine ./setup_dxvk.sh install

cd ..

# Patch and Install Fusion 360

wget https://dl.appstreaming.autodesk.com/production/installers/Fusion%20360%20Admin%20Install.exe

sudo apt-get install p7zip-full git curl

7z x -osetup/ "Fusion 360 Admin Install.exe"

curl -Lo setup/platform.py https://github.com/python/cpython/raw/3.5/Lib/platform.py

sed -i 's/winver._platform_version or //' setup/platform.py

wine setup/streamer.exe -p deploy -g -f log.txt --quiet

# run script in addition to the start menu entry that is automatically added

#please edit $WINEPREFIX to match your config. If using default prefix it is equal to ~/.wine

cat << EOF > runFusion360.sh

#!/bin/bash

env WINEPREFIX="/home/jcugnoni/Fusion360" /opt/wine-staging/bin/wine C:\\\\\\\\windows\\\\\\\\command\\\\\\\\start.exe /Unix /home/jcugnoni/Fusion360/dosdevices/c:/ProgramData/Microsoft/Windows/Start\ Menu/Programs/Autodesk/Autodesk\ Fusion\ 360.lnk

EOF

chmod +x runFusion360.sh

# RUN FUSION 360 a first time !

./runFusion360.sh

# Fixing issues with adcefbrowser and online connection:

#- 1. run winecfg, change platform to Win7 for both Default and eventually also adcefbrowser app if specified (see below)
In case ofgraphics issues, go to preferences->General and switch from DX11 to DX9

#- 2. run "winecfg", Disable dx11 as a default setting (ps JC: indeed d3d11 was not detected by fusion in my default config, so I did not change that in the end), then add the adcefbrowser.exe in the applications tab in winecfg (its in [AUTODESK PATH]/webdeploy/production/[folder with the most files in it, more than 3]/WIN64/ ) and create an override in the libraries tab to set d3d11.dll on native for adcefbrowser.exe. This point was not needed for me.

