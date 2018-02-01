## Setting up a Virtual Machine

Download the Oracle VM VirtualBox Manager from https://www.virtualbox.org/
Download Ubuntu Desktop: https://www.ubuntu.com/download/desktop

In the Oracle VM, choose new and Ubuntu, 64 bit edition (likely)

Go through the settings, choose 25 GB for hard drive if you have it, 8G for memory (if you have it)

Choose to Install Ubuntu <b>not</b> Try Ubuntu
- Choose "Download Updates while Installing Ubuntu" and click Continue 
- Next screen choose "Erase disk and install Ubuntu" and click Install Now and Continue on warning window.
- Choose location 
- For keyboard layout choose English (US) - English (US) and press Continue
- Call computer name "HadoopNode"
- Choose a username and password (remember your password!).
- Do <b>not</b> choose "Log in automatically"
- Once installation completes, click "Restart now"
- Press any key on the prompt to "Remove the installation key and press Enter"

After install reboot.

Update your Ubuntu

> sudo apt-get update

Install Java Development Kit 

> sudo apt-get install default-jdk

Now we will change the our bash file so the Java settings are automatically set on booting.

> sudo gedit /etc/bash.bashrc

A text editor pops up.  At the end of the editor type the following:
> export JAVA_HOME=/usr/lib/jvm/default-java

> PATH=$PATH:$JAVA_HOME/bin

Reboot the system
> sudo reboot

Check to make sure Java is installed:

> java -version

Check to make sure you have a Java Home variable
>echo $JAVA_HOME

## Adding a Hadoop User and Group

Add a group called hadoop

>sudo addgroup hadoop

Add a user within this group called hduser

>sudo adduser --ingroup hadoop hduser

Choose a password for this user ("user")
Leave rest of information empty (just press enter)

Add hduser to the sudo group 

> sudo adduser hduser sudo

Check to make sure this user has been added. Switch users using "su" command
> su hduser
Enter the password you created for this user

You should see your prompt changing to hduser@HadoopNode 

Reboot your machine again
> sudo reboot

## Install Passwordless SSH and RSYNC

<b>Secure Shell (SSH)</b> is a cryptographic network protocol for operating network services securely over an unsecured network. The best known example application is for remote login to computer systems by users.

<b>rsync</b> is a utility for efficiently transferring and synchronizing files across computer systems, by checking the timestamp and size of files. It is commonly found on Unix-like systems and functions as both a file synchronization and file transfer program. The rsync algorithm is a type of delta encoding, and is used for minimizing network usage. Zlib may be used for additional compression, and SSH or stunnel can be used for data security.

First install the ssh server
> sudo apt-get install openssh-server

Next we will generate an RSA key
ssh-keygen -f ~/.ssh/id_rsa -t rsa -P ""

You should get something like this

Generating public/private rsa key pair.
Created directory '/home/hduser/.ssh'.
Your identification has been saved in /home/hduser/.ssh/id_rsa.
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:xUWPIP8dcBTFrCHmMATzP410zkt1yOO0zJ3bKfl7tYg hduser@ubuntu
The key's randomart image is:
+---[RSA 2048]----+
|        +oo.+.+=.|
|         *ooo*..o|
|          ==o.Ooo|
|         . +.%.*o|
|        S   = @..|
|             o..=|
|            .ooo+|
|           E .o..|
|               oo|
+----[SHA256]-----+

In your file manager, you should now see a .ssh folder (Press cntrl-h to make hidden filles visible)


Copy the RSA Security to .SSH Folder
> ssh-copy-id -i hduser@HadoopNode

We basically have added hduser as a user which can log remotely through SSH

Now try to ssh to HadoopNode
> ssh HadoopNode

Can also use localhost which is the same in this case
> ssh localhost

We have made a lot of change, so better to reboot to allow these changes to take effect.
> sudo reboot

Let's install Rsync (probably already done)

>sudo apt-get install rsync

## Install and Configure Hadoop on a Single Node Cluster

Hadoop has a very aggressive release cycle.  Generally it is preferrable to chooose a release that is stable
All the information regarding releases can be found at http://hadoop.apache.org/

Use the following link to find an appropriate distribution: http://apache.forsale.plus/hadoop/common

We will use version 2.9 which is the most stable release so far.

Download version 2.9
> wget http://apache.forsale.plus/hadoop/common/hadoop-2.9.0/hadoop-2.9.0.tar.gz -P ~/Downloads/Hadoop

Depending on your internet connection speed, this might take several minutes.

Uncompress the Hadoop tar file into the /usr/local folder

> sudo tar zxvf ~/Downloads/Hadoop/hadoop-*.tar.gz -C /usr/local

Rename the folder hadoop-2.9 to just hadoop (just for esthetics)
>sudo mv /usr/local/hadoop-* /usr/local/hadoop

## Setting Environment Variables

Now once again we need to modify our .bashrc file

> gedit ~/.bashrc

In the file that opens, at the very end type the following:

> export HADOOP_HOME=/sur/local/hadoop

> export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop

> export HADOOP_DATA_HOME=/home/$USER/hadoop_data/hdfs

> PATH = $PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

Now to have to these changes take into effect, we need to reboot our machine (as we did previously).  However, another way to manually run the script in .bashrc

> source ~/.bashrc

Generally, however, to be safe, you are better off just rebooting your machine

> sudo reboot

For a sanity check, see if the environment variables have actually been updated

> echo $HADOOP_HOME

## Build the Hadoop data Directories

Almost there but first we have to create some hadoop data directories for the hduser.

> mkdir -p $HADOOP_DATA_HOME /namenode

> mkdir -p $HADOOP_DATA_HOME /datanode

> mkdir -p $HADOOP_DATA_HOME /tmp

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
<pre>
 <configuration>
  
  <property>
  
    <name>hadoop.tmp.dir</name>

    <value>/home/$(user.name)/hadoop_data/hdfs/tmp/</value>

    <description> A base for other temporary directories. </description>

  </property>

  <property>
  
    <name>fs.defaultFS</name>

    <value>hdfs://localhost:9000</value>

    <description> localhost may be replaced with a DNS that points to the NameNode. </description>

  </property>
</pre>

All this is doing is telling Hadoop where things are found.

Now modify $HADOOP_CONF_DIR/yarn-site.xml

> sudo gedit $HADOOP_CONF_DIR/yarn-site.xml

Add the following lines to the configuration section of the yarn-site.xml file.

<pre>
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
</pre>

Some of these things you don't need but keep it for now.

Modify $HADOOP_CONF_DIR/mapred-site.xml
Copy the mapred-site.xml template and rename the new file mapred-site.xml

> sudo cp $HADOOP_CONF_DIR/mapred-site.xml.template $HADOOP_CONF_DIR/mapred-site.xml

Change permissions

> sudo chmod 777 $HADOOP_CONF_DIR/mapred-site.xml

Open the file

> sudo gedit $HADOOP_CONF_DIR/mapred-site.xml

Edit the file as follows:

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


Modify $HADOOP_CONF_DIR/hdfs-site.xml

> sudo gedit $HADOOP_CONF_DIR/hdfs-site.xml

Edit the file as follows:

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
    
