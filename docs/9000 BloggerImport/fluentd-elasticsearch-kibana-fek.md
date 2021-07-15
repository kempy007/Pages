---
title: 'FluentD ElasticSearch Kibana (FEK platform)'
date: 2016-03-31T23:19:00.001+01:00
draft: false
aliases: [ "/2016/03/fluentd-elasticsearch-kibana-fek.html" ]
parent: Blogger
---
#### date: 2016-03-31

Date 31-03-2016

Ubuntu 14.04  
Kibana 4.5.0  
ElasticSearch 2.3.0  
FluentD aka td-agent 2.3.1

Do basic install of ubuntu with openssh.

\# nano /etc/network/interfaces

> \# This file describes the network interfaces available on your system  
> \# and how to activate them. For more information, see interfaces(5).
> 
> \# The loopback network interface  
> auto lo  
> iface lo inet loopback
> 
> \# The primary network interface  
> auto eth0  
> iface eth0 inet static  
> address 192.168.0.204   # change to your ip etc  
> netmask 255.255.255.0  
> gateway 192.168.0.1  
> dns-nameservers 208.67.222.222  

\# apt-get update  
\# apt-get upgrade

}>>pre reqs

\# nano /etc/security/limits.conf    #  add to end  

>     root soft nofile 65536  
>     root hard nofile 65536  
>     \* soft nofile 65536  
>     \* hard nofile 65536
> 
>   

\# edit /etc/sysctl.conf     # add to end

>     net.ipv4.tcp\_tw\_recycle = 1  
>     net.ipv4.tcp\_tw\_reuse = 1  
>     net.ipv4.ip\_local\_port\_range = 10240    65535  
>     net.ipv6.conf.all.disable\_ipv6 = 1  
>     net.ipv6.conf.default.disable\_ipv6 = 1  
>     net.ipv6.conf.lo.disable\_ipv6 = 1

reboot  
Check network works fine

}>>install fluentd aka td-agent

\# curl -L [https://toolbelt.treasuredata.com/sh/install-ubuntu-trusty-td-agent2.sh](https://toolbelt.treasuredata.com/sh/install-ubuntu-trusty-td-agent2.sh) | sh

}>>install elasticsearch

\# sudo apt-get install openjdk-7-jre-headless --yes

\# wget -qO - [https://packages.elastic.co/GPG-KEY-elasticsearch](https://packages.elastic.co/GPG-KEY-elasticsearch) | sudo apt-key add -  
echo "deb [http://packages.elastic.co/elasticsearch/2.x/debian](http://packages.elastic.co/elasticsearch/2.x/debian) stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list

\# sudo apt-get update && sudo apt-get install elasticsearch

\# sudo update-rc.d elasticsearch defaults 95 10

\# nano /etc/elasticsearch/elasticsearch.yml   # add to end of file

> network.host: 127.0.0.1 # change to your host ip if over network  
> http.port: 9200
> 
>  

  
}>>install kibana

\# cd /opt  
\# wget [https://download.elastic.co/kibana/kibana/kibana-4.5.0-linux-x64.tar.gz](https://download.elastic.co/kibana/kibana/kibana-4.5.0-linux-x64.tar.gz)  
\# tar -zxvf kibana-4.5.0-linux-x64.tar.gz

\# nano kibana-4.5.0-linux-x64/config/kibana.yml   # add to end of file  

> elasticsearch.url: "[http://localhost:9200"](http://localhost:9200")

}>>install td-agent plugins

\# sudo apt-get install make libcurl4-gnutls-dev --yes  
\# sudo /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-elasticsearch  
\# sudo /opt/td-agent/embedded/bin/fluent-gem install fluent-plugin-record-reformer

\# nano /etc/td-agent/td-agent.conf    # add to top of file and comment out any other lines  

> <source>  
>    @type tcp  
>    format json  
>    port 5514  
>    tag windowslog  
> </source>  
> <match windowslog>  
> #   @type stdout # use for testing  
>         @type elasticsearch  
>         host localhost  
>         port 9200  
>         index\_name fluentd  
>         type\_name fluentd  
> </match>

I haven’t explored the pro’s and cons to this solution but one pro is no 500mb limits that Splunk would trip you over…

I suppose a con at this stage is the pivot features that splunk does have. I think it can be done with kibana but takes more effort.

References  
[http://docs.fluentd.org/articles/install-by-deb](http://docs.fluentd.org/articles/install-by-deb)  
[https://www.elastic.co/downloads/kibana](https://www.elastic.co/downloads/kibana)  
[https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)  
[https://www.digitalocean.com/community/tutorials/elasticsearch-fluentd-and-kibana-open-source-log-search-and-visualization](https://www.digitalocean.com/community/tutorials/elasticsearch-fluentd-and-kibana-open-source-log-search-and-visualization)  
[https://github.com/uken/fluent-plugin-elasticsearch](https://github.com/uken/fluent-plugin-elasticsearch)