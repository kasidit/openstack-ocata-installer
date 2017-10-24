# openstack-ocata-installer

Copyright 2017 Kasidit Chanchio 

Contact: kasiditchanchio@gmail.com <br>
Department of computer science <br>
Faculty of science and technology <br>
Thammasat University.

<p>
<h3>การติดตั้งระบบ OpenStack Ocata แบบ Multi-node & DVR ด้วย installation scripts บน ubuntu 16.04 </h3> <br>
<p>
ให้ นศ เตรียมเครื่องตามส่วนที่ 1 และหลังจากนั้นเลือกเอาอันใดอันหนึ่งว่าจะติดตั้งด้วย scripts(ส่วนที่ 2) หรือด้วยมือ (ส่วนที่ 3)  
<ul>
 <li> 1. <a href="#part1">เตรียมเครื่องและเนตสำหรับติตดั้ง</a>
      <ul>
       <li> <a href="#kvmhost">1.1 การเตรียมเครื่องเพื่อติดตั้งบน kvm vm หรือเครื่องจริง</a>
       <li> <a href="#vboxhost">1.2 การเตรียมเครื่องสำหรับติดตั้งบน vbox vm</a>
      </ul>
 <li> 2. <a href="#part2">ติดตั้งด้วย scripts</a> 
 <li> 3. <a href="#part3">ติดตั้งด้วยมือ</a> 
</ul>
<p>
<a id="part1"><h4>ส่วนที่ 1: เตรียมเครื่องและเนตสำหรับติดตั้ง</h4></a>
<p>
 เราจะมีการเตรียมการสองแบบคือ การเตรียมการสำหรับการติดตั้งโดยใช้ kvm vm หรือโดยใช้ virtual box vm (vbox)
<p>
 <i><a id="kvmhost">1.1 การเตรียมเครื่องเพื่อติดตั้งบน kvm vm หรือเครื่องจริง</a></i>
<p> 
  นศ จะเตรียมเครื่อง ubuntu 16.04.x จำนวน 4 เครื่องเชื่อมต่อกันบนเนตดังภาพที่ 1 ได้แก่เครื่องชื่อ controller network compute และ compute1 (ชื่อเครื่องต้องตรงกับผลจากคำสั่ง hostname) จากภาพกำหนดให้เครื่องที่ controller มี spec แนะนำคือ cpu 4 cores RAM 6 ถึง 8 GB Disk 16-20 GB เครื่อง network มี cpu 1-2 cores RAM 512MB-1GB Disk 8-10 GB เครื่อง compute และ compute1 มี cpu 4 cores RAM 2-4 GB Disk 16-20 GB (เป็น spec ใช้สำหรับการศึกษา ถ้าจะ deploy ขอให้ดู official OpenStck doc) 
  <p>
  <img src="documents/architecture.png"> <br>
   ภาพที่ 1 <br>
กำหนดให้ทุกเครื่องมี username คือ opensatck และ password คือ openstack และเพื่อความสะดวกแนะนำว่าให้ทำให้ทุกเครื่องใช้ sudo โดยไม่ต้องป้อน password อีกอย่างที่สำคัญคือเครื่องเหล่านี้ควรมีเวลาใกล้เคียงกัน
   
<p>
สำหรับเนต (network) กำหนดให้มีเนตสี่แบบเชื่อมกับเครื่องทั้งสี่ดังภาพได้แก่ 
 <ul>
 <li> management network: มี cidr 10.0.10.0/24 และ gateway คือ 10.0.10.1  openstack ใช้เนตนี้เป็นเนตหลักเพื่อออกอินเตอเนตและส่งคำสั่งระหว่างโหนด(หรือเครื่องทั้ง 4)ต่างๆของมัน 
 <li> data tunnel network: ใช้สร้าง tunnel สำหรับส่งข้อมูลของ vm ที่จะถูกสร้างขึ้นภายใน openstack เนตนี้ใช้สำหรับส่งข้อมูลระว่าง vm กันเอง (east-west) และระหว่าง vm กับ internet (north-south)
 <li> vlan network: ใช้ส่งข้อมูลระหว่าง vm ภายใน openstack กับ vlan network ภายนอก openstack 
 <li> external network: คือเนตที่เป็น internet service provider ของ openstack ในที่นี้เราจะใช้ management network 
 </ul>
