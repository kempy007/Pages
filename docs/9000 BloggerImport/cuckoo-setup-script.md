---
title: 'Cuckoo Setup Script'
date: 2016-06-26T16:40:00.001+01:00
draft: false
aliases: [ "/2016/06/cuckoo-setup-script.html" ]
parent: Blogger
---
#### date: 2016-06-26


#!/bin/bash  
\# Author Martyn Kemp www.blogsploit.co.uk  
\# This file is for Cuckoo Sandbox - [http://www.cuckoosandbox.org](http://www.cuckoosandbox.org)  

printf "\*\*install order  
1 update  
2 apt-get  
4 mysql  
5 git cuckoo  
12 fix ssdeep  
6-10 confs  
3 pip"

\# system will have dhcp, do ifconfig to discover IP and then ssh this file on and chmod then execute  
\# Ubuntu 16.04 tested

\# This program is free software: you can redistribute it and/or modify  
\# it under the terms of the GNU General Public License as published by  
\# the Free Software Foundation, either version 3 of the License, or  
\# any later version.

\# This program is distributed in the hope that it will be useful,  
\# but WITHOUT ANY WARRANTY; without even the implied warranty of  
\# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the  
\# GNU General Public License for more details.

\# You should have received a copy of the GNU General Public License  
\# along with this program.  If not, see <[http://www.gnu.org/licenses/](http://www.gnu.org/licenses/)\>.  
\# pinching bits from [https://github.com/buguroo/cuckooautoinstall/blob/master/cuckooautoinstall.bash](https://github.com/buguroo/cuckooautoinstall/blob/master/cuckooautoinstall.bash)

