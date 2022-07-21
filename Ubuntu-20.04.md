# Deploy on Ubuntu 20.04

## Prerequisite

1. Create a new group *apps* and a new user *app* by the following commands.

   ```shell
   sudo addgroup apps
   sudo adduser --ingroup apps app
   ```

2. Install required packages by the following commands.

   ```shell
   sudo apt update
   sudo apt install build-essential autoconf automake bzip2 libbz2-dev libgmp-dev libaio-dev libasan5 libffi-dev libmpc-dev librocksdb-dev libtool liblz4-dev libmpfr-dev numactl libssl-dev python3-snappy libsnappy-dev supervisor zlib1g zlib1g-dev
   ```

   Here is the checklist.

   - [ ] autoconf
   - [ ] autoconfig
   - [ ] automake
   - [ ] bzip2
   - [ ] bzip2-devel
   - [ ] gcc
   - [ ] gcc-c++
   - [ ] gmp-devel
   - [ ] libaio
   - [ ] libasan
   - [ ] libffi-dev
   - [ ] libmpc-devel
   - [ ] librocksdb-dev (required by [python-rocksdb](https://python-rocksdb.readthedocs.io/en/latest/installation.html#with-distro-package-and-pypi))
   - [ ] libtool
   - [ ] lz4-devel
   - [ ] make
   - [ ] mpfr-devel
   - [ ] numactl
   - [ ] openssl-devel
   - [ ] snappy
   - [ ] snappy-devel
   - [ ] supervisor
   - [ ] zlib
   - [ ] zlib-devel

3. Install JDK 1.8 and verify by the following commands.

   ```shell
   sudo apt install openjdk-8-jdk
   java -version
   javac -version
   ```

   After installation, export `JAVA_HOME` for the user *app* by the following command.

   ```shell
   echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> /home/app/.bashrc
   ```

4. Install Python and virtualenv by the following commands.

   ```shell
   sudo apt install python3 python3-pip python3-venv
   ```

   After installation, change to user *app* by executing `su app`. Then execute the following commands.

   ```shell
   python3 -m venv .venv_eggroll
   source .venv_eggroll/bin/activate
   ```

   Create a file `requirements.txt` under `~/.venv_eggroll` by `nvim ~/.venv_eggroll/requirements.txt`. Then write the following content into `requirements.txt`. More info [here](https://github.com/WeBankFinTech/eggroll/blob/main/requirements.txt).

   ```text
   cachetools>=3.0.0
   cloudpickle==0.6.1
   grpcio-tools==1.24.3
   grpcio==1.24.3
   llvmlite==0.33.0
   lmdb==0.98
   mmh3==3.0.0
   numba==0.50.0
   numpy>=1.17.0
   pandas>=1.0.4
   protobuf==3.6.1
   psutil>=5.7.0
   pyarrow==0.17.1
   pystack-debugger>=0.9.0
   python-rocksdb==0.7.0
   ```

   After writing the file. Execute the following commands.

   ```shell
   pip3 install wheel
   pip3 install -r ~/.venv_eggroll/requirements.txt
   ```

5. Install and start MySQL server by the following commands.

   ```shell
   sudo apt install mysql-server
   sudo systemctl start mysql.service
   ```

## Deployment