จากภาพที่ 1 สมมุตว่า NIC ที่ 1 คือ ens3 NIC ที่ 2 คือ ens4 NIC ที่ 3 คือ ens5 NIC ที่ 4 คือ ens6 จะเห็นว่าเครื่อง conroller มี ens3 อันเดียว เครื่อง network compute แบะ compute1 ทั้งหมด มี ens3 ถึง ens6 
<p>
 ในขั้นต้นให้ นศ กำหนดค่า apt configuration ของเครื่องต่างๆให้ใช้ ubuntu repository ในประเทศไทย โดยกำหนดค่าใน /etc/apt/sources.list ด้วยมือ หรือใช้คำสั่ง 
 <pre>
  sudo sed -i "s/us.arch/th.arch/g" /etc/apt/sources.list
 </pre>
 และให้ นศ กำหนด network configuration ดังตัวอย่างข้างล่าง ซึ่งเป็นการกำหนดค่า IP address ของทุกเครื่งอบน management network โดยที่ทุก interface มี MTU คือ 1500 
<p>
 <b>เครื่อง controller </b> 
<pre>
openstack@controller:~$ cat /etc/network/interfaces
...
auto lo
iface lo inet loopback
...
auto ens3
iface ens3 inet static
address 10.0.10.11
netmask 255.255.255.0
network 10.0.10.0
gateway 10.0.10.1
dns-nameservers 8.8.8.8

openstack@controller:~$
</pre>
<pre>
openstack@controller:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr 00:54:09:25:20:17
          inet addr:10.0.10.11  Bcast:10.0.10.255  Mask:255.255.255.0
          inet6 addr: fe80::254:9ff:fe25:2017/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:17777 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12906 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:5715989 (5.7 MB)  TX bytes:2963058 (2.9 MB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)
openstack@controller:~$
</pre>
<pre>
openstack@controller:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:20:17 brd ff:ff:ff:ff:ff:ff
openstack@controller:~$
</pre>
<p>
 <b>เครื่อง network </b>
<pre>
openstack@network:~$ cat /etc/network/interfaces
...
auto lo
iface lo inet loopback
...
auto ens3
iface ens3 inet static
address 10.0.10.21
netmask 255.255.255.0
network 10.0.10.0
gateway 10.0.10.1
dns-nameservers 8.8.8.8
openstack@network:~$
</pre>

<pre>
openstack@network:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr 00:54:09:25:21:17
          inet addr:10.0.10.21  Bcast:10.0.10.255  Mask:255.255.255.0
          inet6 addr: fe80::254:9ff:fe25:2117/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4053 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3014 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:4715574 (4.7 MB)  TX bytes:255812 (255.8 KB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:160 errors:0 dropped:0 overruns:0 frame:0
          TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)

openstack@network:~$

</pre>
<pre>
openstack@network:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:21:17 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:21:18 brd ff:ff:ff:ff:ff:ff
4: ens5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:21:19 brd ff:ff:ff:ff:ff:ff
5: ens6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:21:16 brd ff:ff:ff:ff:ff:ff
openstack@network:~$
</pre>
<p>
 <b>เครื่อง compute </b>
