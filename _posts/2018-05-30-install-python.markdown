---
layout: post
title:  "Build FEniCS 2017.2.0 from source with OpenBLAS"
date:   2018-05-30 
author: Evan Cummings
categories: jekyll update
---

If you are working with a cluster system, or simply want to take advantage of [OpenBLAS][openblas] linear-algebra libraries that are tuned for your particular computer architecture, you'll have to build some machine code from source.
The [FEniCS][fenics] devs are currently working on the latest 2018.1.0 version which will finally no longer support Python 2.7 and also take advantage of the lastest [PETSc][petsc] version.
Until this latest version is released, it is required to install some older software versions of the [FEniCS][fenics] dependencies; it is my hope that this letter will provide a demonstration for users to follow in order to get any past or future software which depends on [FEniCS][fenics] version 2017.2.0 up and running.

The steps to follow will build all the dependencies for my glacier simulator [cslvr][cslvr] which uses [FEniCS][fenics], and will be completed by calling the following function, which calls installation scripts for each of the dependencies to be described below:

{% highlight bash %}
function build_all()
{
  load_environment;
  install_dependencies;
  install_openblas;
  install_python;
  install_libarchive;
  install_cmake;
  install_pybind11;
  install_boost;
  install_eigen3;
  install_swig;
  install_hdf5;
  install_petsc;
  install_slepc;
  install_fenics;
  install_hsl;
  install_ipopt;
  install_dolfin_adjoint;
  install_basemap;
  install_gmsh;
  install_cslvr;
}
{% endhighlight %}

First, define environment variables for the versions of each code we'll need.
Intead of placing all the software in one folder---where each library may become intermingled---I install each in its own directory and manually add each path to my environment:

{% highlight bash %}
function load_environment()
{
  export PYTHON_VERSION=2.7.15;
  export NUMPY_VERSION=1.13;
  export SCIPY_VERSION=1.0;
  export MATPLOTLIB_VERSION=2.2;

  export LIBARCHIVE_VERSION=3.3.2;
  export PYBIND11_VERSION=2.2.1;  
  export BOOST_VERSION=1.67.0;
  export EIGEN_VERSION=3.3.4;
  export CMAKE_VERSION=3.11.1;
  export SWIG_VERSION=3.0.12;
  export HDF5_VERSION=1.10.2;
  export OPENBLAS_VERSION=0.2.20;
  
  export SLEPC_VERSION=3.8.2;
  export PETSC_VERSION=3.8.3;
  export PETSC4PY_VERSION=3.8.1;
  export SLEPC4PY_VERSION=3.8.0;
  
  export MSHR_VERSION="2017.2.0";
  export DOLFIN_VERSION="2017.2.0.post0";
  export PYPI_FENICS_VERSION=">=2017.2.0,<2018.1.0";
  export DOLFIN_ADJOINT_VERSION="2017.2.0";
  export PYADJOINT_VERSION="2017.2.0";
  
  export IPOPT_VERSION=3.12.9;
  
  export PREFIX=$HOME/local;
  export SFT_DIR=$HOME/software;
  
  export PYTHON_DIR=${PREFIX}/python-${PYTHON_VERSION};
 
  export LIBARCHIVE_DIR=${PREFIX}/libarchive-${LIBARCHIVE_VERSION};
  export PYBIND11_DIR=${PREFIX}/pybind11-${PYBIND11_VERSION}; 
  export CMAKE_DIR=${PREFIX}/cmake-${CMAKE_VERSION}; 
  export SWIG_DIR=${PREFIX}/swig-${SWIG_VERSION};
  export BOOST_DIR=${PREFIX}/boost-${BOOST_VERSION};
  export EIGEN_DIR=${PREFIX}/eigen-${EIGEN_VERSION};
  export HDF5_DIR=${PREFIX}/hdf5-${HDF5_VERSION};
  export OPENBLAS_DIR=${PREFIX}/openblas-${OPENBLAS_VERSION};
  export SLEPC_DIR=${PREFIX}/slepc-${SLEPC_VERSION};
  export PETSC_DIR=${PREFIX}/petsc-${PETSC_VERSION};
  export DOLFIN_DIR=${PREFIX}/dolfin-${DOLFIN_VERSION};
  export HSL_DIR=${PREFIX}/hsl;
  export IPOPT_DIR=${PREFIX}/ipopt-${IPOPT_VERSION};
  export LIBADJOINT_DIR=${PREFIX}/libadjoint-${DOLFIN_ADJOINT_VERSION};
  export GMSH_DIR=${PREFIX}/gmsh;
  
  export BUILD_THREADS=4;
  
  # make the packages available to paths :
  # NOTE: this will produce an error when loading before tools are installed.
  source ${DOLFIN_DIR}/share/dolfin/dolfin.conf;
  
  # add dependencies to path :
  export LD_LIBRARY_PATH=${LIBARCHIVE_DIR}/lib:$LD_LIBRARY_PATH;
  export LD_LIBRARY_PATH=${PYTHON_DIR}/lib:$LD_LIBRARY_PATH;
  export LD_LIBRARY_PATH=${BOOST_DIR}/lib:$LD_LIBRARY_PATH;
  export LD_LIBRARY_PATH=${HDF5_DIR}/lib:$LD_LIBRARY_PATH;
  export LD_LIBRARY_PATH=${OPENBLAS_DIR}/lib:$LD_LIBRARY_PATH;
  export LD_LIBRARY_PATH=${IPOPT_DIR}/lib:$LD_LIBRARY_PATH;
  export LD_LIBRARY_PATH=${LIBADJOINT_DIR}/lib:$LD_LIBRARY_PATH;
  export LD_LIBRARY_PATH=${GMSH_DIR}/lib:$LD_LIBRARY_PATH;

  export C_INCLUDE_PATH=${LIBARCHIVE_DIR}/include:$C_INCLUDE_PATH;
  export C_INCLUDE_PATH=${PYBIND11_DIR}/include:$C_INCLUDE_PATH;
  export C_INCLUDE_PATH=${BOOST_DIR}/include:$C_INCLUDE_PATH;
  export C_INCLUDE_PATH=${EIGEN_DIR}/include:$C_INCLUDE_PATH;
  export C_INCLUDE_PATH=${HD55_DIR}/include:$C_INCLUDE_PATH;
  export C_INCLUDE_PATH=${OPENBLAS_DIR}/include:$C_INCLUDE_PATH;
  export C_INCLUDE_PATH=${IPOPT_DIR}/include:$C_INCLUDE_PATH;
  export C_INCLUDE_PATH=${LIBADJOINT_DIR}/include:$C_INCLUDE_PATH;
  export C_INCLUDE_PATH=${GMSH_DIR}/include:$C_INCLUDE_PATH;

  export PATH=${LIBARCHIVE_DIR}/bin:$PATH;
  export PATH=${PYTHON_DIR}/bin:$PATH;
  export PATH=${CMAKE_DIR}/bin:$PATH;
  export PATH=${SWIG_DIR}/bin:$PATH;
  export PATH=${HDF5_DIR}/bin:$PATH;
  export PATH=${GMSH_DIR}/bin:$PATH;
  
  # add pythonpaths :
  export PYTHONPATH=${LIBADJOINT_DIR}/lib/python2.7/site-packages/:$PYTHONPATH
  export PYTHONPATH=${GMSH_DIR}/lib/python2.7/site-packages/:$PYTHONPATH

  # make the directory to store the source code :
  mkdir -p $SFT_DIR;
}
{% endhighlight %}

The function ``load_environment()`` can be called after all the software is installed to load all of your packages into your working environment.
We install source code into `$SFT_DIR` and Python will go into `$PYHON_DIR`.
While it may be simplier to just throw each of the packages in a single directory---such as `$HOME/.local`---I prefer to keep each package separated by version numbers so that if I ever want to compile the sources again with a new dependency version, I can simply change the version number at the top to change my entire compilation or operating environment.

