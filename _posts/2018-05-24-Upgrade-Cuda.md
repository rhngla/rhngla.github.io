---
layout: post
title:  "Upgrading to CUDA 9.1"
date:   2018-05-24 08:00:00 -0700
categories: Research
---

The first half of this post lists steps to upgrade to CUDA 9.1. The second half documents a failed attempt to install drivers from the official NVIDIA archive.

Configuration summary:
- Ubuntu 16.04 with NVIDIA GeForce GTX 1080 Ti
- **NVIDIA driver: 384.111 --> 390.59**
- **CUDA: 9.0 --> 9.1**
- **cuDNN: v7.0.5 for CUDA 9.0 --> v7.0.5 for CUDA 9.1**
- Python 3.5
- Tensorflow-gpu 1.8

This document describes steps to:
1. [Upgrade NVIDIA driver](#upgrade-nvidia-driver)
2. [Install CUDA 9.1](#install-cuda-9-1)
3. [Install CUDA libraries (cuDNN)](#install-cuda-libraries)

#### <a href="#upgrade-nvidia-driver">Upgrade NVIDIA driver</a>
CUDA 9.1 requires the NVIDIA-390 driver. The latest version available on the [official Ubuntu 16.04 repository](https://packages.ubuntu.com/xenial/allpackages) is only 384. I was [unable to install](#failed-attempt-nvidia-driver-installation) version 390 using the .run file from [NVIDIA](http://www.nvidia.com/object/unix.html). Drivers provided by the [graphics-drivers PPA](https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa), (neither Ubuntu nor NVIDIA officially support this) worked for me.

{% highlight bash %}
sudo add-apt-repository ppa:graphics-drivers/ppa # Add PPA that provides NVIDIA drivers
sudo apt-get update # Refresh list of available updates
apt list --upgradable # Should include multiple NVIDIA drivers from the PPA we added.
sudo apt-get upgrade # Apply all available updates. (will install multiple NVIDIA drivers, which is not a problem)
{% endhighlight %}

Before you switch drivers, open the grub file and set `nomodeset` as a default. Excerpt from a [forum thread](https://ubuntuforums.org/showthread.php?t=1613132):
> "...the nomodeset parameter instructs the kernel to not load video drivers and use BIOS modes instead until X is loaded..." 

I had previously encountered a blank screen immediately after the boot screen because the driver was not being loaded correctly (see my notes regarding the [failed driver installation](#failed-attempt-nvidia-driver-installation)).

{% highlight bash %}
#Open /etc/default/grub in text editor
sudo subl /etc/default/grub

#Edit the following lines to add the nomodeset option, save and quit.
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
GRUB_CMDLINE_LINUX="nomodeset"
{% endhighlight %}

Now switch graphics drivers: open `Additional Drivers` settings under `Applications` using the Unity Dash. Switch to the `nvidia-390` driver, apply changes (takes some time to complete this operation), followed by a system restart.

#### <a href="#install-cuda-9-1">Install CUDA 9-1</a>
Download the base installer from the [NVIDIA archive](https://developer.nvidia.com/cuda-91-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1604&target_type=deblocal). The update patches don't need to be downloaded manually - we will apply them automatically using `apt`. 

{% highlight bash %}
sudo dpkg -i cuda-repo-ubuntu1604-9-1-local_9.1.85-1_amd64.deb
sudo apt-key add /var/cuda-repo-9-1-local/7fa2af80.pub
sudo apt-get update # Get list of updates 
sudo apt-get install cuda # Applies updates to CUDA
{% endhighlight %}

Now update paths in the .bashrc file:
{% highlight bash %}
subl ~/.bashrc # Open bashrc in a text editor

#Add CUDA 9.1 paths to bashrc
export PATH=/usr/local/cuda-9.1/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.1/lib64\ ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
{% endhighlight %}

Close and restart the terminal to reflect changes made to the .bashrc file. 
{% highlight bash %}
nvcc -V # Should show: Cuda compilation tools, release 9.1, V9.1.85
{% endhighlight %}

#### <a href="#install-cuda-libraries">Install CUDA libraries</a>
Install the correct version of cuDNN for CUDA 9.1 on Ubuntu 16.04: `cuDNN v7.0.5 (Dec 5, 2017), for CUDA 9.0`. Download files corresponding to the Runtime library, and Developer library and Code samples. Navigate to folder with these files and install them:

{% highlight bash %}
sudo dpkg -i libcudnn7_7.0.5.15-1+cuda9.1_amd64.deb 
sudo dpkg -i libcudnn7-dev_7.0.5.15-1+cuda9.1_amd64.deb 
sudo dpkg -i libcudnn7-doc_7.0.5.15-1+cuda9.1_amd64.deb 
{% endhighlight %}
 
Test the installation:
{% highlight bash %}
cp -r /usr/src/cudnn_samples_v7/ $HOME
cd  $HOME/cudnn_samples_v7/mnistCUDNN
make clean && make
./mnistCUDNN
{% endhighlight %}

#### <a href="#failed-attempt-nvidia-driver-installation">Failed attempt NVIDIA driver installation</a>

I downloaded the `NVIDIA-Linux-x86_64-390.59.run` file from [http://www.nvidia.com/object/unix.html](http://www.nvidia.com/object/unix.html), and attempted the installation:
{% highlight bash %}
sudo apt-get purge nvidia* # Removes existing NVIDIA drivers
cd ~/Downloads/ 
chmod +x NVIDIA-Linux-x86_64-390.59.run # Sets permissions to allow the .run file to be executed
sudo ./NVIDIA-Linux-x86_64-390.59.run... #Executes the file
{% endhighlight %}

This resulted in an error message `X server still running`. The full error message was similar to the one in this [forum thread](https://askubuntu.com/questions/149206/how-to-install-nvidia-run)

The suggested workaround is to log into a tty, and stop the X server. I used `Ctrl+Alt+F1` to start the tty.
{% highlight bash %}
sudo service lightdm stop # Stops the X server
{% endhighlight %}

This resulted in  blank screen. On doing a hard reset, the display would not show anything past the boot screen (even before the grub menu).

```
Note: In hindsight, I think I could have re-entered the GUI with Ctrl+Alt+F7.
```

I knew the graphics driver was causing this issue. [Previously](/research/2018/03/22/Setup-for-ML), I had been able to set `nomodeset` by holding down the `shift` key after the boot screen, and log into the GUI. However, the solution did not work this time.

#### Fix to set nomodeset:
 - Use the `Try Ubuntu without installing` option on a Ubuntu installation USB.
 - Modify grub files of the installed operating system to use `nomodeset` as default
 [Reference:](https://askubuntu.com/questions/38780/how-do-i-set-nomodeset-after-ive-already-installed-ubuntu)

{% highlight bash %}
#Create temporary directory to mount hard disc with the existing installation
cd /mnt 
mkdir old_install 

#My existing installation was on /dev/sda1
sudo mount /dev/sda1 /mnt/old_install 

#Open /etc/default/grub in text editor
sudo vi /mnt/old_install/etc/default/grub

#Edit the following lines to add the nomodeset option, save and quit.
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
GRUB_CMDLINE_LINUX="nomodeset"
{% endhighlight %}

I attempted installing the NVIDIA driver once again, which led to blank screen once again, and I abandoned this way of installing the driver. 




  