<pre>
openstack@compute:~$ cat /etc/network/interfaces
...
auto lo
iface lo inet loopback
...
auto ens3
iface ens3 inet static
address 10.0.10.31
netmask 255.255.255.0
network 10.0.10.0
gateway 10.0.10.1
dns-nameservers 8.8.8.8
openstack@compute:~$
</pre>
<pre>
openstack@compute:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr 00:54:09:25:31:17
inet addr:10.0.10.31  Bcast:10.0.10.255  Mask:255.255.255.0
inet6 addr: fe80::254:9ff:fe25:3117/64 Scope:Link
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
RX packets:5322 errors:0 dropped:0 overruns:0 frame:0
TX packets:3096 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:7418377 (7.4 MB)  TX bytes:224114 (224.1 KB)

lo        Link encap:Local Loopback
inet addr:127.0.0.1  Mask:255.0.0.0
inet6 addr: ::1/128 Scope:Host
UP LOOPBACK RUNNING  MTU:65536  Metric:1
RX packets:160 errors:0 dropped:0 overruns:0 frame:0
TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1
RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)
openstack@compute:~$
</pre>
<pre>
openstack@compute:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:31:17 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:31:18 brd ff:ff:ff:ff:ff:ff
4: ens5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:31:19 brd ff:ff:ff:ff:ff:ff
5: ens6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:31:16 brd ff:ff:ff:ff:ff:ff
openstack@compute:~$
</pre>
<p>
 <b>เครื่อง compute1 </b>
<pre>
openstack@compute1:~$ cat /etc/network/interfaces
...
auto lo
iface lo inet loopback
...
auto ens3
iface ens3 inet static
address 10.0.10.32
netmask 255.255.255.0
network 10.0.10.0
gateway 10.0.10.1
dns-nameservers 8.8.8.8
openstack@compute1:~$

</pre> 
<pre>
openstack@compute1:~$ ifconfig
ens3      Link encap:Ethernet  HWaddr 00:54:09:25:32:17
inet addr:10.0.10.32  Bcast:10.0.10.255  Mask:255.255.255.0
inet6 addr: fe80::254:9ff:fe25:3217/64 Scope:Link
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
RX packets:5345 errors:0 dropped:0 overruns:0 frame:0
TX packets:3117 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:7418954 (7.4 MB)  TX bytes:226152 (226.1 KB)

lo        Link encap:Local Loopback
inet addr:127.0.0.1  Mask:255.0.0.0
inet6 addr: ::1/128 Scope:Host
UP LOOPBACK RUNNING  MTU:65536  Metric:1
RX packets:160 errors:0 dropped:0 overruns:0 frame:0
TX packets:160 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1
RX bytes:11840 (11.8 KB)  TX bytes:11840 (11.8 KB)
openstack@compute1:~$
</pre>
<pre>
openstack@compute1:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:32:17 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:32:18 brd ff:ff:ff:ff:ff:ff
4: ens5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:32:19 brd ff:ff:ff:ff:ff:ff
5: ens6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:54:09:25:32:16 brd ff:ff:ff:ff:ff:ff
openstack@compute1:~$
</pre>
<p>
ขอให้ นศ make sure ว่า นศ สามารถใช้ NIC ทุกอันส่งข้อมูลได้ นศ อาจใช้วิธี ping IP address ใน management network โดยเช็คว่าสามารถ ping จาก controller ผ่าน ens3 ไปยัง IP ของ ens3 บนเครื่องอื่นทุกๆเครื่องได้ 
<p>
สำหรับ rns4 ens5 ens6 ให้<b>แอบกำหนดค่า IP</b> (หมายถึงกำหนดแล้วลบทิ้ง คือกำหนดเพื่อเช็คต่อไปนี้เฉยๆ แล้วลบทิ้ง ifdown หรือ ifconfig down ก่อนติดตั้งในส่วนที่ 2 หรือ 3) ให้กับ ensx interface ทุกๆอันที่เหลือและให้เช็คว่า ens4 IP ของทุกเครื่องสามารถ ping กันได้ และ ens5 IP ของทุกเครื่องสามารถ ping กันได้ และ ens6 IP ของทุกเครื่อง ping กันและกันได้ หมายเหตุ ขอให้ระวังว่า ens4 IP ไม่ควร ping ens3 IP หรือ ens5 IP หรือ ens6 IP ได้ พูดอีกอย่างคือ  data tunnel network subnet และ vlan network subnet และ management network subnet ต้องแยก isolate จากกัน 
<p>
เมื่อเช็คเสร็จแล้วให้ ลบ และ ifdown หรือ ifconfig down IP address ของ ens4 ens5 ens6 บนทุกเครื่องออก (เราจะใช้ installation scripts กำหนดค่า หรือกำหนดค่าเองด้วยมือภายหลัง)   
<p>
 <i><a id="vboxhost">1.2 การเตรียมเครื่องสำหรับติดตั้งบน vbox vm </a></i>
