# Ettus X310 PCI
This is a guide on how to interface the Ettus X310 SDR over PCI to a computer to allow for higher transfer speeds than conventional Ethernet communication.

![20250723_141332](https://github.com/user-attachments/assets/8d5df17c-d2ce-450b-85a0-76698ad5d22e)

## Requirements:

- **Ubuntu 18.04 Bionic Beaver**
- **PCI Card adapter**

It was found that to successfully install the NI RIO PCI drivers, Ubuntu versions other than 18.04 were found be troublesome and had many dependancy issues.

Run the following to ensure you have all the relevant software package dependancies before continuing:

```shell
sudo apt-get -y install git swig cmake doxygen build-essential libboost-all-dev libtool libusb-1.0-0 libusb-1.0-0-dev libudev-dev libncurses5-dev libfftw3-bin libfftw3-dev libfftw3-doc libcppunit-1.14-0 libcppunit-dev libcppunit-doc ncurses-bin cpufrequtils python-numpy python-numpy-doc python-numpy-dbg python-scipy python-docutils qt4-bin-dbg qt4-default qt4-doc libqt4-dev libqt4-dev-bin python-qt4 python-qt4-dbg python-qt4-dev python-qt4-doc python-qt4-doc libqwt6abi1 libfftw3-bin libfftw3-dev libfftw3-doc ncurses-bin libncurses5 libncurses5-dev libncurses5-dbg libfontconfig1-dev libxrender-dev libpulse-dev swig g++ automake autoconf libtool python-dev libfftw3-dev libcppunit-dev libboost-all-dev libusb-dev libusb-1.0-0-dev fort77 libsdl1.2-dev python-wxgtk3.0 git libqt4-dev python-numpy ccache python-opengl libgsl-dev python-cheetah python-mako python-lxml doxygen qt4-default qt4-dev-tools libusb-1.0-0-dev libqwtplot3d-qt5-dev pyqt4-dev-tools python-qwt5-qt4 cmake git wget libxi-dev gtk2-engines-pixbuf r-base-dev python-tk liborc-0.4-0 liborc-0.4-dev libasound2-dev python-gtk2 libzmq3-dev libzmq5 python-requests python-sphinx libcomedi-dev python-zmq libqwt-dev libqwt6abi1 python-six libgps-dev libgps23 gpsd gpsd-clients python-gps python-setuptools
```

Then reboot the system

```shell
sudo reboot
```

---
# Installation Steps

To communicate to the X310, the `uhd` drivers and `gnu-radio` must first be installed. These must be compiled from source with the correct version in order to work.

Make the `workarea` folder where the packages can be built:

```shell
cd $HOME
mkdir workarea
cd workarea
```

## Install `uhd` Drivers:

Clone the `uhd` drivers to this folder to build later:

```shell
git clone https://github.com/EttusResearch/uhd
cd uhd
```

You must set the version to make later:

```shell
git checkout release_003_009_005
```

Change directory into `/host` and create your build folder:

```shell
cd host
mkdir build
cd build
```

Inside the `/build` folder, run:

```shell
cmake ..
```

Then run `make`, this can be sped up using more cores by `-j #` where # is the number of cores you want to use:

```shell
make -j 4
```

You can test if your build was successful by running:

```shell
make test
```

Install `uhd` by running as sudo:

```shell
sudo make install -j 4
```

Update your system cache:

```shell
sudo ldconfig
```

You will also need to define the `LD_LIBRARY_PATH` environment variable. This can be done through running on terminal the below or adding it to your `$HOME/.bashrc` to run at startup.

```shell
export LD_LIBRARY_PATH=/usr/local/lib
```

Test that your system has installed `uhd` correctly by running:

```shell
uhd_find_devices
```

This will return something similar to:
```
linux; GNU C++ version 7.5.0; Boost_106501; UHD_003.009.005-0-g32951af2
No UHD Devices Found
```

## Download `uhd` FPGA images:

To interface with the SDR, FPGA images need to be installed. Now the drivers are installed this can be called through the command:

```shell
sudo uhd_images_downloader
```

# Install `GNU-radio`:

