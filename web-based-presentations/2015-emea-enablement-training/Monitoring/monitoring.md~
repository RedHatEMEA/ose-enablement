
### Monitoring OSE Infrastructure

---
###Monitoring Best Practices

![Broker](/content/images/Broker.png) <!-- .element: class="noshadow" -->

--
###Monitoring Best Practices
####BROKER

* __`oo-admin-chk`__ 
* __`oo-accept-broker`__
* App creation/status check
* Rpm consistency check with other brokers
* mco ping compared with nodes in districts (__`oo-mco ping`__)
* Capacity (__`oo-stats`__)

---
###Monitoring Best Practices
####NODES

* __`oo-accept-node -v`__ (Checks that node setup is valid and functional and its gears are in good condition)
   * Rpm consistency check with other ex-nodes
   * Selinux status (enforcing)
   * Iptables status (make sure rules are loaded)


__Try to keep ex-nodes as similar as possible and monitor to ensure this__

---
###Monitoring Best Practices
####MongoDB

* Replica set health
* DB size
* Addition / removal of indexes
* Many other DB related metrics (eg: Mikoomi)
* Rpm consistency check with other Mongo nodes
* Mongo process / port

* Mikoomi project: https://code.google.com/p/mikoomi/wiki/03
  - __`MongoDB Plugin`__ for Zabbix to monitor standalone, replicated as well as clustered MongoDB instances

--

##Some monitoring tools

* __Openshift Admin console__
* __Zabbix__ ---> Reference Architecture available and used by Openshift online
* CA Wily Introscope EPAgent (eg: Produban)
* ....................

---

##Openshift Admin console

* An optional Administration Console is available for OpenShift Enterprise
* Allows administrators to search and navigate entities to plan the capacity of an OpenShift Enterprise deployment.

__The Administration Console is read-only__, so the settings or data cannot be modified.

---

##Accessing the Openshift Admin console

* The Administration Console runs as a plug-in to the OpenShift Enterprise broker application
        Configuration file /etc/openshift/plugins.d/openshift-origin-admin-console.conf
* By default does not provide access control __You must log in to an OpenShift Enterprise host__
* You can enable external access by modifying the broker host httpd proxy configuration
* You can also configure authentication based on: user credentials, kerberos, htpasswd, host name, client IP or LDAP.

--

##Openshift Admin console

![Admin console](/content/images/AdminConsole1.png) <!-- .element: class="noshadow" -->
--

##Openshift Admin console

![Admin console](/content/images/AdminConsole2.png) <!-- .element: class="noshadow" -->
--

##Openshift Admin console

![Admin console](/content/images/AdminConsole3.png) <!-- .element: class="noshadow" -->
--

##Openshift Admin console

![Admin console](/content/images/AdminConsole4.png) <!-- .element: class="noshadow" -->

---
##Zabbix

* OpenShift Online's Zabbix scripts and monitoring bits 
https://github.com/openshift/openshift_zabbix
* Zabbix Monitoring Cartridges for OpenShift
https://blog.openshift.com/introducing-zabbix-monitoring-cartridges-for-openshift/

---

###CA Wily Introscope EPAgent @ Produban

![Produban](/content/images/produban.png) <!-- .element: class="noshadow" fullscreen-size="contain"-->

https://github.com/Produban/OpenShift20_Monitoring