If you have access to `sudo`, install all the Python dependencies the easy way.
Make sure to check the box *Source code* in your software data sources: press ``win`` key (we don't support planned-obsolescent policies), then type "software & updates".
Then the command ``apt-get build-dep`` will work:

{% highlight bash %}
function install_dependencies()
{
  # ensure a fortran compiler is available :
  sudo apt-get install -y gfortran;
  
  # install python dependencies (make sure to check box `Source code` sources) :
  sudo apt-get install -y git build-essential libmpich-dev;
  sudo apt-get build-dep -y python2.7;
  
  # install misc dependencies :
  sudo apt-get install -y curl slib libuv1-dev qt4-qmake \
                          libpng-dev zlib1g-dev libpcre3-dev;
  
  # install petsc dependencies :
  sudo apt-get install -y git valgrind libbison-dev flex;
  
  # install mshr dependencies :
  sudo apt-get install -y libgmp-dev libmpfr-dev;
  
  # install geos for basemap :
  sudo apt-get install -y libgeos-dev;
  
  # install gmsh dependencies :
  sudo apt-get install -y libfltk1.3-dev libcanberra-gtk-module;
  
  # install cslvr dependencies :
  sudo apt-get install -y libnetcdf-dev;

  # install latex for plotting with matplotlib :
  sudo apt-get install -y texlive-base texlive-latex-base texlive-latex-extra 
}
{% endhighlight %}

If you do not have access to ``sudo``, you'll have to request help from your system admin or build these dependencies yourself.
Next, build and install [OpenBLAS][openblas]:

{% highlight bash %}
function install_openblas()
{
  # install openblas :
  cd $SFT_DIR;
  wget --read-timeout=10 \
    -nc https://github.com/xianyi/OpenBLAS/archive/v${OPENBLAS_VERSION}.tar.gz \
    -O openblas.tar.bz;
  tar -xf openblas.tar.bz;
  rm openblas.tar.bz;
  cd OpenBLAS-${OPENBLAS_VERSION};
  make -j ${BUILD_TREADS} \
        USE_THREAD=0 \
        NUM_THREADS=1 \
        CC=gcc \
        FC=gfortran \
        PREFIX=${OPENBLAS_DIR};
  make install PREFIX=${OPENBLAS_DIR};
}
{% endhighlight %}
 
While not a neccessary requirement, let's build Python version 2.7.15 in a user directory, along with [pip][pip], [NumPy][numpy], [SciPy][scipy], and [Matplotlib][matplotlib], and some other dependencies: 

{% highlight bash %}
function install_python()
{
  # install python :
  cd $SFT_DIR;
  url="https://www.python.org/ftp/python/${PYTHON_VERSION}/\
       Python-${PYTHON_VERSION}.tgz";
  url=$(tr -d ' ' <<< "$url");   # remove spaces to fit the url within 80 chr
  wget $url;
  tar -xzf Python-${PYTHON_VERSION}.tgz;
  rm Python-${PYTHON_VERSION}.tgz;
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
  rm get-pip.py;

  # install various python packages :
  pip install sympy \
              ipython \
              mpi4py \
              ply \
              sphinx \
              cython \
              pytest \
              nose;
  
  # install numpy with openblas :
  # https://leemendelowitz.github.io/blog/installing-numpy-with-openblas.html
  cd $SFT_DIR;
  git clone https://github.com/numpy/numpy.git;
  cd $SFT_DIR/numpy;
  git checkout maintenance/${NUMPY_VERSION}.x;
  touch site.cfg;
  echo [DEFAULT] >> site.cfg;
  echo library_dirs = ${OPENBLAS_DIR}/lib >> site.cfg;
  echo include_dirs = ${OPENBLAS_DIR}/include >> site.cfg;
  echo "" >> site.cfg;
  echo [atlas] >> site.cfg;
  echo atlas_libs   = openblas >> site.cfg;
  echo libraries    = openblas >> site.cfg;
  echo "" >> site.cfg;
  echo [openblas] >> site.cfg;
  echo libraries    = openblas >> site.cfg;
  echo library_dirs = ${OPENBLAS_DIR}/lib >> site.cfg;
  echo include_dirs = ${OPENBLAS_DIR}/include >> site.cfg;
  pip install . -v;

  # install scipy ignoring numpy's version (pip will fail, so use setup.py) :
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

The rest of this "tutorial" is rather terse, but you get the idea.
[Libarchive][libarchive], needed by [HDF5][hdf5] and [CMake][cmake]:

{% highlight bash %}
function install_libarchive()
{
  # install libarchive:
  cd $SFT_DIR;
  url="http://www.libarchive.org/downloads/\
       libarchive-${LIBARCHIVE_VERSION}.tar.gz";
  url=$(tr -d ' ' <<< "$url");   # remove spaces to fit the url within 80 chr
  wget $url;
  tar -xzf libarchive-${LIBARCHIVE_VERSION}.tar.gz;
  rm libarchive-${LIBARCHIVE_VERSION}.tar.gz;
  cd $SFT_DIR/libarchive-${LIBARCHIVE_VERSION};
  ./configure --disable-static \
              --prefix=${LIBARCHIVE_DIR};
  make -j ${BUILD_THREADS};
  make install;
}
{% endhighlight %}

Next, the lastest version of [CMake][cmake]:

{% highlight bash %}
function install_cmake()
{
  # install cmake :
  cd $SFT_DIR;
  url="https://cmake.org/files/v${CMAKE_VERSION%.*}/\
       cmake-${CMAKE_VERSION}.tar.gz";
  url=$(tr -d ' ' <<< "$url");   # remove spaces to fit the url within 80 chr
  wget $url;
  tar -xzf cmake-${CMAKE_VERSION}.tar.gz;
  rm cmake-${CMAKE_VERSION}.tar.gz;
  cd $SFT_DIR/cmake-${CMAKE_VERSION};
  ./bootstrap --parallel=${BUILD_THREADS} \
              --prefix=${CMAKE_DIR};
  make -j ${BUILD_THREADS};
  make install;
}
{% endhighlight %}

[Pybind11][pybind11]:

{% highlight bash %}
function install_pybind11()
{
  # install pybind11 :
  cd $SFT_DIR;
  url="https://github.com/pybind/pybind11/archive/v${PYBIND11_VERSION}.tar.gz";
  wget -nc $url -O pybind.tgz;
  tar -xzf pybind.tgz;
  rm pybind.tgz;
  mkdir pybind11-${PYBIND11_VERSION}/build;
  cd pybind11-${PYBIND11_VERSION}/build;
  cmake -D PYBIND11_TEST=OFF \
        -D PYTHON_EXECUTABLE:FILEPATH=$(which python) \
        -D CMAKE_INSTALL_PREFIX=${PYBIND11_DIR} ..;
  make install;
}
{% endhighlight %}

[Boost][boost]:

{% highlight bash %}
function install_boost()
{
  # install boost (with mpi) :
  cd $SFT_DIR;
  boost_ver=$(tr "\." "_" <<< ${BOOST_VERSION});
  url="https://dl.bintray.com/boostorg/release/${BOOST_VERSION}/\
       source/boost_${boost_ver}.tar.gz";
  url=$(tr -d ' ' <<< "$url");   # remove spaces to fit the url within 80 chr
  wget $url;
  mkdir -p boost-${BOOST_VERSION};
  tar -xzf boost_${boost_ver}.tar.gz -C boost-${BOOST_VERSION} \
            --strip-components 1;
  rm boost_${boost_ver}.tar.gz;
  cd $SFT_DIR/boost-${BOOST_VERSION};
  ./bootstrap.sh --with-toolset=gcc \
                 --with-python=$(which python) \
                 --prefix=${BOOST_DIR};
  echo "using mpi : mpicxx ;" >> project-config.jam;
  ./b2 -j ${BUILD_THREADS} toolset=gcc --build-dir=build install;
}
{% endhighlight %}

[Eigen][eigen3]:

{% highlight bash %}
function install_eigen3()
{
  # install eigen3 :
  cd $SFT_DIR;
  wget http://bitbucket.org/eigen/eigen/get/${EIGEN_VERSION}.tar.bz2;
  mkdir -p eigen-${EIGEN_VERSION};
  tar -xf ${EIGEN_VERSION}.tar.bz2 -C eigen-${EIGEN_VERSION} \
          --strip-components 1;
  rm ${EIGEN_VERSION}.tar.bz2;
  rm $SFT_DIR/eigen-${EIGEN_VERSION}/build;
  mkdir $SFT_DIR/eigen-${EIGEN_VERSION}/build;
  cd $SFT_DIR/eigen-${EIGEN_VERSION}/build;
  cmake -D BOOST_ROOT=${BOOST_DIR} \
        -D CMAKE_INSTALL_PREFIX=${EIGEN_DIR} ..;
  make install;
}
{% endhighlight %}

[SWIG][swig]:

{% highlight bash %}
function install_swig()
{ 
  # install swig :
  cd $SFT_DIR;
  url="http://downloads.sourceforge.net/swig/swig-${SWIG_VERSION}.tar.gz";
  wget -nc $url -O swig-${SWIG_VERSION}.tar.gz;
  tar -xf swig-${SWIG_VERSION}.tar.gz;
  rm swig-${SWIG_VERSION}.tar.gz;
  cd $SFT_DIR/swig-${SWIG_VERSION};
  ./configure --with-python=$(which python) \
              --with-boost=${BOOST_DIR} \
              --prefix=${SWIG_DIR};
  make -j ${BUILD_THREADS};
  make install;
}
{% endhighlight %}

[HDF5][hdf5]:

{% highlight bash %}
function install_hdf5()
{
  # install hdf5 :
  cd $SFT_DIR;
  url="https://support.hdfgroup.org/ftp/HDF5/releases/\
       hdf5-${HDF5_VERSION%.*}/hdf5-${HDF5_VERSION}/\
       src/hdf5-${HDF5_VERSION}.tar.gz";
  url=$(tr -d ' ' <<< "$url");   # remove spaces to fit the url within 80 chr
  wget $url;
  tar -xzf hdf5-${HDF5_VERSION}.tar.gz;
  rm hdf5-${HDF5_VERSION}.tar.gz;
  rm -rf $SFT_DIR/hdf5-${HDF5_VERSION}/build;
  mkdir $SFT_DIR/hdf5-${HDF5_VERSION}/build;
  cd $SFT_DIR/hdf5-${HDF5_VERSION}/build;
  ../configure CC=mpicc \
               CXX=mpicxx \
               FC=mpifort \
               --with-default-api-version=v18 \
               --enable-build-mode=production \
               --enable-parallel \
               --enable-shared \
               --enable-fortran \
               --with-zlib=yes \
               --with-szlib=yes \
               --prefix=${HDF5_DIR};
  make -j ${BUILD_THREADS};
  #make check;  # takes forever
  make install;
  make check-install;
}
{% endhighlight %}

[PETSc][petsc]:

{% highlight bash %}
function install_petsc()
{
  # install petsc :
  cd $SFT_DIR;
  wget -nc https://bitbucket.org/petsc/petsc/get/v${PETSC_VERSION}.tar.gz \
       -O ${SFT_DIR}/petsc-${PETSC_VERSION}.tar.gz;
  mkdir -p  ${SFT_DIR}/petsc-${PETSC_VERSION};
  tar -xzf ${SFT_DIR}/petsc-${PETSC_VERSION}.tar.gz \
             -C ${SFT_DIR}/petsc-${PETSC_VERSION} \
             --strip-components 1;
  rm ${SFT_DIR}/petsc-${PETSC_VERSION}.tar.gz;
  cd $SFT_DIR/petsc-${PETSC_VERSION};
  export PETSC_DIR=$(pwd);
  ./configure --with-cc=mpicc \
              --with-cxx=mpicxx \
              --with-fc=mpifort \
              --COPTFLAGS="-O2" \
              --CXXOPTFLAGS="-O2" \
              --FOPTFLAGS="-O2" \
              --with-c-support=1 \
              --with-cxx-dialect=C++11 \
              --with-debugging=0 \
              --with-shared-libraries=1 \
              --with-boost-dir=${BOOST_DIR} \
              --with-hdf5-dir=${HDF5_DIR} \
              --with-blas-lib=${OPENBLAS_DIR}/lib/libopenblas.a \
              --with-lapack-lib=${OPENBLAS_DIR}/lib/libopenblas.a \
              --download-scalapack=1 \
              --download-blacs=1 \
              --download-hypre=1 \
              --download-metis=1 \
              --download-mumps=1 \
              --download-parmetis=1 \
              --download-ptscotch=1 \
              --download-spai=1 \
              --download-elemental=1 \
              --download-ml=1 \
              --download-suitesparse=1 \
              --download-superlu=1 \
              --download-superlu_dist=1 \
              --prefix=${PREFIX}/petsc-${PETSC_VERSION};
  make PETSC_DIR=${PETSC_DIR} PETSC_ARCH=arch-linux2-c-opt all;
  make PETSC_DIR=${PETSC_DIR} PETSC_ARCH=arch-linux2-c-opt install;
  export PETSC_DIR=${PREFIX}/petsc-${PETSC_VERSION};
  make PETSC_DIR=${PETSC_DIR} PETSC_ARCH="" test;
  
  # install petsc4py 
  # https://www.mcs.anl.gov/petsc/petsc4py-current/docs/usrman/install.html :
  cd $SFT_DIR;
  url="https://bitbucket.org/petsc/petsc4py/downloads/\
       petsc4py-${PETSC4PY_VERSION}.tar.gz";
  wget $url;
  tar -xzvf petsc4py-${PETSC4PY_VERSION}.tar.gz;
  rm petsc4py-${PETSC4PY_VERSION}.tar.gz;
  cd $SFT_DIR/petsc4py-${PETSC4PY_VERSION};
  pip install . -v;
}
{% endhighlight %}

[SLEPc][slepc]:

{% highlight bash %}
function install_slepc()
{
  # install slepc :
  cd $SFT_DIR;
  wget -nc http://slepc.upv.es/download/distrib/slepc-${SLEPC_VERSION}.tar.gz \
       -O $SFT_DIR/slepc-${SLEPC_VERSION}.tar.gz;
  mkdir -p $SFT_DIR/slepc-${SLEPC_VERSION};
  tar -xzf $SFT_DIR/slepc-${SLEPC_VERSION}.tar.gz 
             -C $SFT_DIR/slepc-${SLEPC_VERSION} \
             --strip-components 1;
  rm $SFT_DIR/slepc-${SLEPC_VERSION}.tar.gz;
  cd $SFT_DIR/slepc-${SLEPC_VERSION};
  export SLEPC_DIR=$(pwd);
  ./configure --prefix=${PREFIX}/slepc-${SLEPC_VERSION};
  make SLEPC_DIR=${SLEPC_DIR} PETSC_DIR=${PETSC_DIR};
  make SLEPC_DIR=${SLEPC_DIR} PETSC_DIR=${PETSC_DIR} install;
  export SLEPC_DIR=${PREFIX}/slepc-${SLEPC_VERSION};
  make SLEPC_DIR=${SLEPC_DIR} PETSC_DIR=${PETSC_DIR} PETSC_ARCH="" test;

  # install slepc4py 
  # http://slepc.upv.es/slepc4py-current/docs/usrman/install.html :
  cd $SFT_DIR;
  url="https://bitbucket.org/slepc/slepc4py/\
       downloads/slepc4py-${SLEPC4PY_VERSION}.tar.gz";
  wget $url;
  tar -xzvf slepc4py-${SLEPC4PY_VERSION}.tar.gz;
  rm slepc4py-${SLEPC4PY_VERSION}.tar.gz;
  cd $SFT_DIR/slepc4py-${SLEPC4PY_VERSION};
  pip install . -v;
}
{% endhighlight %}

[FEniCS][fenics]:

{% highlight bash %}
function install_fenics()
{
  # install dolfin dependencies :
  pip install fenics${PYPI_FENICS_VERSION} -v;

  # install dolfin :
  cd $SFT_DIR;
  git clone https://bitbucket.org/fenics-project/dolfin.git;
  cd dolfin;
  git checkout ${DOLFIN_VERSION};
  rm -rf $SFT_DIR/dolfin/build;
  mkdir $SFT_DIR/dolfin/build;
  cd $SFT_DIR/dolfin/build;
  cmake -D PYTHON_EXECUTABLE:FILEPATH=$(which python) \
        -D BLAS_LIBRARIES=${OPENBLAS_DIR}/lib/libopenblas.a \
        -D BLAS_LINKER_FLAGS="-I${OPENBLAS_DIR}/include" \
        -D LAPACK_LIBRARIES==${OPENBLAS_DIR}/lib/libopenblas.a \
        -D LAPACK_LINKER_FLAGS="-I${OPENBLAS_DIR}/include" \
        -D SLEPC_INCLUDE_DIRS=${SLEPC_DIR}/include \
        -D PETSC_INCLUDE_DIRS=${PETSC_DIR}/include \
        -D SWIG_EXECUTABLE=${SWIG_DIR}/bin/swig \
        -D BOOST_ROOT=${BOOST_DIR} \
        -D HDF5_ROOT=${HDF5_DIR} \
        -D BOOST_LIBRARYDIR=${BOOST_DIR}/lib \
        -D EIGEN3_INCLUDE_DIR=${EIGEN_DIR}/include/eigen3 \
        -D DOLFIN_USE_PYTHON3=false \
        -D DOLFIN_ENABLE_HDF5=true \
        -D DOLFIN_ENABLE_MPI=true \
        -D DOLFIN_ENABLE_DOCS=true \
        -D DOLFIN_ENABLE_SPHINX=true \
        -D DOLFIN_SKIP_BUILD_TESTS=false \
        -D CMAKE_CXX_FLAGS="-O2" \
        -D CMAKE_C_FLAGS="-O2" \
        -D CMAKE_C_COMPILER=mpicc \
        -D CMAKE_CXX_COMPILER=mpicxx \
        -D CMAKE_INSTALL_PREFIX=${DOLFIN_DIR} ..;
  make -j ${BUILD_THREADS};
  make install;

  # ensure that dolfin is in all paths :
  source ${DOLFIN_DIR}/share/dolfin/dolfin.conf;

  # install mshr :  
  cd $SFT_DIR;
  git clone https://bitbucket.org/fenics-project/mshr.git;
  cd $SFT_DIR/mshr;
  git checkout ${MSHR_VERSION};
  rm -rf $SFT_DIR/mshr/build;
  mkdir $SFT_DIR/mshr/build;
  cd $SFT_DIR/mshr/build;
  cmake -D EIGEN3_INCLUDE_DIR=${EIGEN_DIR}/include/eigen3 \
        -D SWIG_EXECUTABLE=${SWIG_DIR}/bin/swig \
        -D CMAKE_CXX_FLAGS="-O2" \
        -D CMAKE_C_FLAGS="-O2" \
        -D CMAKE_C_COMPILER=mpicc \
        -D CMAKE_CXX_COMPILER=mpicxx \
        -D CMAKE_INSTALL_PREFIX=${DOLFIN_DIR} ..;
  make -j ${BUILD_THREADS};
  make install;
}
{% endhighlight %}

[HSL][hsl]:

{% highlight bash %}
function install_hsl()
{
  mkdir -p ${HSL_DIR};
  tar -xzf /path/to/coinhsl-linux-x86_64-2015.06.23.tar.gz \
             -C ${HSL_DIR} \
             --strip-components 1;
}
{% endhighlight %}

[IPOPT][ipopt]:

{% highlight bash %}
function install_ipopt()
{
  # first, build ipopt :
  # NOTE: remember to copy hsl to $PREFIX/hsl :
  cd $SFT_DIR;
  url="https://www.coin-or.org/download/source/Ipopt/\
       Ipopt-${IPOPT_VERSION}.tgz";
  wget -nc $url -O $SFT_DIR/ipopt-${IPOPT_VERSION}.tgz;
  mkdir -p $SFT_DIR/ipopt-${IPOPT_VERSION};
  tar -xvf $SFT_DIR/ipopt-${IPOPT_VERSION}.tgz \
             -C $SFT_DIR/ipopt-${IPOPT_VERSION} \
             --strip-components 1;
  rm $SFT_DIR/ipopt-${IPOPT_VERSION}.tgz;
  cd $SFT_DIR/ipopt-${IPOPT_VERSION};
  rm -rf $SFT_DIR/ipopt-${IPOPT_VERSION}/build;
  mkdir $SFT_DIR/ipopt-${IPOPT_VERSION}/build;
  cd $SFT_DIR/ipopt-${IPOPT_VERSION}/build;
  ../configure --prefix=${IPOPT_DIR} \
               --enable-shared \
               --enable-debug \
               --with-hsl-lib="-L${HSL_DIR}/lib -lcoinhsl -lmetis" \
               --with-hsl-incdir="${HSL_DIR}/include" \
               --with-metis-lib="-Wl,-rpath,${PETSC_DIR}/lib \
                                 -L${PETSC_DIR}/lib -lmetis" \
               --with-metis-incdir="${PETSC_DIR}/include -I/usr/include/mpi" \
               --with-blas-lib=${OPENBLAS_DIR}/lib/libopenblas.a \
               --with-blas-incdir=${OPENBLAS_DIR}/include \
               --with-lapack-lib=${OPENBLAS_DIR}/lib/libopenblas.a \
               --with-lapack-incdir=${OPENBLAS_DIR}/include \
               F77=mpifort \
               CC=mpicc \
               CXX=mpicxx;
  make -j ${BUILD_THREADS};
  make test;
  make install;
  
  # pyipopt :
  cd $SFT_DIR;
  git clone git@github.com:pf4d/pyipopt.git;
  cd $SFT_DIR/pyipopt;
  git checkout openblas;   # I created this just for you.

  # add the additional include directories for our installation :
  sed -i "s/coinmumps/dmumps/g" setup.py;
  sed -i "s/library_dirs=[IPOPT_LIB]\
           /library_dirs=[IPOPT_LIB,\
                          '${PETSC_DIR}/lib',\
                          '${HSL_DIR}/lib',\
                          '${OPENBLAS_DIR}/lib']/g" \
         setup.py;
  sed -i "s/include_dirs=[numpy_include, IPOPT_INC]\
           /include_dirs=[numpy_include, IPOPT_INC,\
                          '${PETSC_DIR}/include',\
                          '${HSL_DIR}/include',\
                          '${OPENBLAS_DIR}/include']/g" \
         setup.py;
  pip uninstall -y pyipopt;
  rm -rf build;
  python setup.py build;
  pip install . -v;
}
{% endhighlight %}

[Dolfin-adjoint][dolfin_adjoint]:

{% highlight bash %}
function install_dolfin_adjoint()
{
  # install libadjoint :
  cd $SFT_DIR;
  git clone https://bitbucket.org/dolfin-adjoint/libadjoint;
  cd $SFT_DIR/libadjoint;
  git checkout libadjoint-${DOLFIN_ADJOINT_VERSION};
  rm -rf $SFT_DIR/libadjoint/build;
  mkdir $SFT_DIR/libadjoint/build;
  cd $SFT_DIR/libadjoint/build;
  cmake -D PYTHON_EXECUTABLE:FILEPATH=$(which python) \
        -D CMAKE_CXX_FLAGS="-O2" \
        -D CMAKE_C_FLAGS="-O2" \
        -D CMAKE_C_COMPILER=mpicc \
        -D CMAKE_CXX_COMPILER=mpicxx \
        -D CMAKE_INSTALL_PREFIX=${LIBADJOINT_DIR} ..;
  make;
  make install;

  # dolfin_adjoint :
  cd $SFT_DIR;
  git clone https://bitbucket.org/dolfin-adjoint/dolfin-adjoint
  cd $SFT_DIR/dolfin-adjoint;
  git checkout dolfin-adjoint-${DOLFIN_ADJOINT_VERSION};
  pip install . -v;
}
{% endhighlight %}

[Basemap][basemap]:

{% highlight bash %}
function install_basemap()
{
  cd $SFT_DIR;
  git clone https://github.com/matplotlib/basemap.git;
  cd $SFT_DIR/basemap;
  pip install . -v; 
}
{% endhighlight %}

[Gmsh][gmsh]:

{% highlight bash %}
function install_gmsh()
{
  # install gmsh :
  cd $SFT_DIR
  git clone https://gitlab.onelab.info/gmsh/gmsh.git;
  rm -rf $SFT_DIR/gmsh/build;
  mkdir $SFT_DIR/gmsh/build;
  cd $SFT_DIR/gmsh/build;
  cmake -D BLAS_LAPACK_LIBRARIES="${OPENBLAS_DIR}/lib/libopenblas.so" \
        -D ENABLE_WRAP_PYTHON=ON \
        -D ENABLE_FLTK=ON \
        -D ENABLE_PRIVATE_API=ON \
        -D SWIG_EXECUTABLE=${SWIG_DIR}/bin/swig \
        -D PYTHON_EXECUTABLE:FILEPATH=$(which python) \
        -D ENABLE_PETSC=OFF \
        -D ENABLE_SLEPC=OFF \
        -D ENABLE_PETSC4PY=OFF \
        -D ENABLE_MPI=ON \
        -D CMAKE_C_COMPILER=mpicc \
        -D CMAKE_CXX_COMPILER=mpicxx \
        -D CMAKE_Fortran_COMPILER=mpifort \
        -D CMAKE_INSTALL_PREFIX=${GMSH_DIR} ..;
  make -j ${BUILD_THREADS};
  make install;
}
{% endhighlight %}

[cslvr][cslvr]:

{% highlight bash %}
function install_cslvr()
{
  # cslvr dependencies (tifffile requires numpy==1.14 which is incompatible 
  # with fenics, so ignore its dependencies) :
  pip install tifffile --no-deps;
  pip install pyproj termcolor colored shapely futures netcdf4 image;

  # download cslvr source :
  cd $SFT_DIR;
  git clone git@github.com:pf4d/cslvr.git;
  export PYTHONPATH="$SFT_DIR/cslvr:$PYTHONPATH"
}
{% endhighlight %}

There, all done!
Now, your `$PREFIX` directory will contain the software

{% highlight bash %}
$ ls $PREFIX
boost-1.67.0           gmsh          libadjoint-2017.2.0  python-2.7.15
cmake-3.11.1           hdf5-1.10.2   openblas-0.2.20      slepc-3.8.2
dolfin-2017.2.0.post0  hsl           petsc-3.8.3          swig-3.0.12
eigen-3.3.4            ipopt-3.12.9  pybind11-2.2.1
{% endhighlight %}

Hope this helps! :]

[intel-mkl]:      https://software.intel.com/en-us/mkl
[openblas]:       https://www.openblas.net/
[fenics]:         https://fenicsproject.org/
[cslvr]:          http://cslvr.readthedocs.io/en/latest/
[pip]:            https://pypi.org/project/pip/#description
[latex]:          https://www.latex-project.org/
[sphinx]:         http://www.sphinx-doc.org/en/master/
[sympy]:          http://www.sympy.org
[matplotlib]:     https://matplotlib.org/
[numpy]:          http://www.numpy.org/
[scipy]:          https://scipy.org/
[python]:         https://www.python.org/
[paraview]:       https://www.paraview.org/
[libarchive]:     https://www.libarchive.org/
[cmake]:          https://cmake.org/
[eigen3]:         http://eigen.tuxfamily.org/index.php?title=Main_Page
[swig]:           http://www.swig.org/index.php
[pybind11]:       https://pybind11.readthedocs.io/en/stable/
[boost]:          https://www.boost.org/
[hdf5]:           https://portal.hdfgroup.org/display/HDF5/HDF5
[petsc]:          https://www.mcs.anl.gov/petsc/
[slepc]:          http://slepc.upv.es/
[hsl]:            http://www.hsl.rl.ac.uk/ipopt/
[IPOPT]:          https://projects.coin-or.org/Ipopt
[dolfin_adjoint]: http://www.dolfin-adjoint.org/en/latest/
[basemap]:        https://matplotlib.org/basemap/
[gmsh]:           http://gmsh.info/


