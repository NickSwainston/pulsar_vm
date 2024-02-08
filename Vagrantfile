# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "linuxmint-21.3-cinnamon"

  config.ssh.username = 'pulsar'
  config.ssh.password = 'pulsar'

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "shared", "/home/pulsar/shared"
  #
  # View the documentation for the provider you are using for more
  # information on available options.


  config.vm.provision "apt-dependencies", type: "shell", inline: <<-SHELL
    # Install dependencies
    DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get install -y software-properties-common
    add-apt-repository universe
    apt-get update
    apt-get --no-install-recommends -y install \
      build-essential \
      autoconf \
      autotools-dev \
      automake \
      autogen \
      libtool \
      pkg-config \
      cmake \
      csh \
      g++ \
      gcc \
      gfortran \
      wget \
      git \
      expect \
      libcfitsio-dev \
      hwloc \
      perl \
      pcre2-utils \
      libpcre2-dev \
      pgplot5 \
      python3.10 \
      python3-dev \
      python3-testresources \
      python3-pip \
      python3-setuptools \
      python3-tk \
      libfftw3-3 \
      libfftw3-bin \
      libfftw3-dev \
      libfftw3-single3 \
      libx11-dev \
      libpcre3 \
      libpcre3-dev \
      libpng-dev \
      libpnglite-dev \
      libhdf5-dev \
      libhdf5-serial-dev \
      libxml2 \
      libxml2-dev \
      libltdl-dev \
      libffi-dev \
      libssl-dev \
      libxft-dev \
      libfreetype6-dev \
      libblas-dev \
      liblapack-dev \
      libatlas-base-dev \
      libsuitesparse-dev \
      gsl-bin \
      libgsl-dev \
      bc
    rm -rf /var/lib/apt/lists/*
    apt-get -y clean
  SHELL

  config.vm.provision "file", source: "bashrc", destination: "/home/pulsar/.bashrc"
  config.vm.provision "bashrc", type: "shell", inline: <<-SHELL
    source ~/.bashrc
    mkdir -p $PSRHOME
    git config --global http.postBuffer 1048576000
  SHELL

  config.vm.provision "python-dependencies", type: "shell", inline: <<-SHELL
    pip install -U \
      pip \
      setuptools
    pip install -U \
      cython \
      scikit-image  \
      numpy==1.23.0
    pip install -U \
      pandas \
      matplotlib \
      astropy \
      scipy
  SHELL

  config.vm.provision "swig", type: "shell", inline: <<-SHELL
    wget https://sourceforge.net/projects/swig/files/swig/swig-4.0.1/swig-4.0.1.tar.gz
    tar -xvf swig-4.0.1.tar.gz -C $PSRHOME
    rm swig-4.0.1.tar.gz
    cd $PSRHOME/swig-4.0.1
    ./configure --prefix=$SWIG_DIR
    make -j $(nproc)
    make install
    make clean
  SHELL

  config.vm.provision "calceph", type: "shell", inline: <<-SHELL
    wget https://www.imcce.fr/content/medias/recherche/equipes/asd/calceph/calceph-3.5.1.tar.gz
    tar -xvf calceph-3.5.1.tar.gz -C $PSRHOME
    rm calceph-3.5.1.tar.gz
    cd $CALCEPH
    ./configure --prefix=$CALCEPH/install --with-pic --enable-shared --enable-static --enable-fortran --enable-thread
    make -j $(nproc)
    make check
    make install
    make clean
  SHELL

  config.vm.provision "psrcat", type: "shell", inline: <<-SHELL
    wget http://www.atnf.csiro.au/people/pulsar/psrcat/downloads/psrcat_pkg.tar.gz
    tar -xvf psrcat_pkg.tar.gz -C $PSRHOME
    rm psrcat_pkg.tar.gz
    cd $PSRHOME/psrcat_tar
    # add flag to ignore a multiple definition error for gcc version >9
    sed -i 's&/usr/bin/gcc&/usr/bin/gcc -fcommon&g' makeit
    /bin/bash makeit
  SHELL

  config.vm.provision "psrxml", type: "shell", inline: <<-SHELL
    cd $PSRHOME
    git clone https://github.com/SixByNine/psrxml.git
    cd $PSRXML
    autoreconf --install --warnings=none
    ./configure --prefix=$PSRXML/install
    make -j $(nproc)
    make install
    make clean
  SHELL

  config.vm.provision "tempo", type: "shell", inline: <<-SHELL
    cd $PSRHOME
    git clone git://git.code.sf.net/p/tempo/tempo tempo
    cd $TEMPO_DIR
    ./prepare
    ./configure --prefix=$TEMPO
    FFLAGS="$FFLAGS -O3 -m64"
    make -j $(nproc)
    make install
    # copy data files and build/install utilities
    cp -r clock/ ephem/ tzpar/ obsys.dat tempo.cfg tempo.hlp $TEMPO
    sed -i "s;${TEMPO_DIR};${TEMPO};g" ${TEMPO}/tempo.cfg
    cd ${TEMPO_DIR}/src
    make matrix
    cp matrix ${TEMPO}/bin/
    cd ${TEMPO_DIR}/util/lk
    gfortran -o lk lk.f
    cp lk ${TEMPO}/bin/
    cp ${TEMPO_DIR}/util/dmx/* ${TEMPO}/bin/
    cp ${TEMPO_DIR}/util/dmxparse/* ${TEMPO}/bin/
    cp ${TEMPO_DIR}/util/dmx_ranges/* ${TEMPO}/bin/
    chmod +x ${TEMPO}/bin/DMX_ranges2.py
    cp ${TEMPO_DIR}/util/dmx_broaden/* ${TEMPO}/bin/
    cp ${TEMPO_DIR}/util/cull/cull.pl ${TEMPO}/bin/cull
    cp ${TEMPO_DIR}/util/extract/extract.pl ${TEMPO}/bin/extract
    cp ${TEMPO_DIR}/util/obswgt/obswgt.pl ${TEMPO}//bin/obswg
    cd ${TEMPO_DIR}/util/print_resid
    make -j $(nproc)
    cp print_resid ${TEMPO}/bin/
    cp ${TEMPO_DIR}/util/res_avg/* ${TEMPO}/bin/
    cp ${TEMPO_DIR}/util/wgttpo/wgttpo.pl ${TEMPO}/bin/wgttpo
    cp ${TEMPO_DIR}/util/wgttpo/wgttpo_emin.pl ${TEMPO}/bin/wgttpo_emin
    cp ${TEMPO_DIR}/util/wgttpo/wgttpo_equad.pl ${TEMPO}/bin/wgttpo_equad
    cd ${TEMPO_DIR}/util/ut1
    gcc -o predict_ut1 predict_ut1.c $(gsl-config --libs)
    cp predict_ut1 check.ut1 do.iers.ut1 do.iers.ut1.new get_ut1 get_ut1_new make_ut1 ${TEMPO}/bin/
    cp ${TEMPO_DIR}/util/compare_tempo/compare_tempo ${TEMPO}/bin/
    cp ${TEMPO_DIR}/util/pubpar/pubpar.py ${TEMPO}/bin/
    chmod +x ${TEMPO}/bin/pubpar.py
    cp ${TEMPO_DIR}/util/center_epoch/center_epoch.py ${TEMPO}/bin/
    cd ${TEMPO_DIR}/util/avtime
    gfortran -o avtime avtime.f
    cp avtime ${TEMPO}/bin/
    cd ${TEMPO_DIR}/util/non_tempo
    cp dt mjd aolst ${TEMPO}/bin/
    cd ${TEMPO_DIR}
  SHELL

  config.vm.provision "tempo2", type: "shell", inline: <<-SHELL
    cd $PSRHOME
    git clone https://bitbucket.org/psrsoft/tempo2.git
    cd $TEMPO2_DIR
    ./bootstrap
    cp -r T2runtime/ $TEMPO2/
    ./configure --prefix=$TEMPO2 --with-x --x-libraries=/usr/lib/x86_64-linux-gnu --with-fftw3-dir=/usr/ --with-calceph=$CALCEPH/install/lib \
    --enable-shared --enable-static --with-pic \
    CPPFLAGS="-I"$CALCEPH"/install/include -L"$CALCEPH"/install/lib/ -I"$PGPLOT_DIR"/include/ -L"$PGPLOT_DIR"/lib/" \
    CXXFLAGS="-I"$CALCEPH"/install/include -L"$CALCEPH"/install/lib/ -I"$PGPLOT_DIR"/include/ -L"$PGPLOT_DIR"/lib/"
    make -j $(nproc)
    make -j $(nproc) plugins
    make install
    make plugins-install
    make clean && make plugins-clean
  SHELL

  config.vm.provision "psrchive", type: "shell", inline: <<-SHELL
    cd $PSRHOME
    git clone git://git.code.sf.net/p/psrchive/code psrchive
    cd $PSRCHIVE_DIR
    ./bootstrap
    ./configure --prefix=$PSRCHIVE --with-x --x-libraries=/usr/lib/x86_64-linux-gnu --with-psrxml-dir=$PSRXML/install --enable-shared --enable-static \
    F77=gfortran \
    PYTHON=$(which python3)\
    CPPFLAGS="-I"$CALCEPH"/install/include -L"$CALCEPH"/install/lib/ -I"$PGPLOT_DIR"/include/ -L"$PGPLOT_DIR"/lib/" \
    CXXFLAGS="-I"$CALCEPH"/install/include -L"$CALCEPH"/install/lib/ -I"$PGPLOT_DIR"/include/ -L"$PGPLOT_DIR"/lib/" \
    LDFLAGS="-L"$PSRXML"/install/lib -L"$CALCEPH"/install/lib/  -L"$PGPLOT_DIR"/lib/ " LIBS="-lpsrxml -lxml2"
    make -j $(nproc)
    make install
    make clean
  SHELL

  config.vm.provision "enterprise", type: "shell", inline: <<-SHELL
    cd $PSRHOME
    git clone https://github.com/nanograv/enterprise.git
    cd enterprise
    pip install -r requirements.txt
    export TEMPO2_PREFIX=$TEMPO2
    pip install libstempo jupyter
    pip install .
    pip install git+https://github.com/nanograv/enterprise_extensions@master
  SHELL

  # Add in the worksho files
  config.vm.provision "file", source: "2024_workshop", destination: "/home/pulsar/2024_workshop"

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true

    # Customize the amount of memory on the VM:
    vb.memory = "8192"
    vb.cpus   = 8
  end
end