\# Configuration variables. You can override these in config.  
SUDO="sudo"  
#TMPDIR=$(mktemp -d)  
#RELEASE=$(lsb\_release -cs)  
CUCKOO\_USER="cuckoo"  
ORIG\_DIR=$( cd "$( dirname "${BASH\_SOURCE\[0\]}"  )" && pwd  )  
CUCKOO\_REPO='[https://github.com/cuckoobox/cuckoo'](https://github.com/cuckoobox/cuckoo')

#LOG=$(mktemp)

#machinetype 1=esx 2=baremetal  
machinetype="2"  
usemenu="yes"

#ESXi Machine secrets  
esxhost="192.168.0.210"  
esxuser="root"  
esxpw="bythepowerofgrayskull1337"  
esxInt="eth2" #was eth1  
esxvm1ip="192.168.100.1"  
esxvm1snapshot="CleanRunning"

#BareMetal Machine secrets  
bmInt="eth0"  
bmSanHost="192.168.0.150"  
bmSanUser="root"  
bmSanpw="null" #deprecated by using keypairs for ssh auth  
bmh1IP="192.168.0.130"  
bmh1ipmiip="192.168.0.199"  
bmh1ipmiusername="admin"  
bmh1ipmipassword="bythepowerofgrayskull1337"  
bmh1sanscript="/root/vmh02.sh"

#mySql secrets  
mysqlpw="qaz123"

  
#declare -a packages  
#declare -a python\_packages

packages="python-sqlalchemy python-bson python-dpkt python-jinja2 python-magic python-pymongo python-gridfs python-libvirt python-bottle python-pefile python-chardet python-django python-libxml2 python-pyrex python-ssdeep build-essential python-dev python-pip git automake libtool yara ssdeep libvirt-bin volatility tcpdump libffi-dev libssl-dev libxml2-dev libxslt1-dev mongodb autoconf dh-autoreconf libcurl4-gnutls-dev libmagic-dev libcap2-bin libpciaccess-dev libnl-3-dev pkg-config libxml2-dev libgnutls-dev libdevmapper-dev libcurl4-gnutls-dev w3-recs  ipmitool"  
#error with libnl-dev (trying libnl-3-dev) and w3c-dtd-xhtml (trying w3-recs)

pip\_packages="pymongo django pydeep maec py3compat lxml cybox distorm3 pycrypto alembic beautifulsoup4 cffi chardet cryptography dpkt ecdsa elasticsearch enum34 Flask HTTPReplay idna ipaddress itsdangerous Jinja2 jsbeautifier Mako MarkupSafe ndg-httpsclient oletools pefile pyasn1 pycparser pymisp pyOpenSSL python-dateutil python-editor python-magic requests six SQLAlchemy tlslite-ng wakeonlan Werkzeug pymysql"

\# Pretty icons  
log\_icon="\\e\[31m✓\\e\[0m"  
log\_icon\_ok="\\e\[32m✓\\e\[0m"  
log\_icon\_nok="\\e\[31m✗\\e\[0m"

check\_viability(){  
    \[\[ $UID != 0 \]\] && {  
        type -f $SUDO || {  
            echo "You're not root and you don't have $SUDO, please become root or install $SUDO before executing $0"  
            exit  
        }  
    } || {  
        SUDO=""  
    }  
}

#1  
upgradeSys(){  
echo "\*\*\*>>>updating system"  
read -p "press \[enter\] to continue"  
apt-get update  
apt-get upgrade -y  
return 0  
}  
#2  
installPackages(){  
echo "\*\*\*>>>installing packages"  
read -p "press \[enter\] to continue"  
apt-get install -y ${packages}  
return 0  
}

yaraManualCompile(){  
#included by default in Ubuntu 16.04  
echo "\*\*\*>>>Fetching Yara and compiling, then adding python libs"  
#read -r -p "press y to continue, any other key to skip" resp1  
#if \[\[$resp1 == ^(yes|y| ) \]\]; then  
#    wget [https://github.com/plusvic/yara/archive/v3.1.0.tar.gz](https://github.com/plusvic/yara/archive/v3.1.0.tar.gz)

#    tar -zxf v3.1.0.tar.gz  
#    cd yara-3.1.0/  
#    bash build.sh  
#    make install

#    cd yara-python/  
#    python setup.py install  
#fi  
}  
ssdeepManualCompile(){  
#included by default in Ubuntu 16.04, but pip install pydeep fails     
echo "\*\*\*>>>Fetching ssdeep and compiling, then adding python libs"  
read -r -p "press \[enter\] to continue"  
apt-get remove ssdeep  
wget [http://sourceforge.net/projects/ssdeep/files/ssdeep-2.13/ssdeep-2.13.tar.gz/download](http://sourceforge.net/projects/ssdeep/files/ssdeep-2.13/ssdeep-2.13.tar.gz/download)  
tar -zxf download  
cd ssdeep-2.13  
./configure  
make  
make install  
pip install pydeep  
}  
libvirtManualCompile(){  
#looks like libvirt 1.3.1 is included by default in Ubuntu 16.04  
echo "\*\*\*>>>#commented out>Fetching libvirt and compiling \*ignore html.tmp errors"  
#read -p "press \[enter\] to continue"  
#wget [http://libvirt.org/sources/libvirt-1.3.1.tar.gz](http://libvirt.org/sources/libvirt-1.3.1.tar.gz)  
#tar zxf libvirt-1.3.1.tar.gz libvirt-1.3.1  
#cd libvirt-1.3.1/  
#./configure --with-esx=yes  
#make  
#make install  
}  
#3  
installPIPpackages(){  
echo "\*\*\*>>>installing pip packages"  
read -p "press \[enter\] to continue"  
pip install ${pip\_packages}  
#pip install --upgrade pip  
return 0  
}  
#4  
setupMysql(){  
echo "\*\*\*>>>installing mysql packages"  
read -p "press \[enter\] to continue"  
apt-get install -y mysql-server  
apt-get install -y python-mysqldb  
echo "\*\*\*>>>Say no to changing root pw and YES to all other options"  
mysqld --initialize  
#mysql\_secure\_installation  
echo "\*\*\*>>>creating mySQL db for cuckoo"  
read -p "press \[enter\] to continue"  
#todo fix below to use variable  
mysql -u root -p -e "CREATE DATABASE cuckoo; GRANT ALL ON cuckoo.\* TO 'cuckoo'@'localhost' IDENTIFIED BY '"$mysqlpw"';"  
return 0  
}  
#5  
fetchCuckoo(){  
echo "\*\*\*>>>Fetching cuckoo from git"  
read -p "press \[enter\] to continue"  
cd /opt  
git clone $CUCKOO\_REPO  
cd cuckoo  
git checkout 743fe2c5410ff12a39dc158e343b7f8a044ff120  
\# only seems to fetch the dev build not rc1

echo "\*\*\*>>>Updating cuckoo community modules"  
read -p "press \[enter\] to continue"  
/opt/cuckoo/utils/community.py -af  
   
pip install -r /opt/cuckoo/requirements.txt  
#cd /opt  
#apt-get install unzip  
#wget [https://github.com/cuckoosandbox/cuckoo/archive/2.0-rc1.zip](https://github.com/cuckoosandbox/cuckoo/archive/2.0-rc1.zip)  
return 0  
}  
#6  
updateMongodbConf(){  
echo "\*\*\*>>>Enabling mongodb in reporting.conf"  
read -p "press \[enter\] to continue"  
sed -i '/\\\[mongodb\\\]/{ N; s/.\*/\\\[mongodb\\\]\\nenabled = yes/; }' /opt/cuckoo/conf/reporting.conf  
return 0  
}

#>>>>>>>touching config files<<<<<<<<<<<<<<<<<<  
#7  
updateGrubConf(){  
echo "\*\*\*>>>Editing /etc/default/grub to use old style interface names ethX"  
read -p "press \[enter\] to continue"  
sed -i '/^GRUB\_CMDLINE\_LINUX=/s/=.\*/="net.ifnames=0 biosdevname=0"/' /etc/default/grub  
#cat /etc/default/grub

echo "\*\*\*>>>Running grub-mkconfig"  
read -p "press \[enter\] to continue"  
grub-mkconfig -o /boot/grub/grub.cfg  
echo "\*\*\*>>>system needs restart for changes to take effect"  
return 0  
}  
#8  
updateRCconf(){  
echo "\*\*\*>>>Writing new /etc/rc.local"  
read -p "press \[enter\] to continue"  
cat >/etc/rc.local <<EOL  
#!/bin/sh -e  
#  
\# rc.local  
#  
\# This script is executed at the end of each multiuser runlevel.  
\# Make sure that the script will "exit 0" on success or any other  
\# value on error.  
#  
\# In order to enable or disable this script just change the execution  
\# bits.  
#  
\# By default this script does nothing.  
ifconfig eth1 up  
ifconfig eth1 promisc  
#enable routing  
iptables -A FORWARD -o eth0 -i eth2 -s 192.168.100.0/24 -m conntrack --ctstate NEW  
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  
iptables -A POSTROUTING -t nat -j MASQUERADE  
sysctl -w net.ipv4.ip\_forward=1  
exit 0  
EOL  
return 0  
}  
#9  
updateInterfaceConf(){  
echo "\*\*\*>>>Writing new /etc/network/interfaces"  
read -p "press \[enter\] to continue"  
cat >/etc/network/interfaces <<EOL  
\# This file describes the network interfaces available on your system  
\# and how to activate them. For more information, see interfaces(5).

\# The loopback network interface  
auto lo  
iface lo inet loopback

\# The primary network interface  
auto eth0  
iface eth0 inet static  
   address 192.168.0.202  
   netmask 255.255.255.0  
   gateway 192.168.0.1  
   dns-nameservers 208.67.222.222 208.67.220.220

\# The Monitor network interface  
auto eth1  
iface eth1 inet manual  
   up ip address add 0/0 dev $IFACE  
   up ip link set $IFACE up  
   up ip link set $IFACE promisc on  
down ip link set $IFACE promisc off  
down ip link set $IFACE down

\# The Analysis network interface  
auto eth2  
iface eth2 inet static  
address 192.168.100.254  
netmask 255.255.255.0  
EOL  
return 0  
}  
#10 todo clean out password stuff  
updateCuckooConfs(){  
#cat /opt/cuckoo/conf/esx.conf | egrep -v "(#.\*|^$)"  
echo "\*\*\*>>>Writing new /opt/cuckoo/conf/esx.conf"  
read -p "press \[enter\] to continue"  
cp /opt/cuckoo/conf/esx.conf /opt/cuckoo/conf/esx.conf.orig  
cat >/opt/cuckoo/conf/esx.conf <<EOL  
\[esx\]  
dsn = esx://$esxhost/?no\_verify=1  
username = $esxuser  
password = $esxpw  
machines = CuckooVM1  
interface = $esxInt  
\[CuckooVM1\]  
label = CuckooVM1  
platform = windows  
snapshot = $esxvm1snapshot  
ip = $esxvm1ip  
EOL

echo "\*\*\*>>>Writing new /opt/cuckoo/conf/baremetal.conf"  
read -p "press \[enter\] to continue"  
cat >/opt/cuckoo/conf/baremetal.conf <<EOL  
\# Author Martyn Kemp www.blogsploit.co.uk  
\# This file is for Cuckoo Sandbox - [http://www.cuckoosandbox.org](http://www.cuckoosandbox.org)  
\[baremetal\]  
\# Specify a comma-separated list of available machines to be used. For each  
\# specified ID you have to define a dedicated section containing the details  
\# on the respective machine. (E.g. baremetal1,baremetal2,baremetal3)  
machines = baremetal1

\# Default network interface. eth0 physical switch, eth1 vmware vswitch only  
interface = $bmInt

\[san\]  
\# Credentials to access the Solaris SAN server.  
\# we ssh onto this server and invoke a preconfigured script  
\# the script should do  
#     stmfadm delete-lu -k {id obtain via napp-it webgui eg 600144f0107a0900000055fa987b0001}  
#     zfs rollback -f pool01/boot01@{timestamp}\_{comment}   
#     sbdadm import-lu /dev/zvol/rdsk/pool01/boot01  
#  
\# setup ssh keypair so no password required, still need to know username  
#  
host = $bmSanHost  
username = $bmSanUser  
password = $bmSanpw

\[baremetal1\]  
\# Specify the label name of the current machine as specified in your  
\# baremetal machine configuration.  
label = baremetal1

\# Specify the operating system platform used by current machine  
\# \[windows/darwin/linux\].  
platform = windows

\# Specify the IP address of the current machine. Make sure that the IP address  
\# is valid and that the host machine is able to reach it. If not, the analysis  
\# will fail.  
ip = $bmh1IP

\# Specify the IPMI IP address of the baremetal host hardware.  
ipmiip = $bmh1ipmiip  
ipmiusername = $bmh1ipmiusername  
ipmipassword = $bmh1ipmipassword  
sanscript = $bmh1sanscript  
EOL

echo "\*\*\*>>>Writing new /opt/cuckoo/modules/machinery/baremetal.py"  
read -p "press \[enter\] to continue"  
cat >/opt/cuckoo/modules/machinery/baremetal.py <<EOL  
\# Author Martyn Kemp www.blogsploit.co.uk  
\# This file is for Cuckoo Sandbox - [http://www.cuckoosandbox.org](http://www.cuckoosandbox.org)

import logging  
import subprocess

log = logging.getLogger(\_\_name\_\_)

from lib.cuckoo.common.abstracts import Machinery  
from lib.cuckoo.common.exceptions import CuckooCriticalError  
from lib.cuckoo.common.exceptions import CuckooMachineError

class BareMetal(Machinery):  
    """Manage BareMetal sandboxes."""

    # BareMetal machine states.  
    RUNNING = "running"  
    STOPPED = "stopped"  
    ERROR = "error"  
     
    #IPMI states  
    STATUSOFF = "Chassis Power is off"  
    TURNEDON = "Chassis Power Control: Up/On"  
    STATUSON = "Chassis Power is on"  
    TURNEDOFF = "Chassis Power Control: Down/Off"

  
    def \_initialize\_check(self):  
        """Ensures that credentials have been entered into the config file.  
        @raise CuckooCriticalError: if no credentials were provided or if  
            one or more BareMetal machines are offline.  
        """  
        # TODO This should be moved to a per-machine thing.  
        if not self.options.san.host or not self.options.san.username or not self.options.san.password:  
            raise CuckooCriticalError(  
                "BareMetal machine San details are missing, please add it to "  
                "the BareMetal machinery configuration file."  
            )

  
    def start(self, label, task):  
        """Start a baremetal machine.  
        @param label: baremetal machine name.  
        @param task: task object.  
        @raise CuckooMachineError: if unable to start.  
        """  
        ipmiip = self.options.get(label)\["ipmiip"\]  
        ipmiusername = self.options.get(label)\["ipmiusername"\]  
        ipmipassword = self.options.get(label)\["ipmipassword"\]  
        args = \["ipmitool", "-H", ipmiip, "-U", ipmiusername, "-P", ipmipassword, "-I", "lanplus", "chassis", "power", "on" \]  
        output = subprocess.check\_output(args)  
        if "Chassis Power Control: Up/On" not in output:  
            raise CuckooMachineError("Unable to initiate IPMI start request")  
        else:  
            log.debug("IPMI start success: %s." % label)  
             
    def stop(self, label):  
        """Stops a baremetal machine.  
        @param label: baremetal machine name.  
        @raise CuckooMachineError: if unable to stop.  
        """  
        ipmiip = self.options.get(label)\["ipmiip"\]  
        ipmiusername = self.options.get(label)\["ipmiusername"\]  
        ipmipassword = self.options.get(label)\["ipmipassword"\]  
        args = \["ipmitool", "-H", ipmiip, "-U", ipmiusername, "-P", ipmipassword, "-I", "lanplus", "chassis", "power", "off" \]  
             
        output = subprocess.check\_output(args)  
         
        if "Chassis Power Control: Down/Off" not in output:  
            raise CuckooMachineError("Unable to initiate IPMI stop request")  
        else:  
            log.debug("IPMI stop success: %s." % label)  
            self.snapshotrestore(label)

    def snapshotrestore(self, label):  
        sanscript = self.options.get(label)\["sanscript"\]

        args = \["ssh", self.options.san.username + "@" + self.options.san.host, sanscript \]

        output = subprocess.check\_output(args)

        if "step 3  0" not in output:  
            raise CuckooMachineError("Unable to initiate SAN Snapshot request")  
        else:  
            log.debug("SAN snapshot restore success: %s." % label)

EOL  
echo "\*\*\*>>>Editing /opt/cuckoo/conf/cuckoo.conf"  
read -p "press \[enter\] to continue"  
cp /opt/cuckoo/conf/cuckoo.conf /opt/cuckoo/conf/cuckoo.conf.orig  
sed -i '/^connection =/s/=.\*/= mysql:\\/\\/cuckoo:'$mysqlpw'@localhost\\/cuckoo/' /opt/cuckoo/conf/cuckoo.conf

case "$machinetype" in  
  1 ) sed -i '/^machinery =/s/=.\*/= esx/' /opt/cuckoo/conf/cuckoo.conf; sed -i '/^ip =/s/=.\*/= 192.168.100.254/' /opt/cuckoo/conf/cuckoo.conf;;  
  2 ) sed -i '/^machinery =/s/=.\*/= baremetal/' /opt/cuckoo/conf/cuckoo.conf; sed -i '/^ip =/s/=.\*/= 192.168.0.202/' /opt/cuckoo/conf/cuckoo.conf;;  
esac  
return 0  
}  
#11 todo change hardcoded for vars at top of file  
setupKeypair(){  
#setup keypair so cuckoo no longer needs prompting  
ssh-keygen  
ssh-copy-id $bmSanUser@$bmSanHost  
return 0  
}

