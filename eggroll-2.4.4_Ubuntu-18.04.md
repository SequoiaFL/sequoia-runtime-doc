# Deploy Eggroll 2.4.4 on Ubuntu 18.04

## Prerequisite

Each node should have no less than **8GB** RAM.

1. (**optional**) Install some commonly used packages.

   ```shell
   sudo apt install curl git htop neovim ssh zsh
   ```

2. Create a new group *apps* and a new user *app* by the following commands.

   ```shell
   sudo addgroup apps
   sudo adduser --ingroup apps app
   ```

3. Install required packages by the following commands.

   ```shell
   sudo apt install build-essential autoconf automake bzip2 libbz2-dev libgmp-dev libaio-dev libasan5 libffi-dev libmpc-dev librocksdb-dev libtool liblz4-dev maven libmpfr-dev numactl libssl-dev python3-snappy libsnappy-dev supervisor zlib1g zlib1g-dev
   ```

   Here is the checklist.

   - [x] autoconf
   - [ ] autoconfig (**???** did not find this package)
   - [x] automake
   - [x] bzip2
   - [x] bzip2-devel
   - [x] gcc
   - [x] gcc-c++
   - [x] gmp-devel
   - [x] libaio
   - [x] libasan
   - [x] libffi-dev
   - [x] libmpc-devel
   - [x] librocksdb-dev (required by [python-rocksdb](https://python-rocksdb.readthedocs.io/en/latest/installation.html#with-distro-package-and-pypi))
   - [x] libtool
   - [x] lz4-devel
   - [x] make
   - [x] maven (required by the packaging process)
   - [x] mpfr-devel
   - [x] numactl
   - [x] openssl-devel
   - [x] snappy
   - [x] snappy-devel
   - [x] supervisor
   - [x] zlib
   - [x] zlib-devel

4. Install JDK 1.8 and verify by the following commands.

   ```shell
   sudo apt install openjdk-8-jdk
   sudo update-alternatives --config java
   sudo update-alternatives --config javac
   java -version
   javac -version
   ```

   After installation, export `JAVA_HOME` for the user *app* by the following command (**you may want to check the real path on your own system**).

   ```shell
   echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> /home/app/.bashrc
   ```

5. Install Python 3.6 and virtualenv by the following commands.

   ```shell
   sudo apt install libpython3.6 libpython3.6-dev libpython3.6-minimal libpython3.6-stdlib python3.6 python3.6-distutils python3-pip python3-virtualenv
   ```

   After installation, change to user *app* by executing `su app`. Then execute the following commands to create and activate virtual environment (**you may want to change the name of your own venv**).

   ```shell
   # python3.6 -m venv ~/.venv_eggroll
   python3.6 -m virtualenv --python python3.6 ~/.venv_eggroll
   source ~/.venv_eggroll/bin/activate
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

   After writing the file. Execute the following commands to install required packages.

   ```shell
   pip3 install -r ~/.venv_eggroll/requirements.txt
   ```

6. Change back to the *admin user*, install and start MySQL server 8.0 by the following commands. More info [here](https://www.digitalocean.com/community/tutorials/how-to-install-the-latest-mysql-on-ubuntu-18-04).

   ```shell
   mkdir ~/tmp
   cd ~/tmp
   curl -OL https://dev.mysql.com/get/mysql-apt-config_0.8.23-1_all.deb
   sudo dpkg -i mysql-apt-config*
   sudo rm -r /var/lib/mysql
   sudo apt install mysql-server
   sudo systemctl start mysql.service
   ```

## Deploy

### Configure MySQL (Optional)

Configure MySQL if it's a fresh installation. To proceed, change back to the *admin user*. Then enter MySQL monitor as root by `sudo mysql` and change the root user’s authentication method to one that uses a password by the following command. Then `exit` MySQL monitor.

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
SELECT User, Host, plugin FROM mysql.user;
```

After changing the authentication method, run the security script by the following command.

```shell
sudo mysql_secure_installation
```

Then enter MySQL monitor as root by `mysql -u root -p` and change the root user's authentication method by the following command. Then `exit` the MySQL monitor.

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
```

To create a dedicated MySQL user and grant privileges, first enter MySQL monitor by `sudo mysql` and execute the following commands. Then `exit` the MySQL monitor.

```sql
CREATE USER 'app'@'localhost' IDENTIFIED BY 'sequoia';
GRANT CREATE, ALTER, DROP, INSERT, INDEX, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'app'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### Pack Eggroll

Change to user *app*. Create a workspace directory and clone the project (**you may want to use the real path of your workspace on your own system**). The following commands can be a reference.

```shell
mkdir ~/workspace
cd ~/workspace
git clone -b main https://github.com/WeBankFinTech/Eggroll.git
```

After cloning, do packing by the following commands.

```shell
cd ~/workspace/Eggroll/deploy
bash auto-packaging.sh
```

After packing, move the `eggroll.tar.gz` to the deploy directory and then unpack (**you may want to use the real path of your deploy directory on your own system**). The following commands can be a reference.

```shell
cd ~/workspace
mkdir Eggroll_deploy
mv ~/workspace/Eggroll/eggroll.tar.gz ~/workspace/Eggroll_deploy/
cd ~/workspace/Eggroll_deploy
tar -xzf eggroll.tar.gz
```

### Edit Config Files

Edit config files. Change to user *app*, then change the working directory to Eggroll's deploy directory.

1. Edit `conf/eggroll.properties` as described [here](https://github.com/WeBankFinTech/eggroll/blob/v2.x/deploy/Eggroll%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E.md#32--%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6). **Below are some tips**.

   - Check MySQL port by the following command in MySQL monitor.

     ```sql
     show global variables like 'port';
     ```

   - Better make the `eggroll.resourcemanager.clustermanager.port` and `eggroll.resourcemanager.nodemanager.port` different.

2. Edit `conf/route_table.json` as described [here](https://github.com/WeBankFinTech/eggroll/blob/v2.x/deploy/Eggroll%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E.md#32--%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6). **Below are some tips**.

   - Make sure the `party id`, `port` and `ip` match the configs in `eggroll.properties`.

3. Edit `conf/create-eggroll-meta-tables.sql` as described [here](https://github.com/WeBankFinTech/eggroll/blob/v2.x/deploy/Eggroll%E9%83%A8%E7%BD%B2%E6%96%87%E6%A1%A3%E8%AF%B4%E6%98%8E.md#32--%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6). Then execute the following commands in MySQL monitor as user *app* (**you may want to use the real `ip` and `port` that match your config**).

   ```sql
   source ~/workspace/Eggroll_deploy/conf/create-eggroll-meta-tables.sql;
   INSERT INTO server_node (host, port, node_type, status) values ('$cluster_ip', '$cluster_port', 'CLUSTER_MANAGER', 'HEALTHY');
   INSERT INTO server_node (host, port, node_type, status) values ('$node_ip', '$node_port', 'NODE_MANAGER', 'HEALTHY');
   SELECT * FROM server_node;
   ```

### Start Services

Change the working directory to Eggroll's working directory. Then execute the following commands.

```shell
bash bin/eggroll.sh all start
bash bin/eggroll.sh all status
```

Use the following command to verify open ports.

```shell
ss -ltnp
```

## Test

### Test Environment Init

When first-time login to the server, execute the following commands.

```shell
export EGGROLL_HOME=/home/app/workspace/Eggroll_deploy
export PYTHONPATH=${EGGROLL_HOME}/python
source ~/.venv_eggroll/bin/activate
echo $EGGROLL_HOME
echo $PYTHONPATH
```

### Eggroll Self Tests

#### Test `roll_pair`

It is recommended **not** to use the standalone test script because it seems to use the same port for both the cluster manager and the node manager, which is not recommended in our previous setup.

```shell
cd ${EGGROLL_HOME}/python/eggroll/roll_pair/test
python -m unittest test_roll_pair.TestRollPairCluster
```

#### Test `roll_site`

##### Communication Test

1. Guest Test

   ```shell
   cd ${EGGROLL_HOME}/python/eggroll/roll_site/test
   python -m unittest test_roll_site.TestRollSiteCluster.test_remote
   ```

2. Host Test

   ```shell
   cd ${EGGROLL_HOME}/python/eggroll/roll_site/test
   python -m unittest test_roll_site.TestRollSiteCluster.test_get
   ```

##### Multi-Partition Communication Test

1. Guest Test

   ```shell
   cd ${EGGROLL_HOME}/python/eggroll/roll_site/test
   python -m unittest test_roll_site.TestRollSiteCluster.test_remote_rollpair_big
   ```

2. Host Test

   ```shell
   cd ${EGGROLL_HOME}/python/eggroll/roll_site/test
   python -m unittest test_roll_site.TestRollSiteCluster.test_get_rollpair_big
   ```

##### `roll_pair` Communication Test

1. Guest Test

   ```shell
   cd ${EGGROLL_HOME}/python/eggroll/roll_site/test
   python -m unittest test_roll_site.TestRollSiteCluster.test_remote_rollpair
   ```

2. Host Test

   ```shell
   cd ${EGGROLL_HOME}/python/eggroll/roll_site/test
   python -m unittest test_roll_site.TestRollSiteCluster.test_get_rollpair
   ```

### Sequoia Demo

```shell
export EGGROLL_HOME=/home/app/workspace/Eggroll_deploy
cd python/fate_script2/test
python -m unittest test_hetero_logistic_regression_rtw.TestHeteroLREncryptedRTW
```

```python
import sys
sys.path.insert(0, "/home/app/workspace/Eggroll_deploy/python")
sys.path.insert(1, "/home/app/workspace/sequoia-demo/python")
```
