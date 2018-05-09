---
layout: post
title:  "Build FEniCS from source with Intel MKL on Ubuntu 18.04 LTS"
date:   2018-05-09 
author: Evan Cummings
categories: jekyll update
---

If you are working with a cluster system, or simply want to take advantage of Intel's [MKL][intel-mkl] linear-algebra libraries with your finite-element calculations using [FEniCS][fenics], you'll have to build some code from source.
First, define environment variables for the versions of each code we'll need to build FEniCS,

{% highlight bash %}
export PYTHON_VERSION=2.7.15;
export NUMPY_VERSION=1.13;
export SCIPY_VERSION=1.0;
export MATPLOTLIB_VERSION=2.2;
export MPI4PY_VERSION=3.0.0;
{% endhighlight %}

the directory we install source code into (`$SFT_DIR`), where MKL is installed (`$INTEL_DIR = $HOME/intel` by default), and where the Python source code will go (`$PYHON_DIR`):
  
{% highlight bash %}
export SFT_DIR=$HOME/software;
export INTEL_DIR=$HOME/intel;
export PREFIX=$HOME/local;
export PYTHON_DIR=${PREFIX}/python-${PYTHON_VERSION};
{% endhighlight %}

Next, load the 64-bit MKL libraries into the system path using Intel's `*vars.sh` script:

{% highlight bash %}
source ${INTEL_DIR}/mkl/bin/mklvars.sh intel64;
{% endhighlight %}

While not a neccessary requirement, let's build Python version 2.7.15 in a user directory.
In anticipation of the python installation to follow, let's add some environment variables to the path now:

{% highlight bash %}
export PATH=$PYTHON_DIR/bin:$PATH;
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PYTHON_DIR/lib;
{% endhighlight %}

Finally, it is convenient to save the number of threads to build the software with in a variable, such as

{% highlight bash %}
export BUILD_THREADS=4;
{% endhighlight %}

If you have access to `sudo`, install all the python dependencies the easy way (make sure to check box *Source code* in data sources) :

{% highlight bash %}
sudo apt-get install -y git build-essential libmpich-dev;
sudo apt-get build-dep -y python2.7;
{% endhighlight %}

OK, now compile Python with optimizations:

{% highlight bash %}
cd $SFT_DIR;
wget https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz;
tar -xzf Python-${PYTHON_VERSION}.tgz;
cd $SFT_DIR/Python-${PYTHON_VERSION};
./configure --enable-optimizations \
            --enable-shared \
            --with-system-expat \
            --with-system-ffi \
            --enable-ipv6 \
            --with-threads \
            --with-pydebug \
            --with-lto \
            --prefix=${PYTHON_DIR};
make -j ${BUILD_THREADS};
make install;
{% endhighlight %}

Here's the entire function:

{% highlight bash %}
function install_python_from_source_gnu()
{
  export PYTHON_VERSION=2.7.15;
  export NUMPY_VERSION=1.13;
  export SCIPY_VERSION=1.0;
  export MATPLOTLIB_VERSION=2.2;
  export MPI4PY_VERSION=3.0.0;
  
  export INTEL_DIR=$HOME/intel;

  # make the packages available to paths :
  source ${INTEL_DIR}/mkl/bin/mklvars.sh intel64;

  export PREFIX=$HOME/local;
  export SFT_DIR=$HOME/software;
  export PYTHON_DIR=${PREFIX}/python-${PYTHON_VERSION};

  # add the soon to be populated python directory to path :  
  export PATH=$PYTHON_DIR/bin:$PATH;
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$PYTHON_DIR/lib;

  export BUILD_THREADS=4;
  
  # install python dependencies (make sure to check box `Source code` sources) :
  sudo apt-get install -y git build-essential libmpich-dev;
  sudo apt-get build-dep -y python2.7;
  
  mkdir -p $SFT_DIR

  # install python :
  cd $SFT_DIR;
  wget https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz;
  tar -xzf Python-${PYTHON_VERSION}.tgz;
  cd $SFT_DIR/Python-${PYTHON_VERSION};
  ./configure --enable-optimizations \
              --enable-shared \
              --with-system-expat \
              --with-system-ffi \
              --enable-ipv6 \
              --with-threads \
              --with-pydebug \
              --with-lto \
              --prefix=${PYTHON_DIR};
  make -j ${BUILD_THREADS};
  make install;

  # install pip (https://pip.pypa.io/en/stable/installing/) :
  cd $SFT_DIR;
  wget https://bootstrap.pypa.io/get-pip.py;
  python get-pip.py;

  # install various python packages :
  pip install sympy \
              ipython \
              mpi4py \
              ply \
              sphinx \
              cython \
              pytest \
              pybind11 \
              nose;
  
  # install numpy with mkl :
  # https://software.intel.com/en-us/articles/numpyscipy-with-intel-mkl
  cd $SFT_DIR;
  git clone https://github.com/numpy/numpy.git;
  cd $SFT_DIR/numpy;
  git checkout maintenance/${NUMPY_VERSION}.x;
  touch site.cfg;
  echo [mkl] >> site.cfg;
  echo library_dirs = ${MKLROOT}/lib/intel64 >> site.cfg;
  echo include_dirs = ${MKLROOT}/include >> site.cfg;
  echo mkl_libs = mkl_rt >> site.cfg;
  echo lapack_libs = >> site.cfg;
  pip install . -v;

  # install scipy :
  cd $SFT_DIR;
  git clone https://github.com/scipy/scipy.git;
  cd $SFT_DIR/scipy;
  git checkout maintenance/${SCIPY_VERSION}.x;
  python setup.py install --record=install_files.txt;
  
  # finally, install matplotlib :
  cd $SFT_DIR;
  git clone https://github.com/matplotlib/matplotlib;
  cd $SFT_DIR/matplotlib;
  git checkout v${MATPLOTLIB_VERSION}.x;
  pip install . -v;
}
{% endhighlight %}

[intel-mkl]:   https://software.intel.com/en-us/mkl
[fenics]:      https://fenicsproject.org/
