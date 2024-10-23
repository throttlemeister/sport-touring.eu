---
title: Monitoring your network or homelab using Zabbix and Grafana
author: throttlemeister
type: post
date: 2021-07-26T21:30:00+00:00
url: /monitoring-your-network-or-homelab-using-zabbix-and-grafana/
categories:
  - Computer
  - homelab
  - Linux
  - Monitoring

---
For the longest time, I have been monitoring my network and homelab using Observium. This worked and does work very well. Observium is very good at what it does. However, there are a couple of things that do not work so well for me using Observium:

- Observium does not let me add applications to monitor very easily or at all
- Observium is limited to what can be offered through SNMP
- Observium is not open source and as such it cannot be modified or changed to your needs
- Observium is not an application I have come across in my professional life, so knowing how it works does not help me professionally.

That last bit is obviously not a necessity, however I do feel it's always a nice thing to be able to apply things you have learned in your homelab into your professional environment.

After lots of investigations and trial & error, I have settled on using Zabbix for my monitoring needs. Zabbix is open source product that is used a lot in corporate environments and it is very flexible and extensible. Obviously you could add or change code as it is open source, but you don't really need to. As Zabbix is template driven, its functionality can be extended by adding templates and there is a plethora of templates available for Zabbix, both on the Zabbix site itself ([Zabbix Share](https://share.zabbix.com/)) as well as places like GitHub. Also, Zabbix can use an agent installed on the system to collect the data you want to monitor, or you can use SNMP if you can't or don't want to install an agent on a device (for instance a network router).

The one thing I do not like about Zabbix is that the historic view is not easy to get to, nor as pretty displayed in a dashboard-like view as it is in Observium. However, that is not a blocker as we can use Grafana, another open source tool that is used quite a bit in Corporate Land to create dashboards and display relevant historic data.

# Installing Zabbix

Installing Zabbix is no more complicated than installing most other software on Linux. I installed Zabbix on my Raspberry Pi 2B and the short version is this:

a. Install the Zabbix repository:

    # wget <https://repo.zabbix.com/zabbix/5.4/raspbian/pool/main/z/zabbix-release/zabbix-release_5.4-1+debian10_all.deb>
    # dpkg -i zabbix-release_5.4-1+debian10_all.deb
    # apt update

b. Install the Zabbix server, frontend and agent on the server machine  

    # apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

c. Create the database for Zabbix

    # mysql -uroot -ppassword
    mysql> create database zabbix character set utf8 collate utf8_bin;
    mysql> create user zabbix@localhost identified by 'password';
    mysql> grant all privileges on zabbix.* to zabbix@localhost;
    mysql> quit;

And import the database schema  

    # zcat /usr/share/doc/zabbix-sql-scripts/mysql/create.sql.gz | mysql -uzabbix -p zabbix

If your database is not residing on your Zabbix server, you can add -h <server> to the above to make sure you are connecting to the remote database server. Also, when creating the user and you are using a remote database, make sure the Zabbix user is allowed to connect over the network to the database.

d. Configure the Zabbix server to use the database you have created by filling out the relevant fields in /etc/zabbix/zabbix_server.conf

e. Start your brand-new Zabbix server:

    # systemctl restart zabbix-server zabbix-agent apache2
    # systemctl enable zabbix-server zabbix-agent apache2

f. You are done. You can now login to your server on http://<server>:3000 and log in with user Admin and password zabbix and configure everything else using the web frontend.

# Installing Grafana

Installing Grafana is equally simple.  
a. Install the pre-requisites and add the repository key

    sudo apt-get install -y apt-transport-https
    sudo apt-get install -y software-properties-common wget
    wget -q -O - <https://packages.grafana.com/gpg.key> | sudo apt-key add -`

b. Add the repository  

    echo "deb <https://packages.grafana.com/oss/deb> stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

c. Install Grafana  

    apt update<br>apt install grafana

d. Then it is just a matter of starting the Grafana server and making sure it starts at boot.  

    sudo systemctl daemon-reload
    sudo systemctl start grafana-server
    sudo systemctl status grafana-server
    sudo systemctl enable grafan-server

That's it! You can now log in using the default username/password of admin/admin.

Note: if you want to apply specific configurations, like for instance database (mysql, postgres, sqlite3) beyond the default, you should refer the Grafana manuals as that would be a bit beyond the scope of this page.

# Tying things together

First you will need to install the Zabbix app into Grafana. We can do this using the commandline interface for Grafana:

    grafana-cli plugins install alexanderzobnin-zabbix-app

After you do this, you can configure the Zabbix datasource. Go to Configuration -> Data sources and click add data source. Scroll down to the bottom, and you will see one called Zabbix'.

Fill in the details of your Zabbix installation, and make sure you add the api_jsonrpc.php to the end of your URL. Check With credentials' under auth and under Zabbix API details add your username and password. Click Save & test and if all is ok, it will give you a green checkmark while saying data source updated.

You are now ready to add a dashboard to your Grafana and start monitoring your Zabbix data. I use [this dashboard](https://grafana.com/grafana/dashboards/5363) from Paulo Paim. You can add it by going to Dashboards -> Manage and then clicking the import button. In the box saying import from grafana.com, type the ID of the dashboard. In this case 5363 and click load.

That's it!

Links:  

- [Zabbix manual](https://www.zabbix.com/documentation/current/manual)
- [Grafana documentation](https://grafana.com/docs/grafana/latest/)
- [Zabbix templates and add-ons](https://share.zabbix.com/)
- Grafana [dashboards](https://grafana.com/grafana/dashboards) and [plugins](https://grafana.com/grafana/plugins/)