<p>
นศ สามารถอ่านคำอธิบายการเตรียม vbox vm สำหรับติดตั้ง openstack ocata ได้ที่เอกสาร <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/openstack-ocata-vbox-vm-preparation.pdf">documents/openstack-ocata-vbox-vm-preparation.pdf</a> ขอให้สร้าง vm และกำหนดค่าต่างๆตามนั้น 
<p>
นศ สามารถดูคลิป youtube ประกอบได้ที่ (ผมไม่ได้เตียม script พูดประกอบ clip ขอให้ทนฟังการพูดตะกุกตะกักหน่อยนะครับ :-) ) <a href="https://www.youtube.com/watch?v=AkDoef8gUJY&index=1&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ">set up virtual gateway on controller</a> และ <a href="https://www.youtube.com/watch?v=N3AfvrlJw2M&index=2&list=PLmUxMbTCUhr4vYsaeEKVkvAGF5K1Tw8oJ">install and setup network, compute, and compute1 vbox vms</a>
<a id="part2"> 
<h4>ส่วนที่ 2: ติดตั้งด้วย scripts</h4>
</a>
<p>
<i><a id="downloadinstaller">2.1 ดาวน์โหลด openstack-ocata-installer scripts</a></i><br>
<p>
นศ จะใช้เครื่อง controller เป็นหลักในการติดตั้งด้วย script เริ่มต้นด้วยการ login เข้า openstack user (makes sure ว่า username และ password คือ "openstack" บนทุกเครื่อง) และ download script ด้วยคำสั่ง 
<pre>
$ cd $HOME
$ git clone https://github.com/kasidit/openstack-ocata-installer
$ cd openstack-ocata-installer
</pre>
เมื่อดู content ของ directory จะมีไฟล์และ subdirectory ดังนี้
<pre>
openstack@controller:~/openstack-ocata-installer$ ls
config.d   exe-config-installer.sh  LICENSE                README.md
documents  install-paramrc.sh       OPSInstaller-init.tar
openstack@controller:~/openstack-ocata-installer$
</pre>
ในกรณีที่ นศ จะติดตั้งบน vbox vm ขอให้ copy ไฟล์ <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/documents/Example.vbox.install-paramrc.sh">Example.vbox.install-paramrc.sh</a> มาเป็น install-paramrc.sh ใน openstack-ocata-installer directory
<pre>
$ cp documents/Example.vbox.install-paramrc.sh install-paramrc.sh
</pre>
<p>
แต่ถ้าติดตั้งบน kvm หรือเครื่องจริงก็ให้ใช้ไฟล์ <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/install-paramrc.sh">install-paramrc.sh</a> ที่มีอยู่แต่เดิมเป็นตัวอย่าง <br> 
<p>
<i><a id="paramrc">2.2 กำหนดค่าพารามีเตอร์สำหรับการติดตั้ง </a></i><br>
<p>
ต่อไป นศ จะกำหนด configuration สำหรับการติอตั้งโดยกำหนดค่าในไฟล์ <a href="https://github.com/kasidit/openstack-ocata-installer/blob/master/install-paramrc.sh">install-paramrc.sh</a> ซึ่งถ้า นศ กำหนดค่า vm และเนตตามที่ระบุใน ส่วนที่ 1.1 และติดตั้งบน kvm (ที่ใช้รหัส ens เป็นชื่อ NIC) นศ ก็สามารถใช้ไฟล์ install-paramrc.sh นี้ได้เลย และในกรณีที่ นศ ใช้ vbox ติดตั้งและกำหนดค่าต่างๆเหมือนที่อธิบายในส่วน 1.2 นศ ก็สามารถใช้ไฟล์ Example.vbox.install-paramrc.sh ได้เลยเช่นกัน (ข้ามไปทำส่วนที่ 2.3 ได้)
<p>
แต่อย่างไรก็ตาม หาก นศ ติดตั้งบนเครื่องจริง ชื่อ NICs และค่าอื่นๆก็อาจเปลี่ยนไป ดังนั้นผมจะอธิบายความหมายของตัวแปรต่างๆในไฟล์ install-paramrc.sh เพื่อที่จะได้กำหนดค่าอย่างถูกต้อง อันดับแรก environment variables สามอันแรกในไฟล์นั้นได้แก่
<pre>
export INSTALL_TYPE=full
export NETWORK_TYPE=dvr_ovs
export PASSWD_TYPE=studypass
</pre>
มีความหมายดังนี้ INSTALL_TYPE เป็นแบบ "full" คือเป็นการติดั้งแบบ 4 nodes ถ้าเปลี่ยนค่าเป็น "compact" จะหมายถึงติดตั้งแบบ 3 nodes ได้แก่ controller network และ compute ส่วน NETWORK_TYPE เป็นตัวแปรที่ระบุชนิดของ network deployment ถ้ากำหนดค่าเป็น "dvr_ovs" หมายถึงใช้ neutron ที่สร้างด้วย openvswitch และปฏิบัติงานแบบ Distributed Virtual Router (DVR) ซึ่งเป็น default configuration ถ้าเปลี่ยนค่าเป็น "classic_ovs" จะหมายถีง neutron ที่สร้างด้วย openvswitch แต่ปฏิบัติงานแบบธรรมดา ไม่มี DVR high availabiility feature สำหรับตัวแปร PASSWD_TYPE เป็นตัวแปรที่ระบุชนิดของ password ที่จะถูกกำหนดสำหรับการติดตั้ง component ต่างๆของ openstack ถ้า่าเป็น "studypass" หมายถึงการกำหนดค่า password ที่เป็น string ธรรมดาที่สื่อความหมายว่าเป็น password ของ component ใด ในทางตรงกันข้าม ถ้ากำหนดค่าเป็น "randompass" จะหมายถึงการกำหนดค่า password สำหรับการติดตั้ง component เหล่านั้นให้เป็นตัวเลข random
<pre>
export OPS_LOGIN_NAME=openstack
export OPS_LOGIN_PASS=openstack
export OPS_TIMEZONE=Asia\\/Bangkok
</pre>
ตัวแปร OPS_LOGIN_NAME และ OPS_LOGIN_PASS ในที่นี้เรากำหนดให้เป็น "openstack" ทั้งคู่ ค่า OPS_LOGIN_NAME และ OPS_LOGIN_PASS นี้ต้องตรงกับชื่อ login name และค่า password ของ Linux account ที่ นศ จะใช้ติดตั้ง OpenStack บนทุก node ส่วน OPS_TIMEZONE นั้นใช้กำหนดค่ว TIMEZONE ซึ่งในทีนี้คือ Asia/Bangkok
<p>

...

<p>
ต่อ.... soon
<a id="part2"> 
<h4>ส่วนที่ 3: ติดตั้งด้วยมือ</h4>
</a>


<b>อ้างอิง</b>
1. http://sciencecloud-community.cs.tu.ac.th/ 
2. http://vasabilab.cs.tu.ac.th/ 
3. http://docs.openstack.org/
