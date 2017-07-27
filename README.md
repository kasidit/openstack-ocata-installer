# openstack-ocata-installer
The "OPSInstaller" OpenStack Ocata Installation Script 

Copyright 2017 Kasidit Chanchio 

Contact: kasiditchanchio@gmail.com <br>
Department of computer science <br>
Faculty of science and technology <br>
Thammasat University.

You will issue every instruction on the controller node
of your OpenStack deployment. 

First, you have to use virtualbox to create a controller node VM. Then, 
you have to login to the your openstack account on the controller node and invoke 
the following commands to obtain this OpenStack installer script. 
<pre>
$ cd $HOME
$ git clone https://github.com/kasidit/openstack-ocata-installer
$ cd openstack-ocata-installer
</pre>

In our example, we follow the example
configuration parameters in the <a href="http://docs.openstack.org/ocata/install-guide-ubuntu/">official installation manual (using ubuntu)</a> of 
openstack as illustrated in the picture below.<br> 
<img src="documents/architecture.png"> <br>
From the figure, we use the same configuration as those of OpenStack installation 
manual for the managment network, data tunnel network, v-lan network, and external network.
However, this script still require the followings. 
<ul>
<li> The name of the 4 hosts in the figure must be "controller", "network", "compute",
and "compute1". </li>
<li> The username of openstack account on every host must be the same. 
The password of those account must be the same across the hosts as well 
(but it can be different from the username).
<li> You should make sure that the time on the controller node is up-to-date before installation
</ul>
Based on the above configuration and requirements, modify ./install-paramrc.sh file by entering 
environment variables' values that fit your deployment. 
<pre>
$ vi ./install-paramrc.sh
</pre>

You may view an example of this file <a href="./install-paramrc.sh">here</a>. 
After that, run the script below to substitute the parameter values in the script 
template tar file (OPSInstaller-init.tar). <b>PLEASE RUN EVERY SCRIPT DESCRIBED HERE AS 
A USER. DO NOT USE SUDO TO RUN THE SCRIPTS!</b> 

<pre>
$ ./exe-config-installer.sh
</pre>

After running the script, you should see a new directory "OPSInstaller" being created. 
This directory contains all scripts and configuration files that will later be distributed and run 
on every node. The details of every script in the "OPSInstaller" can be seen in the 
<pre>
OPSInstaller/scriptmap.html
</pre> 
file. To view this file, you may have to copy the whole "OPSInstaller" directory out from the 
controller to your PC and use a browser to view the file. 
An example scriptmap.html file is also available at <a href="http://vasabilab.cs.tu.ac.th/presentations/OPSInstaller.example/scriptmap.html">http://vasabilab.cs.tu.ac.th/presentations/OPSInstaller.ocata/scriptmap.html</a>. 

<strong><u>
1. Make the controller a gateway of the 10.0.0.0/24 network
</u></strong>

On the controller node, do the followings. 
<pre>
$ cd $HOME
$ cd openstack-ocata-installer
$ cd OPSInstaller/installer
$ OS-installer-00-0-set-gateway.sh
</pre>
Now you should have the gateway to access internet through 10.0.0.0/24 network. 
<p>
Next, you can create the network, compute, and compute1 VMs. These VMs should be able to access 
internet via its first network interface.  

<strong><u>
2. install OpenStack ocata with classic open vswitch network
</u></strong>

On the controller node. 
<pre>
$ cd $HOME
$ cd openstack-ocata-installer
$ cd OPSInstaller/installer
$ ./OS-installer-00-1...
$ ./OS-installer-00-2...
</pre>
<p>
The last script will reboot all hosts. You have to login to the controller node again and make sure that every other node is up before continue the installation. 
<pre>
$ cd $HOME
$ cd openstack-ocata-installer
$ cd OPSInstaller/installer
$ ./OS-installer-01-...
$ ./OS-installer-02-...
$ ./OS-installer-03-...
$ ./OS-installer-04-...
$ ./OS-installer-05-...
$ ./OS-installer-06-...
$ ./OS-installer-07-...
(Press any key at the end of the installation 07)
$ ./OS-installer-08-...
(skip ./OS-installer-09-...)
</pre>

<strong><u>
3. Install OpenStack ocata and deploy 
Distributed Virtual Router (DVR) network
</u></strong>

<pre>
$ cd $HOME
$ cd openstack-ocata-installer
$ cd OPSInstaller/installer
$ ./OS-installer-00-1...(please fill in the rest of the name)
$ ./OS-installer-00-2...
</pre>
<p>
The last script will reboot all hosts. You have to login to the controller node again and make sure that every other node is up before continue the installation. 
<pre>
$ cd $HOME
$ cd openstack-ocata-installer
$ cd OPSInstaller/installer
$ ./OS-installer-01-...
$ ./OS-installer-02-...
$ ./OS-installer-03-...
$ ./OS-installer-04-...
$ ./OS-installer-05-...
$ ./OS-installer-06-...
$ ./OS-installer-07-...
$ ./OS-installer-08-...
$ ./OS-installer-09-...
$ ./OS-installer-10-...
</pre>

Note: This script is written for educational purpose. 

For more information, please consult the following 
documents: 

1. http://sciencecloud-community.cs.tu.ac.th/ 
2. http://vasabilab.cs.tu.ac.th/ 
3. http://docs.openstack.org/
