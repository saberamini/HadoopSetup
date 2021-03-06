## Setting up a Virtual Machine

Download the Oracle VM VirtualBox Manager from https://www.virtualbox.org/

Download Ubuntu Desktop: https://www.ubuntu.com/download/desktop

In the Oracle VM, choose new and Ubuntu, 64 bit edition (likely) - you can search for ubuntu and the system should automatically choose 64 bit version for you.

If virtual box only detects 32 bit and you have a 64 bit machine you must fix this.  One possible solution is:

- reboot your machine

- get to bios settings

- choose "Intel Virtualization Technologies"

If you need more details see

http://www.sysprobs.com/disable-enable-virtualization-technology-bios


On the virtual box prompts, choose default values and click continue.  For space, choose 25 GB for hard drive if you have it, 8G for memory (if you have it).  You can probably get away with as low as 10 GB of hard drive and 1 GB of memory.

Choose to Install Ubuntu <b>not</b> Try Ubuntu

- Choose "Download Updates while Installing Ubuntu" and click Continue 
- Next screen choose "Erase disk and install Ubuntu" and click Install Now and Continue on warning window.
- Choose location 
- For keyboard layout choose English (US) - English (US) and press Continue
- Call computer name "HadoopNode". <b> You can choose a different name but remember for the following instructions you then need to use your computer name instead of "HadoopNode"</b>
- Choose a username and password (remember your password!).
- Do <b>not</b> choose "Log in automatically"
- Once installation completes, click "Restart now"
- Press any key on the prompt to "Remove the installation key and press Enter"

After install reboot.

Update your Ubuntu (this may take a few minutes):

> sudo apt-get update

Install Java Development Kit (this may take a few minutes):

> sudo apt-get install default-jdk

You will get a message asking you to confirm the new installs.  Type "Y" and press enter to confirm.

Now we will change the our bash file so the Java settings are automatically set on booting.

> sudo gedit /etc/bash.bashrc

A text editor pops up.  If you see any warnings in the the terminal (where you typed the above) do not worry about them.  Generally, we do not care much about warnings in most things we do in the Linux environment.  

Click on the very last line of the file (after all the text), click enter and type the following:

> export JAVA_HOME=/usr/lib/jvm/default-java

> PATH=$PATH:$JAVA_HOME/bin

For those new to the linux-based operating systems, the above two commands set the folder path for executing java-based commands.  We will use java later on to do MapReduce tasks.  

We need to reboot the system to have above changes take effect.  You can also source your bash file, but I find that this can sometime lead to problems.  To reboot the system, enter the following command:

> sudo reboot

Log in with your username and password.

Open a terminal and check to make sure Java is installed:

> java -version

You should see something like this:

```
openjdk version "1.8.0_03-Ubuntu"
OpenJDK Runtime Environment (build 1.8.0_03-Ubuntu-8u77-b03-3ubuntu3-b03)
OpenJDK 64-Bit Server VM (build 25.03-b03, mixed mode)''
```

Check to make sure you have a Java Home variable

>echo $JAVA_HOME

You should see the path you set in your bashrc file:

```
/usr/lib/jvm/default-java
```

We are now ready to move on to adding a Hadoop user and group.

## Adding a Hadoop User and Group

Add a group called hadoop

> sudo addgroup hadoop

Add a user within this group called hduser

> sudo adduser --ingroup hadoop hduser

As with all linux command, you will see a log of what is happening with each command.  You should see something like this:
```
Adding user `hduser' ...
Adding new user `hduser' (1001) with group `hadoop' ...
Creating home directory `/home/hduser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
```

Choose a password for this user ("user")

Leave rest of information empty (just press enter) as shown below:

```
passwd: password updated successfully
Changing the user information for hduser
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] Y
```
Now we want to add hduser to the sudo group or the super user group. 

> sudo adduser hduser sudo

Check to make sure this user has been added. Switch users using "su" command which stands for switch user:

> su hduser

Enter the password you created for this user

You should see your prompt changing to hduser@HadoopNode (or whatever your machine name is called).

Check to see what groups this user (hduser) belongs to:

> groups

You should see the two groups you added above (hadoop and sudo)

```
hadoop sudo
```

Reboot your machine again

> sudo reboot

### Your login should be hduser.  If it your username, after logging in switch to hduser!

> su hduser

Your terminal should look like this:

<img src="Loginscreen.jpg" alt="Ubuntu login as hduser">

## Install Passwordless SSH and RSYNC

<b>Secure Shell (SSH)</b> is a cryptographic network protocol for operating network services securely over an unsecured network. The best known example application is for remote login to computer systems by users.

<b>rsync</b> is a utility for efficiently transferring and synchronizing files across computer systems, by checking the timestamp and size of files. It is commonly found on Unix-like systems and functions as both a file synchronization and file transfer program. The rsync algorithm is a type of delta encoding, and is used for minimizing network usage. Zlib may be used for additional compression, and SSH or stunnel can be used for data security.

We need both of these to setup our Hadoop node.

First install the ssh server

> sudo apt-get install openssh-server

Confirm that you want to continue.  You should see a long output similar to this.  If you get any warnings or err
```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  ncurses-term openssh-client openssh-sftp-server ssh-import-id
Suggested packages:
  ssh-askpass libpam-ssh keychain monkeysphere rssh molly-guard
The following NEW packages will be installed:
  ncurses-term openssh-client openssh-server openssh-sftp-server ssh-import-id
0 upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,222 kB of archives.
After this operation, 8,927 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 http://ca.archive.ubuntu.com/ubuntu xenial/main amd64 openssh-client amd64 1:7.2p2-4 [586 kB]
Get:2 http://ca.archive.ubuntu.com/ubuntu xenial/main amd64 ncurses-term all 6.0+20160213-1ubuntu1 [249 kB]
Get:3 http://ca.archive.ubuntu.com/ubuntu xenial/main amd64 openssh-sftp-server amd64 1:7.2p2-4 [38.7 kB]
Get:4 http://ca.archive.ubuntu.com/ubuntu xenial/main amd64 openssh-server amd64 1:7.2p2-4 [338 kB]
Get:5 http://ca.archive.ubuntu.com/ubuntu xenial/main amd64 ssh-import-id all 5.5-0ubuntu1 [10.2 kB]
Fetched 1,222 kB in 2s (516 kB/s)   
Preconfiguring packages ...
Selecting previously unselected package openssh-client.
(Reading database ... 177004 files and directories currently installed.)
Preparing to unpack .../openssh-client_1%3a7.2p2-4_amd64.deb ...
Unpacking openssh-client (1:7.2p2-4) ...
Selecting previously unselected package ncurses-term.
Preparing to unpack .../ncurses-term_6.0+20160213-1ubuntu1_all.deb ...
Unpacking ncurses-term (6.0+20160213-1ubuntu1) ...
Selecting previously unselected package openssh-sftp-server.
Preparing to unpack .../openssh-sftp-server_1%3a7.2p2-4_amd64.deb ...
Unpacking openssh-sftp-server (1:7.2p2-4) ...
Selecting previously unselected package openssh-server.
Preparing to unpack .../openssh-server_1%3a7.2p2-4_amd64.deb ...
Unpacking openssh-server (1:7.2p2-4) ...
Selecting previously unselected package ssh-import-id.
Preparing to unpack .../ssh-import-id_5.5-0ubuntu1_all.deb ...
Unpacking ssh-import-id (5.5-0ubuntu1) ...
Processing triggers for man-db (2.7.5-1) ...
Processing triggers for systemd (229-4ubuntu19) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for ufw (0.35-0ubuntu2) ...
Setting up openssh-client (1:7.2p2-4) ...
Setting up ncurses-term (6.0+20160213-1ubuntu1) ...
Setting up openssh-sftp-server (1:7.2p2-4) ...
Setting up openssh-server (1:7.2p2-4) ...
Creating SSH2 RSA key; this may take some time ...
2048 SHA256:sO8RnyCr500ieKjbKT9DSB0Cg9AypWxE38n0flNG7JI root@HadoopNode (RSA)
Creating SSH2 DSA key; this may take some time ...
1024 SHA256:IRuyXV9pa5fK9iP5lSnMPYYZPt4pfEgzluqF2rse8ls root@HadoopNode (DSA)
Creating SSH2 ECDSA key; this may take some time ...
256 SHA256:1X9eQmEqHzjzJTrxfsfilSXh6zfuDdGjVCzJ5BPG+TI root@HadoopNode (ECDSA)
Creating SSH2 ED25519 key; this may take some time ...
256 SHA256:IH0LXR6D8KtUnp9vvnRIMSZl3fqIC1iT+grYaawnA3M root@HadoopNode (ED25519)
Setting up ssh-import-id (5.5-0ubuntu1) ...
Processing triggers for systemd (229-4ubuntu19) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for ufw (0.35-0ubuntu2) ...
```

