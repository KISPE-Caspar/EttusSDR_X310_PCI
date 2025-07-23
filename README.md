# Ettus X310 PCI
This is a guide on how to interface the Ettus X310 SDR over PCI to a computer to allow for higher transfer speeds than conventional Ethernet communication.

![20250723_141332](https://github.com/user-attachments/assets/8d5df17c-d2ce-450b-85a0-76698ad5d22e)

This is NOT a guide on proper useage of connecting the SDR to the computer using the cable.

### References:

- https://kb.ettus.com/Building_and_Installing_the_USRP_Open-Source_Toolchain_(UHD_and_GNU_Radio)_on_Linux
- https://files.ettus.com/manual/page_ni_rio_kernel.html

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

## Install `GNU-radio`:

After the `uhd` drivers have been installed, then you can install and build `GNU-radio` from source. This is as it has dependencies that rely on the `uhd` drivers. This is a very similar process as building the other package.

Change back to `/workarea`:

```shell
cd $HOME
cd workarea
```

Clone the `GNU-radio` repository:

```shell
git clone --recursive https://github.com/gnuradio/gnuradio
```

And switch to `/gnuradio`:

```shell
cd gnuradio
```

Checkout the branch:

```shell
git checkout v3.7.13.4
```

Update the submodules:

```shell
git submodule update --init --recursive
```

Make and build `GNU-radio`:

```shell
mkdir build
cd build
cmake ../
make -j 4
```

You can test to see if it built correctly, however this might take a long time to complete (skipable):

```shell
make test
```

Then install `GNU-radio`:

```shell
sudo make install -j 4
```

Update system shared cache:

```shell
sudo ldconfig
```

You can check `GNU-radio` is installed correctly through running some basic commands which should return their respective information:

```shell
gnuradio-config-info --version
gnuradio-config-info --prefix
gnuradio-config-info --enabled-components
```

## Installing Linux NI PCI drivers:

**Download the `2020` version of the driver package at the link below:**

[NI Linux Device Drivers](https://www.ni.com/en-us/support/downloads/drivers/download.ni-linux-device-drivers.html)

Extract the downloaded zip file and install the `.deb` file inside:

```shell
sudo dpkg -i {ni drivers repository}.deb
```

Update `apt`:

```shell
sudo apt update
```

Install the kernal headers, these may be already preinstalled:

```shell
sudo apt install linux-headers-$(uname -r)
```

Now install the RIO PCI driver:

```shell
sudo apt install ni-usrp-rio
```

Build the kernal modules:

```shell
dkms autoinstall
```

The drivers will now be installed and you should be able to discover your SDR...

Check by running the following command below:

```shell
uhd_find_devices
```

# Useage with SDR software:

When using the SDR with software, you may get a warning that your version number of the FPGA is not the number expected.

To fix this, you can flash the X310 with new firmware, reccomended through the ethernet cable.

Plug an ethernet cable between the computer and SDR and power it on.

You will need to change your internet settings to have a local address of `192.168.10.1` and subnet mask of `255.255.255.0`.

Then after checking `uhd_find_devices`, run the following to flash `uhd` firmware compatible with your installed version:

```shell
/usr/bin/uhd_image_loader --args="type=x300,addr=192.168.10.2"
```
