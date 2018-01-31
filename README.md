# HadoopSetup
Greetings!
1. Update your ubunto

> sudo apt-get update

2. Install Java Development Kit 

> sudo apt-get install default-jdk

Now we will change the our bash file so some settings are automatically set when we start Ubunto.
> sudo gedit /etc/bash.bashrc

A text editor pops up.  At the end of the editor type the following:
> export JAVA_HOME /usr/lib/jvm/default-java
> PATH=$PATH:$JAVA_HOME/bin
