---
layout: post
title:  "Setup for ML"
date:   2018-03-22 20:00:00 -0700
categories: Research
---
These are my notes to set up the environment to use Tensorflow with the GPU on my machine. This post applies to the following configuration:
- Architecture x86_64 Ubuntu 16.04 LTS
- GeForce GTX 1080 Ti
- NVIDIA driver 384.111
- CUDA 9.0
- cuDNN v7.0.5 (Dec 5, 2017), for CUDA 9.0
- Python 3.5
- Tensorflow 1.6

This document describes the steps to:
- [Install Ubuntu 16-04](#install-ubuntu-16-04)
- [Check CUDA pre-requisites](#check-cuda-pre-requisites)
- [Install CUDA Toolkit](#install-cuda-toolkit)
- [Install CUDA runtime and examples](#install-cuda-runtime-and-examples)
- [Set paths in terminal startup file](#set-paths-in-terminal-startup-file)
- [Test CUDA installation](#test-cuda-installation)
- [Install python packages and tensorflow-gpu](#install-python-packages-and-tensorflow-gpu)
- [Test tensorflow installation](#test-tensorflow-installation)
- [Set terminal theme](#set-terminal-theme)

I will be following steps listed on the [Tensorflow installation page][install-tensorflow] for the CUDA and tensorflow-gpu installations. This post is motivated noticing that 1. the commands listed on the Tensorflow page need to be modified for the version of graphics drivers I'm working with and 2. the instructions relevant for me are scattered all over the page. Here I list all the steps I performed to achieve the final working configuration, in the spirit of [this blog post][config-outline]. 

#### <a href="#install-ubuntu-16-04">Install Ubuntu 16-04</a>

1. Issue: Display went blank after the initial BIOS screen. <br>
Fix: 
- Choose Legacy booting option from the BIOS options
- Press down arrow key while booting, and set the `nomodeset` option. (Source: [StackExchange] [nomodeset-fix]).

2. Issue: Booting into freshly installed OS led to a blank screen after the initial BIOS screen. Occasionally would see garbled text, or error messages. A common message was `error parsing pcc subspaces from pcct after nomodeset`. <br>
Fix:
- Holding the right shift key after the boot screen (supposed to load the grub menu) seemed to work. Not sure what actually happened.

3. The freshly installed OS had a 2-5 second latency even for keyboard input. 
Fix: 
 - Choose the NVIDIA driver from `Applications->Additional Drivers`, and restart. Computer then boots smoothly into the OS.

Notes: NVIDIA version 384.111 is the latest supported driver on the official Ubuntu package archive - and is the one that works for the remainder of the setup. If the default nouveau driver was correclty replaced through `Additional Drivers`, the following command will produce an empty result:
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

A related command (if necessary) to have apt-get reinstall the linux-headers package for current kernel version.
{% highlight bash %}
sudo apt-get --reinstall install linux-headers-`uname -r`
{% endhighlight %}

#### <a href="#install-cuda-toolkit">Install CUDA toolkit</a>
Following the instructions listed here on the [NVIDIA page][install-cuda-toolkit]. 

{% highlight bash %}
#Download the file manually if the next command doesn't work
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

#### <a href="#install-cuda-runtime-and-examples">Install CUDA runtime and examples</a>
Tensorflow uses the graphics card through specific CUDA libraries. Counterintuitively, the archive of releases is available only on following links to download the latest version. You will have to create/log into an NVIDIA account to download these. Login and download the following .deb files for `cuDNN v7.0.5 (Dec 5, 2017), for CUDA 9.0` from [this link][download-cuDNN].
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
This sets the correct paths in bash that will be used by tensorflow:
Note: The `subl` command below refers to the text editor I use (sublime text). 
{% highlight bash %}
subl ~/.bashrc
#Added as per CUDA 9.0 post installation actions
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64\ ${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
/usr/bin/nvidia-persistenced --verbose

#From Tensorflow install instructions
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
pip3 install --upgrade pydot graphviz
{% endhighlight %}

#### <a href="#test-tensorflow-installation">Test tensorflow installation</a>
Start an IPython console or Jupyter notebook. Things will likely break at the import statement in case something went wrong:
{% highlight python %}
import tensorflow as tf
hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
print(sess.run(hello))
{% endhighlight %}

From another [StackExchange post][check-tensorflow-gpu]: A check for whether the GPU was invoked:
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
I am not too fond of the default ubuntu terminal colors, instead I use the Smycks theme from github user [Mayccoll][term-theme]:
{% highlight bash %}
sudo apt-get install dconf-cli
wget -O gogh https://git.io/vQgMr && chmod +x gogh && ./gogh && rm gogh
{% endhighlight %}

[nomodeset-fix]: https://askubuntu.com/questions/162075/my-computer-boots-to-a-black-screen-what-options-do-i-have-to-fix-it/162076#162076
[config-outline]: http://www.python36.com/install-tensorflow141-gpu/
[install-tensorflow]: https://www.tensorflow.org/install/install_linux
[install-cuda-toolkit]: http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html
[download-cuDNN]: https://developer.nvidia.com/rdp/cudnn-download
[check-tensorflow-gpu]: https://stackoverflow.com/questions/38009682/how-to-tell-if-tensorflow-is-using-gpu-acceleration-from-inside-python-shell 
[term-theme]: https://github.com/Mayccoll/Gogh