startmenu(){  
PS3='Please enter your choice: '  
options=("Upgrade System" "Install Pkgs" "PIP Pkgs" "mySQL" "Fetch Cuckoo" "Conf mongodb" "Conf grub" "Conf RC" "Conf Interfaces" "Conf Cuckoo" "SSH Keypair" "Fix ssdeep/pydeep" "Quit")  
select opt in "${options\[@\]}"  
do  
    case $opt in  
        "Upgrade System")  
            upgradeSys  
            ;;  
        "Install Pkgs")  
            installPackages  
            ;;  
        "PIP Pkgs")  
            installPIPpackages  
            ;;  
        "mySQL")  
            setupMysql  
            ;;  
        "Fetch Cuckoo")  
            fetchCuckoo  
            ;;  
        "Conf mongodb")  
            updateMongodbConf  
            ;;  
        "Conf grub")  
            updateGrubConf  
            ;;  
        "Conf RC")  
            updateRCconf  
            ;;  
        "Conf Interfaces")  
            updateInterfaceConf  
            ;;  
        "Conf Cuckoo")  
            updateCuckooConfs  
            ;;  
        "SSH Keypair")  
            setupKeypair  
            ;;  
        "Fix ssdeep/pydeep")  
            ssdeepManualCompile  
            ;;  
        "Quit")  
            break  
            ;;  
        \*) echo invalid option;;  
    esac  
done  
}

#initiate  
#echo -e "${log\_icon\_ok} ${2} init"  
#printf "${log\_icon\_ok} ${2} init2"

check\_viability

case ${usemenu} in  
    "yes") startmenu;;  
    \*) upgradeSys; installPackages; installPIPpackages; setupMysql; fetchCuckoo; updateMongodbConf; updateGrubConf; updateRCconf; updateInterfaceConf; updateCuckooConfs; setupKeypair;  
    ;;  
esac

  
echo "\*\*\*>>>you should cd /opt/cuckoo/web then ./manage.py runserver 0.0.0.0:8080"  
echo "\*\*\*>>>to run cuckoo /opt/cuckoo/cuckoo.py"