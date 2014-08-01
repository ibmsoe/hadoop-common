#IBMSOE: Hadoop#

IBMSOE Hadoop hosts buildable Hadoop source trees optimized for Linux on POWER. These trees support both RedHat Enterprise Linux (RHEL) v6.5 on big-endian with PowerVM and Ubuntu v14.04 on PowerKVM. 

The repository is _work in progress_; that is it will evolve over time, with new versions and additional patches. The source trees are provided 'as-is' without any warranty whatsoever. 

The tree is a clone from [Apache Hadoop](http://hadoop.apache.org "Hadoop"). For more information about Hadoop visit the Apache site or the [Hadoop Wiki](http://wiki.apache.org/hadoop/).

This software carries the same Apache licence version2 as the original without any modification. See the [licence](http://apache.org/licenses/LICENSE-2.0.html) for details.

##Cryptography##
Be aware that this code contains cryptographic libraries the use, import or export of which is controlled in in many places.

##Current version##
The tree currently matches Hadoop 2.4.1 release.


##Prerequisites##
The following components are required prior to building Hadoop:

###Build tool chain###

* [Maven](http://maven.apache.org) Use Maven3 __NOT__ Maven2. This is confusing as Maven v3 is referred to simply as Maven in yum and apt, so it is tempting to think that Maven2 is the more recent version.
* [Ant](http://ant.apache.org)
* cmake
* gcc and g++ (XLC/XLC++ may work but haven't been tested)
* IBM Java JDK. OpenJDK is not usable at the time of writing.
* automake
* autoconf

###Runtime requisites###
* openssl
* openssh
* [libsnappy](http://code.google.com/p/snappy/)
* [snappy-java](https://github.com/xerial/snappy-java) A java port of libsnappy
* zlib
* [protobuf](https://developers.google.com/protocol-buffers/)
* IBM Java run-time. OpenJDK is not usable at the time of writing.

#Building#

Install the [prerequisite packages](#prereqs). On ubuntu `apt-get install build-essential` brings much of what is required. On RedHat it's not quite as simple.

Set the `JAVA_HOME` environment variable to point to your Java installation.

Detailed instructions for building Hadoop are given in the `BUILDING.txt` file in the root directory. For the impatient, here's the short version: 

##Compile and Install into Maven cache##
To compile and install Hadoop in to Maven cache using JNI and snappy use the following build command from the root of the Hadoop installation directory:

`> mvn install -Pnative -DskipTests -Drequire.snappy`
	
The build process on its own will take a few minutes. Without the `-DskipTests` it will the best part of a day to run.

##Create a tar distribution package##
To create a tar containing a Hadoop build use the following command

`> mvn package -Pnative,dist -Drequire.snappy -DskipTests -Dtar`

##To run the built-in tests##
To just run the tests use:

`> mvn test -Pnative -Drequire.snappy` or omit the `-DskipTests` during the compilation, installation or packaging phases.
	
And be prepared to wait.
	
The `-Pnative` profile switch sets the build for JNI. To build without JNI omit this option.

#Configuring and running Hadoop#

Hadoop cluster configuration is beyond the scope of this Readme. Refer to the [Apache wiki](http://wiki.apache.org/hadoop/). Several articles can be found on the web.
