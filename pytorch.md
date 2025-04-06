# How to install torch in *OKdo Nano C100 Developer Kit*.

WORK IN PROGRESS...


**Disclaimer: Follow these notes at your own risk. I'm not responsible for anything you run in your devices. It's up to you!**

I currently have JetPack 4.6.6 installed becasue it'is the image that OKdo provides. If we look at the [Pytorch for Jetson](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048) page, we'll see that the highest available pytorch for JetPack 4 is for JetPack 4.4. I tried to install it, but it didn't work. Unfortunately, I need to follow the last resort: build from source.

*We have to execute this process on the Nano*

First of all, we have to check our JetPack version:

    nano:~$ cat /etc/nv_tegra_release 
    # R32 (release), REVISION: 7.6, GCID: 38171779, BOARD: t210ref, EABI: aarch64, DATE: Tue Nov  5 07:46:14 UTC 2024

`R32 (realease)` and `REVISION: 7.6` correspond to `L4T 32.7.6`. As you can see [here](https://developer.nvidia.com/embedded/jetpack-archive), JetPack 4.6.6 is L4T 32.7.6,

We are going to build [Torch 1.10.0](https://github.com/pytorch/pytorch/tree/v1.10.0). I choose this version because is the latest one that [seems](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048) tested on the JetPack we have. The requirements for building are *"Python 3.6.2 or later and a C++14 compiler"*. To check that:
    
    nano:~$ python3 --version
    Python 3.6.9
    nano:~$ g++ --version 
    g++ (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0

The python version is correct, and the g++ too, because C++14 has been [avaiable since g++ 6.1](https://gcc.gnu.org/projects/cxx-status.html#cxx14) 

To download the source, we'll use Git (if it's not present, run `sudo apt install git`): 

    git clone --recursive --branch v1.10.0 http://github.com/pytorch/pytorch

We move to the `torch` directory. All the next steps assume we're here:

    cd torch

To apply the patch, as stated the *Apply Patch* section of [this page](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048):

    wget https://gist.githubusercontent.com/dusty-nv/ce51796085178e1f38e3c6a1663a93a1/raw/4f1a0f948150c91f877aa38075835df748c81fe5/pytorch-1.10-jetpack-4.5.1.patch -O pytorch-1.10-jetpack-4.5.1.patch
    patch -p1 < pytorch-1.10-jetpack-4.5.1.patch

Install some essential packages:

    sudo apt install build-essential python3-pip python3-venv 

Create and activate a Python `venv`:

    python3 -m venv venv
    source venv/bin/activate

Install some Python packages:

    pip3 uninstall Cython
    pip3 install 'Cython<3' # To avoid problemes with numpy install
    pip3 uninstall setuptools
    pip3 install setuptools==59.5.0 # To avoid AttributeError: module distutils has no attribute version
    pip3 install -r requirements.txt
    pip3 install scikit-build

Again, as [Build from Source](https://forums.developer.nvidia.com/t/pytorch-for-jetson/72048) requires, we add some swap space if it hasn't been done for any other reason:

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

    time python3 setup.py bdist_wheel

It will take some hours to finish. In my case, it took xx hours

...

To check if it works:

    python3 -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"

Exit the Python `venv`:

    deactivate

Remove the swap space added (if any):

    sudo swapoff /swapfile_tmp
    swapon --show
    sudo rm /swapfile_tmp



