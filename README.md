IBMSOE: Hadoop
=============

IBMSOE Hadoop hosts buildable Hadoop source trees optimized for 
Linux on POWER. These trees support both RedHat Enterprise Linux
(RHEL) v6.5 on big-endian with PowerVM and Ubuntu v14.04 on 
litte-endian with PowerKVM. This repository has also been tested
on PowerLinux 7 running RHEL 7.1.

The repository is _work in progress_; that is it will evolve over
time, with new versions and additional patches. The source trees
are provided 'as-is' without any warranty whatsoever. 

The tree is a clone from
[Apache Hadoop](http://hadoop.apache.org "Hadoop"). For more
information about Hadoop visit the Apache site or the
[Hadoop Wiki](http://wiki.apache.org/hadoop/).

This software carries the same Apache licence version2 as the original
without any modification. See the
[licence](http://apache.org/licenses/LICENSE-2.0.html) for details.

Information regarding Linux on Power can be found at the
[Developer Works Linux on Power Community](https://www.ibm.com/developerworks/community/groups/service/html/communityview?communityUuid=fe313521-2e95-46f2-817d-44a4f27eba32)

You can follow IBM Power Linux on [twitter](https://twitter.com/ibmpowerlinux)

##Cryptography
Be aware that this code contains cryptographic libraries the use,
import or export of which is controlled in in many places.

##Current version
The tree currently matches Hadoop 2.4.1 release.

# Building Hadoop

## Prerequisites

The following steps have been used to build hadoop on PowerLinux 7
running RedHat Enterprise Linux (RHEL) 7.1 with OpenJDK Java (more
information later).

### Required Packages

The following packages are required to build hadoop and the native libraries
from source:

```bash
$ sudo yum install -y gcc-c++ make openssl openssl-devel openssh \
    libtool autoconf automake cmake xz xz-devel zlib zlib-devel \
    bzip2 bzip2-libs lbzip2 lbzip2-utils
```


### Java

Hadoop requires JDK 1.6+ in order to be built and run. Install the latest
OpenJDK from the package manager, i.e.:

```bash
$ sudo yum install -y java-1.7.0-openjdk java-1.7.0-openjdk-devel
```

Add the following lines to your `~/.bashrc` file after changing the location of the JDK:

```bash
# JAVA
export JAVA_HOME="/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.85-2.6.1.2.el7_1.ppc64"
export JRE_HOME="${JAVA_HOME}/jre"
```

*Make sure to change the absolute path to your installed path. You can create a symlink to the Java
directory using the `update-alternatives` tool.*

The output of the following commands should be similar to:

```bash
$ source ~/.bashrc
$ java -version
java version "1.7.0_85"
OpenJDK Runtime Environment (rhel-2.6.1.2.el7_1-ppc64 u85-b01)
OpenJDK 64-Bit Server VM (build 24.85-b03, mixed mode)
```

### Maven & Ant

[Apache Maven](http://maven.apache.org/) is required to build
hadoop from source. Download it from
[here](https://maven.apache.org/download.cgi) and (for version 3.3.3):

```bash
$ tar xzf apache-maven-3.3.3-bin.tar.gz
$ sudo cp -R apache-maven-3.3.3 /usr/local/apache-maven/
```

Edit your `~/.bashrc` file and add the following lines:

```bash
# Maven
export M2_HOME="/usr/local/apache-maven/apache-maven-3.3.3"
export M2="${M2_HOME}/bin"
export MAVEN_OPTS="-Xms256m -Xmx512m"
export PATH=${M2}:${PATH}
```

The output of the following commands should be similar to:

```bash
$ source ~/.bashrc
$ mvn -version
Apache Maven 3.3.3 (7994120775791599e205a5524ec3e0dfe41d4a06; 2015-04-22T07:57:37-04:00)
Maven home: /usr/local/apache-maven/apache-maven-3.3.3
Java version: 1.7.0_85, vendor: Oracle Corporation
Java home: /usr/lib/jvm/java-1.7.0-openjdk-1.7.0.85-2.6.1.2.el7_1.ppc64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-229.7.2.el7.ppc64", arch: "ppc64", family: "unix"
```

Install [Apache Ant](http://ant.apache.org):

```bash
$ sudo yum install -y ant
```

### <a name="snappy">Snappy</a>

Hadoop utilizes the [snappy](https://github.com/google/snappy) compression
library, which can be installed either through a package manager, or from
source:

```bash
$ wget https://github.com/google/snappy/releases/download/1.1.3/snappy-1.1.3.tar.gz
$ tar xzf snappy-1.1.3.tar.gz  && cd snappy-1.1.3
$ ./configure
$ make
$ sudo make install
```

### Protobuf

Hadoop makes use of [Protobuf](https://github.com/google/protobuf) as well. Version
2.4.1 is needed to correctly build hadoop from source; you can build it from source
using:

```bash
$ wget https://github.com/google/protobuf/archive/v2.5.0.tar.gz
$ tar xzf v2.5.0.tar.gz && cd protobuf-2.5.0
$ ./autogen.sh
$ ./configure --prefix=/usr
$ make
$ sudo make install
$ cd java
$ mvn install
$ mvn package
```

For PowerPC (PowerLinux) architectures, this will most likely fail (verified on
PowerLinux 7, RHEL 7.1). In such case, hadoop will also work with the default
package manager versions:

```bash
$ sudo yum install -y protobuf protobuf-devel protobuf-c \
    protobuf-c-devel protobuf-compiler protobuf-java protobuf-python git
```

*Not all packages are required, but not confident about which will fail if missing.*

If the installation of Protobuf is successful you should be able to see the
following output when running:

```bash
$ protoc --version
libprotoc 2.5.0
```

## Building

First clone the IBM github repository:

```bash
$ git clone https://github.com/ibmsoe/hadoop-common.git
$ cd hadoop-common
```

Now all we have to do is run the following command, in order to build hadoop and create
a tarball and the native binaries and libraries:

```bash
mvn -X package -Pdist,native -Drequire.snappy -Drequire.openssl -DskipTests -Dtar
```

Where:
* `-X`: debug output (verbose)
* `-P`: `dist` in order to build the distribution of hadoop
* `-P`: `native` in order to build the native libraries
* `-Drequire.snappy`: build will fail if snappy is not available
* `-Drequire.openssl`: build will fail if openssl is not available
* `-Dtar`: create a tarball

If you [built Snappy from source](#snappy) then you will need to append the
following option, provided that you did not change the default install location
(using a `--prefix=` option during the `./configure` step):

`-Dsnappy.prefix=/usr/local`: the location of `snappy`

Therefore, the command to build hadoop would be:

```bash
mvn -X package -Pdist,native -Drequire.snappy -Drequire.openssl -DskipTests -Dtar -Dsnappy.prefix=/usr/local
```

For more options and information, please see the
[`BUILDING.txt`](https://github.com/ibmsoe/hadoop-common/blob/branch-2.4.1/BUILDING.txt)
file in this repository.

## Done?

If everything worked, you should be able to find the following files:

```bash
$ ls -lh hadoop-dist/target/
total 376M
drwxr-xr-x. 2 root root   27 Jul 28 11:03 antrun
-rw-r--r--. 1 root root 1.6K Jul 28 13:25 dist-layout-stitching.sh
-rw-r--r--. 1 root root  636 Jul 28 13:25 dist-tar-stitching.sh
drwxr-xr-x. 9 root root   87 Jul 28 13:25 hadoop-2.4.1
-rw-r--r--. 1 root root 126M Jul 28 13:25 hadoop-2.4.1.tar.gz
-rw-r--r--. 1 root root 2.7K Jul 28 11:03 hadoop-dist-2.4.1.jar
-rw-r--r--. 1 root root 251M Jul 28 13:25 hadoop-dist-2.4.1-javadoc.jar
drwxr-xr-x. 2 root root   50 Jul 28 11:04 javadoc-bundle-options
drwxr-xr-x. 2 root root   27 Jul 28 11:03 maven-archiver
drwxr-xr-x. 2 root root    6 Jul 28 11:03 test-dir
```

To verify that the build process was successful, you should be able to
see similar output to the following (or at least `true` next to the
libraries):

```bash
$ hadoop-dist/target/bin/hadoop checknative -a
15/08/12 13:52:41 INFO bzip2.Bzip2Factory: Successfully loaded & initialized native-bzip2 library system-native
15/08/12 13:52:41 INFO zlib.ZlibFactory: Successfully loaded & initialized native-zlib library
Native library checking:
hadoop: true /usr/lib/hadoop/lib/native/libhadoop.so
zlib:   true /lib64/libz.so.1
snappy: true /lib64/libsnappy.so.1
lz4:    true revision:99
bzip2:  true /lib64/libbz2.so.1
```

As you can imagine, `hadoop-2.4.1.tar.gz` is our final built version
of hadoop, native to the Power architecture. Either extract that, or
simply copy the `hadoop-2.4.1` directory to your preferred location.
For example:

```bash
$ sudo cp -R hadoop-dist/target/hadoop-2.4.1 /usr/lib/hadoop
$ sudo chown -R hadoop:hadoop /usr/lib/hadoop
```

Now you should be ready to roll!


## To run the built-in tests
To just run the tests use:

```bash
$ mvn test -Pnative -Drequire.snappy -Drequire.openssl -DskipTests
 ```
 
or omit the `-DskipTests` option while building hadoop.

*This process will take quite a while.*
	
The `-Pnative` profile switch sets the build for JNI.
To build without JNI omit this option.

# Configuring and running Hadoop

Hadoop cluster configuration is beyond the scope of this Readme.
Refer to the [Apache wiki](http://wiki.apache.org/hadoop/). Several
articles can be found on the web.
