Pacemaker OCF cluster resource that pings MySQL nodes
Same as ping or pingd OCF resources

Useful in case of Master-Master MySQL replication setup for HA for grouping MySQL with VIP on an alive MySQL instance.

           ______________
          |              |
          | MySQL Master |__
          |______________|  | 
                |   |       |
                |   | Repl  | VIP (moving when MySQL/node failures)
           _____V___|____   |
          |              |  |
          | MySQL Master |<–
          |______________|

Resource uses "mysqladmin ping" command.

Example
=======

We  have 2 node cluster with master-master replication. 

Install resource script
----------------------------------

    cd /usr/lib/ocf/resource.d
    mkdir mysql; cd mysql
    wget https://raw.github.com/Sov1et/ocf-mysqlping/master/mysqlping -O  mysqlping # get resource
    chmod +x ./mysqlping

Configure mysqlping resource
----------------------------

    crm configure
    primitive P_MYSQLPING ocf:mysql:mysqlping params dampen=7s host_list="localhost" user="test" password="password multiplier=222 timeout="20" op monitor interval=5s timeout="30s"
    clone CL_MYSQLPING P_MYSQLPING meta globally-unique="false"
    location L_MYSQLPING_01 CL_MYSQLPING 100: first_master_node.local
    location L_MYSQLPING_02 CL_MYSQLPING 100: second_master_node.local
    property default-resource-stickiness=100
    commit

Configure default location for the MySQL VIP
--------------------------------------------

    crm configure
    primitive P_MYSQL_IP ocf:heartbeat:IPaddr2 params ip="192.168.56.163" nic="eth0" op monitor interval="5s"
    location L_MYSQL_IP_01 P_MYSQL_IP 100: first_master_node.local
    location L_MYSQL_IP_02 P_MYSQL_IP 10: second_master_node.local
    commit

Configure VIP collocation with mysqlping score
----------------------------------------------

    crm configure
    location L_MYSQLPING_IP P_MYSQL_IP rule mysqlping: defined mysqlping
    commit