Next we will generate an RSA key

> ssh-keygen -f ~/.ssh/id_rsa -t rsa -P ""

You should get something like this

```
Generating public/private rsa key pair.
Created directory '/home/hduser/.ssh'.
Your identification has been saved in /home/hduser/.ssh/id_rsa.
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Le6t3zfoeRgzbDGbsIVkhYwQqFOkIhOmmwuunRRiQg0 hduser@HadoopNode
The key's randomart image is:
+---[RSA 2048]----+
|.E ...oo o o.    |
|o.o.o   . =      |
|=..+     o .     |
|o=o      .o +    |
|*...    S .= =   |
|=o .   . .. O    |
|...     .  . *   |
|.o .   . . .o.+  |
|. o     ooo.+o . |
+----[SHA256]-----+
```

Copy the RSA Security to .SSH Folder

> ssh-copy-id -i hduser@[computername]

where [computername] is the name you chose for your machine (without [ ] - it is name that appears after the @ sign).  Name chosen for this tutorial was HadoopNode.

You will get a warning that the autenticy of the host cannot be established and whether you want to continue.  Type "yes" and press enter.

```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/hduser/.ssh/id_rsa.pub"
The authenticity of host 'hadoopnode (127.0.1.1)' can't be established.
ECDSA key fingerprint is SHA256:1X9eQmEqHzjzJTrxfsfilSXh6zfuDdGjVCzJ5BPG+TI.
Are you sure you want to continue connecting (yes/no)? yes 
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
hduser@hadoopnode's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'hduser@HadoopNode'"
and check to make sure that only the key(s) you wanted were added.
```


We basically have added hduser as a user which can log remotely through SSH




Now try to ssh to your computer

> ssh [computername]

where [computername] is the name you gave your machine (without [ ] - it is the name that appears after the @ sign)
Can also use localhost which is the same in this case

> ssh localhost

Type "yes" if prompted "Are you sure you want to connect?" .

We have made a lot of change, so better to reboot to allow these changes to take effect.

> sudo reboot

## Again make sure to login as hduser by choosing hduser at the log-in screen.

If you log-in with your own username, just change to hduser using the "su" command.

> su hduser

Let's install Rsync (probably already done but to make sure)

>sudo apt-get install rsync

## Install and Configure Hadoop on a Single Node Cluster

Hadoop has a very aggressive release cycle.  Generally it is preferrable to choose a release that is not the latest because it may have bugs or other problems associated with it that will be fixed overtime. 
All the information regarding releases can be found at http://hadoop.apache.org/

You can use the following link to find an appropriate distribution: http://apache.forsale.plus/hadoop/common

At the time of writing this tutorial, Hadoop version 3.0 had just come out but we will use version 2.9 which has been more rigorously tested and fine turned (So hopefully we will not run into any unexpected problems).

Download version 2.9 using the following command:

> wget http://apache.forsale.plus/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz -P ~/Downloads/Hadoop

Depending on your internet connection speed, this might take several minutes.
```
--2018-02-19 12:51:08--  http://apache.forsale.plus/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz
Resolving apache.forsale.plus (apache.forsale.plus)... 158.69.246.173
Connecting to apache.forsale.plus (apache.forsale.plus)|158.69.246.173|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 366744329 (350M) [application/x-gzip]
Saving to: ‘/home/hduser/Downloads/Hadoop/hadoop-2.9.0.tar.gz’

hadoop-2.9.0.tar.gz 100%[===================>] 349.75M  1.35MB/s    in 3m 48s  

2018-02-19 12:54:56 (1.53 MB/s) - ‘/home/hduser/Downloads/Hadoop/hadoop-2.9.0.tar.gz’ saved [366744329/366744329]
```

Uncompress the Hadoop tar file into the /usr/local folder

> sudo tar zxvf ~/Downloads/Hadoop/hadoop-*.tar.gz -C /usr/local

Rename the folder hadoop-2.9 to just hadoop (just for esthetics)

> sudo mv /usr/local/hadoop-* /usr/local/hadoop

## Setting Environment Variables

Now once again we need to modify our .bashrc file

> sudo gedit ~/.bashrc

Do not worry about any warnings that may come up on the terminal.  

In the file that opens, at the very end type the following:

