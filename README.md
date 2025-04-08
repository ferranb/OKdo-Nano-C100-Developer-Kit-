# OKdo Nano C100 Developer Kit 

**Disclaimer: Follow these notes at your own risk. I'm not responsible for anything you run in your devices. It's up to you!**

This page is a bunch of notes from the process I've done to set up the [OkDo C100 NVIDIA Jetson Nano 4GB Development Kit](https://www.kubii.com/en/development-kit/3882-c100-nvidia-jetson-nano-4gb-development-kit-3272496313705.html). It's an available equivalent to the discontinued [NVIDIA Jetson Nano Developer Kit](https://developer.nvidia.com/embedded/learn/get-started-jetson-nano-devkit) product. The replacement product, [NVIDIA Jetson Nano Super Developr Kit](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/nano-super-developer-kit/), is [usually out of stock](https://forums.developer.nvidia.com/t/jetson-orin-nano-super-availability/325782). 

The problem is that *OKdo Nano C100 Developer Kit* is an extremely poorly documented device. The [okdo website](https://www.okdo.com/) doesn't work. I really don't know if the company is still operating, if they just don't care about the website, or if they were adquired by another company.

If you need more detail on any point and I know it, please, don't hesitate to contact me. These notes are intentionally short because they're enought for me :-)

# Hardware

* [Radxa NX4 IO Board](https://radxa.com/products/io-board/nx5-io-board/). The one I own has no *Markrom key* or *power button*.
* [NVIDIA Jetson Nano](https://developer.nvidia.com/buy-jetson?product=jetson_nano). It has a Tegra processor and the 16GB eMMC storage.

I guess that buying the [board](https://es.aliexpress.com/item/1005007130498108.html?gatewayAdapt=glo2esp)
, the [card](https://www.siliconhighwaydirect.com/ProductDetails.asp?ProductCode=900-13448-0020-000) and the 
[heatsink](https://auvidea.eu/product/heatsink-70752/) would word too. I haven't tried it!

# Powering

You can use the jack for 5V and 4A, or the microUSB for 5V and 2A. Next to the Jack, there is a jumper that determines 
the power source. With the jumper in place, the jack power will be drawn from the jack. Without the jumper, power will come from the microUSB interface.

Because the GPU's power requirements, it's strongly recommended (and probably mandatory) to use the jack.

# Booting from eMMC

Just unboxing the device, it's possible to boot directly from the eMMC. You'll find an Ubuntu 18.04.6 LTS based login screen, using `nano` as both the username and password.

Check the version with:

    nano@nano:~$ lsb_release -rcd
    Description:	Ubuntu 18.04.6 LTS
    Release:	18.04
    Codename:	bionic

It contains NVIDIA L4T ([Linux for Tegra](https://developer.nvidia.com/embedded/linux-tegra-r321)) for Tegra, a NVIDIA Linux distribution for Tegra devices. 

Check the version with:

    nano@nano:~$$ cat /etc/nv_tegra_release
    # R32 (release), REVISION: 7.2, GCID: 30192233, BOARD: t210ref, EABI: aarch64, DATE: Wed Apr 20 21:34:48 UTC 2022

# MicroUSB port

If you have a MicroUSB cable, you can plug it in and connect it to your computer. Your computer will detect a
new USB hub with the following components:

* A modem probably at `/dev/ttyACM0`
* A volume named LT4-README with some files containing details.
* Two network interfaces.

When the eMMC Ubuntu is running you can connect through `SSH` using `nano` as both username and password by running:

    ssh nano@192.168.55.1

To connect to the serial port, use a terminal program like `screen` or `minicom` and connect to the appropriate `/dev/tty*` device. In my case, it is:

    sudo screen /dev/ttyACM0 115200
    
Please note that the microUSB connection depends on linux running. If the linux doesn't boot, you won't have access. If that happens, you can boot from an SD card and recover it.

# Boot from SD card

Burn the image from OKdo, available from [different](https://auto.designspark.info/okdo_images/c100.img.xz) [sources](https://www.rs-online.com/designspark/okdo-software-and-downloads-hub). The file name is `c100.img`. You just need to burn into the SD card using you preferred imaging tool. After that, insert it into the device.

It's very important to connect an HDMI screen and a keyboard to see what's happening and to follow the steps to configure Ubuntu.

# Boot from USB drive

*Option 1. Through SD card*

I've tried different approaches that didn't work. Finally, I used the SD card bootloader as a gateway to boot from USB SSD by modifying `/boot/extlinux/extlinux.conf`. You just need to change the line saying `root=/dev/mmcblk0p1` to `root=/dev/sda1` or, better, `root=PARTUUID='xxxx'`. You can obtain the PARTUUID with `blkid`. It didn't work for me with `root=UUID='xxxx'`. 

This solution its enought for me. It boots fast, but I've sacrificed an SD card just solve this. Its a reasoble price to pay.

*Option 2: Creating a new boot image (TODO)*

This is a trickier option that I haven't had time to try. Theoricaly, you need to create a new image which load the differents usb drivers to boot from an USB drive. Also you can take the advantage that the `c100.img` is the same version that the one in the eMMC.

# About the system updates

**WARNING**. Don't run `sudo apt update && sudo apt upgrade` because it can break some things. It's better to leave everything unchanged to avoid breaking compatibility with CUDA, cuDNN, or other NVIDIA components.

# Checking what's installed

You can check your system configuration by installing and running:

    sudo pip3 install jetson-stats
    sudo jtop

Press `7` to see the `info` page.

We'll also check other ways to obtain that information, as it could be useful in some scenarios.

To check our JetPack version:

    nano:~$ cat /etc/nv_tegra_release 
    # R32 (release), REVISION: 7.6, GCID: 38171779, BOARD: t210ref, EABI: aarch64, DATE: Tue Nov  5 07:46:14 UTC 2024

`R32 (realease)` and `REVISION: 7.6` correspond to `L4T 32.7.6`. As you can see [here](https://developer.nvidia.com/embedded/jetpack-archive), JetPack 4.6.6 is L4T 32.7.6. But... `jtop` says to me 4.6.1. Probably, `jtop` checks the major release, while the contents of `nv_tegra_release` show the latest update.

Python 3 version (In my case, it's 3.6.9):

    python3 --version

Ubuntu release (In my case, Ubuntu 18.04.6 LTS):

    lsb_release -d

CUDA version (In my case, [10.2.300](https://developer.nvidia.com/cuda-10.2-download-archive). [Here are the docs](https://docs.nvidia.com/cuda/archive/10.2/)):

    nvcc --version
    cat /usr/local/cuda/version.txt

cuDNN version (In my case, [8.2.1.32](https://developer.nvidia.com/rdp/cudnn-archive)):

    apt list --installed | grep cudnn

As a recap, my Jetson has:

    Ubuntu 18.04.6 LTS
    JetPack 4.6.6
    Python 3.6.9
    CUDA 10.2.300
    cuDNN 8.2.1.32

# Installing PyTorch 1.10.0

We are going to install [PyTorch 1.10.0](https://github.com/pytorch/pytorch/tree/v1.10.0) because is the one that [NVIDIA recommends](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048):

    wget https://nvidia.box.com/shared/static/fjtbno0vpo676a25cgvuqc1wty0fkkg6.whl -O torch-1.10.0-cp36-cp36m-linux_aarch64.whl
    sudo apt-get install python3-pip libopenblas-base libopenmpi-dev libomp-dev
    pip3 install 'Cython<3'
    pip3 install numpy torch-1.10.0-cp36-cp36m-linux_aarch64.whl

After the install, you can check it:

    python3 <<_
    import torch
    print(torch.__version__)
    print('CUDA available:', str(torch.cuda.is_available()))
    print('cuDNN version:', str(torch.backends.cudnn.version()))
    print('GPUs:', str(torch.cuda.device_count()))
    for i in range(torch.cuda.device_count()):
        print('GPU',i, torch.cuda.get_device_properties(i))
    a = torch.cuda.FloatTensor(2).zero_()
    print('Tensor a = ' + str(a))
    b = torch.randn(2).cuda()
    print('Tensor b = ' + str(b))
    c = a + b
    print('Tensor c = ' + str(c))
    _

It returns this to me (of course, since it randomizes, the tensor numbers will be different for you):
    
    1.10.0
    CUDA available: True
    cuDNN version: 8201
    GPUs: 1
    GPU 0 _CudaDeviceProperties(name='NVIDIA Tegra X1', major=5, minor=3, total_memory=3956MB, multi_processor_count=1)
    Tensor a = tensor([0., 0.], device='cuda:0')
    Tensor b = tensor([-0.0969,  0.5188], device='cuda:0')
    Tensor c = tensor([-0.0969,  0.5188], device='cuda:0')

To install [torchvision](https://pytorch.org/vision/stable/index.html), we select 0.11.1 as [NVIDIA suggests](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048):

    pip3 install torchvision==0.11.1

To install [torchtext](https://github.com/pytorch/text), 0.11.0:

    pip3 install torchtext==0.11.0 
    
To install [torchaudio](https://pytorch.org/audio/main/installation.html#compatibility-matrix), 0.10.0:

    pip3 install torchaudio==0.10.0

# Install Python 3.12.9

Follow these steps to install Python [3.12.9](https://github.com/python/cpython/tree/v3.12.9)

Install the [build dependencies](https://devguide.python.org/getting-started/setup-building/#build-dependencies):

    sudo apt install pkg-config build-essential gdb lcov pkg-config \
      libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \
      libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \
      lzma lzma-dev tk-dev uuid-dev zlib1g-dev libmpdec-dev

Then, we download it, build it, and install it without overwriting the current Python 3 installation:
    
    wget https://www.python.org/ftp/python/3.12.9/Python-3.12.9.tgz
    tar -xvf Python-3.12.9.tgz
    cd Python-3.12.9
    ./configure --enable-optimizations
    time make -j$(nproc)  # 32 minutes
    sudo make altinstall  # Don't overwrite current python3
    sudo pip3.12 install --upgrade pip

To test the installation:

    python3.12 --version # 3.12.9
    ls /usr/local/bin/pip3.12
    python3.12 -m venv --help

# Build and install PyTorch 1.10.0 for Python 3.12.9

*DRAFT WIP*

To install PyTorch we need to build from source. As we have CUDA 10.2.300, the higher compatible release is [1.12.1](https://pytorch.org/get-started/previous-versions/#v1121). But... the improvements versus 1.10.0 are not signitificatives (IMHO) and would require some other builds because 18.04 has older commands (for instance, `cmake`).

Because that, I'll install 1.10.0.

We ensure `PATH` and `LD_LIBRARY_PATH` are correct:

    [[ ":$PATH:" != *":/usr/local/cuda-10.2/bin:"* ]] && export PATH="$PATH:/usr/local/cuda-10.2/bin" ; echo $PATH
    [[ ":$LD_LIBRARY_PATH:" != *":/usr/local/cuda/lib64:"* ]] && export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64" ; echo $LD_LIBRARY_PATH
    
To download the source, we'll use Git (if it's not present, please, run `sudo apt install git`): 

    git clone --recursive --branch v1.10.0 http://github.com/pytorch/pytorch

We move to the `torch` directory. All the next steps assume we're here:

    cd pytorch

Apply the patch, as stated the *Apply Patch* section of [this page](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048):

    wget https://gist.githubusercontent.com/dusty-nv/ce51796085178e1f38e3c6a1663a93a1/raw/4f1a0f948150c91f877aa38075835df748c81fe5/pytorch-1.10-jetpack-4.5.1.patch -O pytorch-1.10-jetpack-4.5.1.patch
    patch -p1 < pytorch-1.10-jetpack-4.5.1.patch

We apply the following patch to avoid some issues with `distutils:

    patch -p1 <_
    --- cmake_new.py	
    +++ tools/setup_helpers/cmake.py
    @@ -116,6 +116,7 @@
             "Returns cmake command."
     
             cmake_command = 'cmake'
    +        return cmake_command
             if IS_WINDOWS:
                 return cmake_command
             cmake3 = which('cmake3')
    _

Install some required packages:

    sudo apt install build-essential

Create and activate a Python `venv`:

    python3.12 -m venv venv
    source venv/bin/activate

Install some Python packages:

    pip3.12 uninstall Cython
    pip3.12 install 'Cython<3' # To avoid problemes with numpy install
    pip3.12 uninstall setuptools
    pip3.12 install -r requirements.txt
    pip3.12 install scikit-build

As [Build from Source](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048) requires, we add some swap space if it hasn't been done for any other reason:

    sudo fallocate -l 4G /swapfile_tmp
    sudo chmod 600 /swapfile_tmp
    sudo mkswap /swapfile_tmp
    sudo swapon /swapfile_tmp
    swapon --show

We set some enviroment variables, again, because the [Build from Source](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048) section says so:

    export USE_NCCL=0
    export USE_DISTRIBUTED=0
    export USE_QNNPACK=0
    export USE_PYTORCH_QNNPACK=0
    export TORCH_CUDA_ARCH_LIST="5.3"
    export PYTORCH_BUILD_VERSION=1.10.0
    export PYTORCH_BUILD_NUMBER=1

Note that `TORCH_CUDA_ARCH_LIST` is set to `5.3`. That's because [here](https://developer.nvidia.com/cuda-gpus) says that the _Compute Capability_ of our Nano is 5.3.

Take a deep breath. Pray. Cross your fingers. Here we go!

    nohup python3.12 setup.py bdist_wheel &
    tail -f nohup.out

It will take about 12 hours to finish... After buid completes without errors, you can install torch:

    python3 setup.py install

To check if it works:

    nano$python3 -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
    nano$python3 -c "import torch; print(torch.backends.cuda.is_built())"

Exit the Python `venv`:

    deactivate

Remove the swap space added (if any):

    sudo swapoff /swapfile_tmp
    swapon --show
    sudo rm /swapfile_tmp


# Next Steps

*TODO*: Test Docker setups, etc.

# References

* [Ubuntu snippets](https://github.com/ferranb/ubuntu_snippets)
* https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/index.html#page/Tegra%2520Linux%2520Driver%2520Package%2520Development%2520Guide%2Fbootflow_jetson_nano.html%23%5B/url%5D
* [OKDO official images mirror from RS-Online](https://www.rs-online.com/designspark/okdo-software-and-downloads-hub)
* [OKDO specifications](https://docs.rs-online.com/9149/A700000009238033.pdf)
* [Customer image for SD](https://github.com/LetsOKdo/c100-bootupd)
* https://gist.github.com/KEINOS/47b0a89f32c77adbf3887b2469e4ce7c
* https://iothonpo.com/nano-c100-setup/
