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

> gedit .bashrc

In the file that opens