```
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_DATA_HOME=/home/$USER/hadoop_data/hdfs
PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

Now to have to these changes take into effect, we need to reboot our machine (as we did previously).  However, another way to manually run the script in .bashrc

> source ~/.bashrc

Generally, however, to be safe, you are better off just rebooting your machine (to be technical, there may be dependencies that are not sourced by just using the above).

> sudo reboot

For a sanity check, see if the environment variables have actually been updated

> echo $HADOOP_HOME

## Build the Hadoop data Directories

Almost there but first we have to create some hadoop data directories for the hduser.

> mkdir -p $HADOOP_DATA_HOME/namenode

> mkdir -p $HADOOP_DATA_HOME/datanode

> mkdir -p $HADOOP_DATA_HOME/tmp

## Hadoop Configuration Files

MOdify permissions on the HADOOP_CONF_DIR to allow you to edit the configuration files.

> sudo chown root -R $HADOOP_HOME

Set the read/write permissions (Wide open for now).

> sudo chmod 777 -R $HADOOP_HOME

Add JAVA_HOME variable to $HADOOP_CONF_DIR/hadoop-env.sh

> sudo gedit $HADOOP_CONF_DIR/hadoop-env.sh

Locate the area in the file that indicates the current JAVA_HOME variable, it should look like:

> export JAVA_HOME=$(JAVA_HOME)

Change it to

> export JAVA_HOME=/usr/lib/jvm/default-java

Next modify $HADOOP_CONF_DIR/core-site.xml.  Use this command to open up the xml file

> sudo gedit $HADOOP_CONF_DIR/core-site.xml

Add the following lines to the configuration section of the core-site.xml file

```
 <configuration>  
  <property>  
    <name>hadoop.tmp.dir</name>
    <value>/home/${user.name}/hadoop_data/hdfs/tmp/</value>
    <description> A base for other temporary directories. </description>
  </property>
  <property>  
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
    <description> localhost may be replaced with a DNS that points to the NameNode. </description>
  </property>
</configuration>
```

All this is doing is telling Hadoop where things are found.

Now modify $HADOOP_CONF_DIR/yarn-site.xml

> sudo gedit $HADOOP_CONF_DIR/yarn-site.xml

Add the following lines to the configuration section of the yarn-site.xml file.

```
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name> mapred.job.tracker</name>
    <value> localhost:9001</value>
  </property>
  <!--property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>-->
</configuration>
```

Some of these things you don't need but keep it for now.

We now need to create a mapred-site.xml file. 

Copy the mapred-site.xml template and rename the new file mapred-site.xml

> sudo cp $HADOOP_CONF_DIR/mapred-site.xml.template $HADOOP_CONF_DIR/mapred-site.xml

Change permissions

> sudo chmod 777 $HADOOP_CONF_DIR/mapred-site.xml

Open the file

> sudo gedit $HADOOP_CONF_DIR/mapred-site.xml

Edit the file as follows:

```
<configuration>
  <property>
    <name>mapreduce.jobtracker.address</name>
    <value>local</value>
  </property>
  
  <property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
  </property>
</configuration>
```

Modify $HADOOP_CONF_DIR/hdfs-site.xml

> sudo gedit $HADOOP_CONF_DIR/hdfs-site.xml

Edit the file as follows:

```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
    <description> Default block replication.
      The actual number of replications can be specified when the file is created.
      The default is used if replication is not specified in create time.
    </description>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///home/hduser/hadoop_data/hdfs/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///home/hduser/hadoop_data/hdfs/datanode</value>
  </property>
  <property>
     <name>dfs.datanode.data.dir</name>
     <value>file:///home/hduser/hadoop_data/hdfs/datanode</value>
  </property>
  <property>. 
     <name>dfs.permissions.enabled</name>
     <value>false</value>
     <description>If "true", enable permission checking in HDF.  If "false", permission checking is turned off, but all other behaviour is unchanged.  Switching from one parameter value to the other does not change the mode, ownder or group of files or directories
     </description>
  </property>
</configuration>
```

We'll return the ownership back to the root on all files in $HADOOP_HOME.  We have made changes so let's just be sure:

> sudo chown root -R $HADOOP_HOME

We will also allow all users to have Read/Write access to your $HADOOP_HOME folder.  <b>NOTE: This is NOT good pracitice in production deployment.</b> 

> sudo chmod 777 -R $HADOOP_HOME

 Format the HDFS

> hdfs namenode -format

Make sure there is no ERRORS during the formatting process.  If there is any, call over the instructor to debug your setup.

We will do a final reboot to have all the changes take effect.
> sudo reboot

## We have now installed and formatted our Hadoop cluster.  Please move on to the tutorial Working with Hadoop to test your setup .
