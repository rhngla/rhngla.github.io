---
layout: post
title:  "Setup for ML"
date:   2018-03-22 08:00:00 -0700
categories: Research
---
These are my notes to set up a new computer to use tensorflow-gpu. The final system configuration:
- x86_64 Ubuntu 16.04
- GeForce GTX 1080 Ti
- NVIDIA driver 384.111
- CUDA 9.0
- cuDNN v7.0.5 for CUDA 9.0
- Python 3.5
- Tensorflow 1.6

This document describes the steps to:
1. [Install Ubuntu 16-04](#install-ubuntu-16-04)
2. [Check CUDA pre-requisites](#check-cuda-pre-requisites)
3. [Install CUDA Toolkit](#install-cuda-toolkit)
4. [Install CUDA Libraries](#install-cuda-libraries)
5. [Set paths in terminal startup file](#set-paths-in-terminal-startup-file)
6. [Test CUDA installation](#test-cuda-installation)
7. [Install python packages and tensorflow-gpu](#install-python-packages-and-tensorflow-gpu)
8. [Test tensorflow installation](#test-tensorflow-installation)
9. [Set terminal theme](#set-terminal-theme)

Commands to install CUDA and tensorflow-gpu are sourced from the [Tensorflow installation page][install-tensorflow], with minor edits to work with my configuration. 

#### <a href="#install-ubuntu-16-04">Install Ubuntu 16-04</a>

1. Issue: Display went blank after the initial BIOS screen. <br>
Fix: 
- Choose Legacy booting option from the BIOS options
- Press the down arrow key (or the shift key) while booting, and set the `nomodeset` option. (Source: [StackExchange](https://askubuntu.com/questions/162075/my-computer-boots-to-a-black-screen-what-options-do-i-have-to-fix-it/162076#162076)).

2. Issue: Booting into newly installed OS led to a blank screen after the initial BIOS screen. Occasionally would show garbled text, or error messages. A common message was `error parsing pcc subspaces from pcct after nomodeset`. <br>
Fix:
- Holding the right shift key after the boot screen (supposed to load the grub menu) seemed to work. Not sure what actually happened.

3. The freshly installed OS had a 2-5 second latency even for keyboard input. 
Fix: 
 - Choose the NVIDIA driver from `Applications->Additional Drivers`, and restart. Computer then boots smoothly into the OS.

Notes: NVIDIA version 384.111 is the latest supported driver on the official Ubuntu package archive as of 3/22/2018. If the default nouveau driver is correctly replaced through `Additional Drivers`, the following command will produce an empty result:
{% highlight bash %}
lsmod | grep nouveau
{% endhighlight %}

#### <a href="#check-cuda-pre-requisites">Check CUDA pre-requisites</a>
Check versions of OS, headers, GCC, and GNU C Library (glibc):
{% highlight bash %}
uname -m && cat /etc/*release
uname -r
gcc --version
ldd --version
{% endhighlight %}
Mine were:
- x86_64 Ubuntu 16.04.04 LTS
- Linux header version 4.13.0
- GCC version 5.4.0
- GLIBC version 2.23

A related command (if necessary) to have apt-get reinstall the linux-headers package for current kernel version:
{% highlight bash %}
sudo apt-get --reinstall install linux-headers-`uname -r`
{% endhighlight %}

#### <a href="#install-cuda-toolkit">Install CUDA toolkit</a>
Following the instructions listed on the [NVIDIA page](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html): 

{% highlight bash %}
#Download the file manually if the wget command fails to retrieve the .deb file
wget -q https://developer.nvidia.com/compute/cuda/9.0/Prod/local_installers/cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64-deb
#Change name to have the .deb extension
mv cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64-deb cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64.deb
#Install the package
sudo dpkg -i cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64.deb
sudo apt-key add /var/cuda-repo-9-0-local/7fa2af80.pub
sudo apt-get update
sudo apt-get install cuda
{% endhighlight %}

Setting paths based on CUDA post installation instructions for current bash session:
{% highlight bash %}
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64\ ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
/usr/bin/nvidia-persistenced --verbose
{% endhighlight %}
We will add the above to the `~/.bashrc` file later (see instructions in [Terminal startup file](#set-paths-in-terminal-startup-file) section).

Next, install command line toolkit as per instructions on the [Tensorflow installation page][install-tensorflow].
{% highlight bash %}
sudo apt-get install cuda-command-line-tools-9-0
{% endhighlight %}

Check version of CUDA compiler driver installed; the expected value is `release 9.0`.
{% highlight bash %}
nvcc -V
{% endhighlight %}

#### <a href="#install-cuda-libraries">Install CUDA libraries</a>
Tensorflow uses the graphics card through specific CUDA libraries. The archive of releases can be found by following links to download the latest version. You may have to create an account/login to download the .deb files for `cuDNN v7.0.5 (Dec 5, 2017), for CUDA 9.0` from [https://developer.nvidia.com/rdp/cudnn-download](https://developer.nvidia.com/rdp/cudnn-download).
- cuDNN v7.0.5 Runtime Library for Ubuntu16.04 (Deb)
- cuDNN v7.0.5 Developer Library for Ubuntu16.04 (Deb)
- cuDNN v7.0.5 Code Samples and User Guide for Ubuntu16.04 (Deb)

Install them using:
{% highlight bash %}
sudo dpkg -i libcudnn7_7.0.5.15-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-dev_7.0.5.15-1+cuda9.0_amd64.deb
sudo dpkg -i libcudnn7-doc_7.0.5.15-1+cuda9.0_amd64.deb
{% endhighlight %}

#### <a href="#set-paths-in-terminal-startup-file">Set paths in terminal startup file</a>
Set paths in bash to point to the CUDA installation
{% highlight bash %}
#Open bashrc file in a text editor (subl is my sublime text install)
subl ~/.bashrc

#1. CUDA 9.0 post installation actions
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64\ ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
/usr/bin/nvidia-persistenced --verbose

#2. Tensorflow install instructions
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/extras/CUPTI/lib64
{% endhighlight %}

#### <a href="#test-cuda-installation">Test CUDA installation</a> 
{% highlight bash %}
cp -r /usr/src/cudnn_samples_v7/ $HOME
cd  $HOME/cudnn_samples_v7/mnistCUDNN
make clean && make
./mnistCUDNN
{% endhighlight %}

The expected output is `Test passed`.

#### <a href="#install-python-packages-and-tensorflow-gpu">Install python packages and tensorflow-gpu</a>
Python 3.5 is already installed with Ubuntu 16.04. I installed some packages that I will likely use in many projects.
{% highlight bash %}
pip3 install --upgrade numpy --user
pip3 install --upgrade scipy --user
pip3 install --upgrade matplotlib --user
pip3 install --upgrade ipython --user
pip3 install --upgrade jupyter --user
pip3 install --upgrade tensorflow-gpu --user
{% endhighlight %}

Some other useful packages
{% highlight bash %}
pip3 install --upgrade seaborn
pip3 install --upgrade keras
{% endhighlight %}

#### <a href="#test-tensorflow-installation">Test tensorflow installation</a>
Start an IPython console or Jupyter notebook. Things will likely break at the import statement in case something went wrong:
{% highlight python %}
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
{% endhighlight %}

From another [StackExchange post](https://stackoverflow.com/questions/38009682/how-to-tell-if-tensorflow-is-using-gpu-acceleration-from-inside-python-shell): A check for whether the GPU was invoked:
{% highlight python %}
import tensorflow as tf
with tf.device('/gpu:0'):
    a = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[2, 3], name='a')
    b = tf.constant([1.0, 2.0, 3.0, 4.0, 5.0, 6.0], shape=[3, 2], name='b')
    c = tf.matmul(a, b)
with tf.Session() as sess:
    print (sess.run(c))
{% endhighlight %}
Output `/gpu:0` indicates that the GPU was used. You may also see the name of the GPU listed in the output.

#### <a href="#set-terminal-theme">Set terminal theme</a>
I am not too fond of the default ubuntu terminal colors, instead I use the Smycks theme from github user [Mayccoll](https://github.com/Mayccoll/Gogh):
{% highlight bash %}
sudo apt-get install dconf-cli
wget -O gogh https://git.io/vQgMr && chmod +x gogh && ./gogh && rm gogh
{% endhighlight %}

[install-tensorflow]: https://www.tensorflow.org/install/install_linux