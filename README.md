## Setting up a Virtual Machine
Greetings!
1. Download the Oracle VM VirtualBox Manager from https://www.virtualbox.org/
2. Download Ubuntu Desktop: https://www.ubuntu.com/download/desktop

3. In the Oracle VM, choose new and Ubuntu, 64 bit edition (likely)

4. Go through the settings, choose 25 GB for hard drive if you have it, 8G for memory (if you have it)

5. Choose to Install Ubuntu <b>not</b> Try Ubuntu
- Choose "Download Updates while Installing Ubuntu" and click Continue 
- Next screen choose "Erase disk and install Ubuntu" and click Install Now and Continue on warning window.
- Choose location 
- For keyboard layout choose English (US) - English (US) and press Continue
- Call computer name "HadoopNode"
- Choose a username and password (remember your password!).
- Choose "Log in automatically"
- Once installation completes, click "Restart now"
- Press any key on the prompt to remove the installation key

6. After install reboot.

7. Update your Ubuntu

> sudo apt-get update

8. Install Java Development Kit 

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

