# Don't edit this file directly, since it was generated from a template, 
# and you're changes will be *clobbered*.  Edit the template instead.


FROM ubuntu:14.04


ENV PYTHONPATH /opt/caffe/python

# Get dependencies
RUN apt-get update && apt-get install -y \
  bc \ 
  cmake \ 
  curl \ 
  gcc-4.6 \ 
  g++-4.6 \ 
  gcc-4.6-multilib \  
  g++-4.6-multilib \ 
  gfortran \ 
  git \ 
  libprotobuf-dev \
  libleveldb-dev \
  libsnappy-dev \
  libopencv-dev \
  libboost-all-dev \ 
  libhdf5-serial-dev \ 
  liblmdb-dev \  
  libjpeg62 \ 
  libfreeimage-dev \  
  libatlas-base-dev \  
  pkgconf \
  protobuf-compiler \ 
  python-dev \  
  python-pip \ 
  unzip \
  wget 

# Use gcc 4.6
RUN update-alternatives --install /usr/bin/cc cc /usr/bin/gcc-4.6 30 && \
  update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-4.6 30 && \ 
  update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.6 30 && \
  update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.6 30



# Clone the Caffe repo 
RUN cd /opt && git clone https://github.com/BVLC/caffe.git

RUN cd /opt/caffe && git checkout -t origin/dev


# Glog 
RUN cd /opt && wget https://google-glog.googlecode.com/files/glog-0.3.3.tar.gz && \
  tar zxvf glog-0.3.3.tar.gz && \
  cd /opt/glog-0.3.3 && \
  ./configure && \
  make && \
  make install

# Workaround for error loading libglog: 
#   error while loading shared libraries: libglog.so.0: cannot open shared object file
# The system already has /usr/local/lib listed in /etc/ld.so.conf.d/libc.conf, so
# running `ldconfig` fixes the problem (which is simpler than using $LD_LIBRARY_PATH)
# TODO: looks like this needs to be run _every_ time a new docker instance is run,
#       so maybe LD_LIBRARY_PATh is a better approach (or add call to ldconfig in ~/.bashrc)
RUN ldconfig

# Gflags
RUN cd /opt && \
  wget https://github.com/schuhschuh/gflags/archive/master.zip && \
  unzip master.zip && \
  cd /opt/gflags-master && \
  mkdir build && \
  cd /opt/gflags-master/build && \
  export CXXFLAGS="-fPIC" && \
  cmake .. && \ 
  make VERBOSE=1 && \
  make && \
  make install

# Build Caffe core
RUN cd /opt/caffe && \
  cp Makefile.config.example Makefile.config && \
   echo "CPU_ONLY := 1" >> Makefile.config && \ 
  echo "CXX := /usr/bin/g++-4.6" >> Makefile.config && \
  sed -i 's/CXX :=/CXX ?=/' Makefile && \
  make all

# Install python deps
RUN cd /opt/caffe && \
  (pip install -r python/requirements.txt; easy_install numpy; pip install -r python/requirements.txt) && \
  easy_install pillow

# Numpy include path hack - github.com/BVLC/caffe/wiki/Setting-up-Caffe-on-Ubuntu-14.04
RUN NUMPY_EGG=`ls /usr/local/lib/python2.7/dist-packages | grep -i numpy` && \
  ln -s /usr/local/lib/python2.7/dist-packages/$NUMPY_EGG/numpy/core/include/numpy /usr/include/python2.7/numpy

# Build Caffe python bindings
RUN cd /opt/caffe && make pycaffe

 
# Make + run tests
RUN cd /opt/caffe && make test && make runtest


# Copy all .bin binaries -> /usr/local/bin and remove .bin extension
RUN cd /opt/caffe/.build_release/tools && find * -name "*.bin" -exec sh -c 'DEST=$(echo {} | cut -d"." -f1) && echo "{} to /usr/local/bin/$DEST"; cp {} /usr/local/bin/$DEST && chmod +x /usr/local/bin/$DEST' \;
