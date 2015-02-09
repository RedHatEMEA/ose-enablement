#**Lab 4: Installing OpenShift Enterprise  with *oo-install***


During this lab you will stand up your OpenShift Enterprise (OSE) environment.
We will **not** be deploying a highly available configuration of the product.
These VM's are deployed in the Amsterdam Demonstration Lab. We were not able
to create 13 VM's like in the BU training, but each student gets 3.

We will deploy the following:

- (1) Brokers (with broker, console, activemq, dns, mongoDB)
- (2) OSE Nodes

#**Puppet or openshift.sh?**#

Many OSE customers have been leveraging puppet to a deploy highly available OSE.
In this lab we will leverage the new oo-install command in OSE 2.2.  oo-install
cannot make the assumption that the customer has or wants puppet and so it
drives automation via the openshift.sh installation script.  oo-install will
interrogate the user, generate a YAML file based on the responses, and feed that
file to openshift.sh as required environment variables.  This mechanism offers a
cleaner and more reproducible installation.

Do not take this to mean we are not also investing in puppet.  Shortly after the
release of OSE 2.2 we will also have completed the porting of the Origin puppet
modules to work with OSE 2.2.  Thus customers will eventually have a choice
between puppet or oo-install.

# **What Has Already Been Done to the Environment** #

# **Environment**

## DNS Names
- student01 to student16

## Virtual machines:
Each user has 2 virtual machines with 4 GB, registered to the AMS2 Lab
Satellite, and subscribed to all the OpenShift channels. The names of
the vms per user are:
- OSE150209-st01-01 and OSE150209-st01-02, for example for student01.

## DHCP, DNS:
All VMs are registered in the DHCP server and will receive the same IP
in each boot. The DNS configuration is a work in progress (I don't have
writing permissions on the DNS server). In the meantime please use the
HOSTS configuration in the /etc/hosts of your system. See the
configuration attached for some copy+paste.


## Authentication steps:

1. Login to https://ipa.gps.hst.ams2.redhat.com/ or SSH to ipa.gps.hst.ams2.redhat.com to change the user password. (This is a mandatory step. To know why refer to http://www.freeipa.org/page/New_Passwords_Expired)
2. Go to https://rhev-manager.gps.hst.ams2.redhat.com/ovirt-engine/userportal/ and login with the login from
   1. Use the domain "gps.hst.ams2.redhat.com". Note that the access is made through the user portal.


**Packaging:**

OSE 2.2 requires RHEL 6.6.

    $ yum repolist

You will notice a combination of some RHUI channels enabled in addition to the
CDN.  Your EC2 image has already been registered with RHN through
subscription-manager and the needed channels subscribed to via
/etc/yum.repos.d/.  oo-install requires the following packages (on all systems):

    $ yum -y install ruby unzip curl ruby193-ruby yum-plugin-priorities

**CPU/MEM/NET**

We will be using the AWS t2.small.  That does not mean we recommend the t2.small
for production use by customers.  Something with 4GB of RAM or higher would be
better suited for production.  We have also already configured the AWS security
groups to allow for the required port access.

**Passwordless Access**

oo-install will be executed from your broker nodes and it will ssh into the other
EC2 instances and configure OpenShift.  In order to allow that to occur,
oo-install requires that a ssh private/public key exchange be used in place of a
normal password challenge.  Due to some last minute issues, we did not get SSH
keys distributed in the classroom image.

In order to distribute the SSH key, perform the following steps:

**Note:** Execute the following on the **broker1** host as the **root** user.

	$ ssh-keygen -t rsa

Accept all of the defaults and do not enter in a password.  The output should like the following:

	Generating public/private rsa key pair.
	Enter file in which to save the key (/home/ec2-user/.ssh/id_rsa):
	Enter passphrase (empty for no passphrase):
	Enter same passphrase again:
	Your identification has been saved in /home/ec2-user/.ssh/id_rsa.
	Your public key has been saved in /home/ec2-user/.ssh/id_rsa.pub.
	The key fingerprint is:
	5d:1a:b7:e5:3b:fb:71:96:7b:19:a3:59:b8:70:bf:34 root@broker1.userX.aussieshift.com
	The key's randomart image is:
	+--[ RSA 2048]----+
	|                 |
	|                 |
	|          . o .  |
	|         . = +   |
	|        S o . o  |
	|           . o =.|
	|            o OE*|
	|             +.**|
	|              .++|
	+-----------------+

Now that we have our SSH keypair created, we need to copy it to all of the hosts
that have been provided to you by the instructors:

**Note:** Execute the following on the **broker** host as the **root** user.
Remember to substitute your user number into the X (eg: user2).

	$ for host in broker1 broker2 amq1 amq2 mongo1 mongo2 mongo3 \
        node1 node2 node3 node4 node5 node6; do ssh-copy-id \
        -i ~/.ssh/id_rsa.pub "root@$host.userX.aussieshift.com"; done

You will be asked for the password of the *root* user on the each host.  Enter
in the credentials provided to you by the instructor.

The oo-install will default to using the root user.  You can use any user.
Should you wish to use another user, you need to insure that full access has
been granted via sudo and that sudo has been configured to be passwordless as
well.  For the purpose of this class, please use the root user when asked during
the oo-install.  We have modified /etc/ssh/sshd_config to allow for root login
access.

# **AWS Networking Concerns** #

During the oo-install, it is important that you remember to always use the
hostnames that are registered with IDM and the AWS public IP address (i.e.
54.x.x.x).  Your hostnames should be in the format of:

**component.userX.aussieshift.com**

For example, **broker1.user5.aussieshift.com or mongo3.user1.aussieshift.com**.

**Should you fail to remember to use the 54.x.x.x network with the IDM
hostnames, your oo-install will fail.**

# **Topology** #

![](http://wp-gadfly.rhcloud.com/wp-content/uploads/2014/10/topology.png)


One thing to note about the HA topology is the IP addresses used.  The only
component that will ask you for a virtual IP address or hostname is the broker.
This is because it is common to place a customer provided load balancer in front
of the brokers.  In order to save money in the classroom, we will not be
standing up this load balancer nor will we be purchasing additional IP addresses.
You will get hands on with the nginx load balancer later in the course when we
discuss HA applications.  With that in mind, when the oo-install asks for a
virtual hostname for the broker, we are going to give it a name (ie
broker-cluster) that will resolve to the IPaddess of one of the brokers.  Should
you be working with a real environment, you would have selected a new IP address
and given that to a load balancer.  AWS can also provide load balancers or round
robin routing for floating IP address for additional costs.

It is interesting to note that the ActiveMQ (amq) and mongo clusters do
not require a virtual or floating IP address.  This is because the clients that
call into them (ie mcollective uses ActiveMQ and the broker uses mongo) are
capable of trying multiple ipadresses when accessing those components.

# Download and Run oo-install #

Now that we have done our pre-work, we can actually issue the oo-install
command.  In this environment, due to the fact we are on AWS and are using a
development build of the product the check list you should run through is:


- **Machine Roles:** do I know what my boxes are what role I'm going to ask them to play (broker, amq, mongo, node)
- **Passwordless SSH:** do I have a passwordless ssh login to the instances from the broker where I will be issuing the oo-install command
- **Power User:** do I know what username I will be giving to oo-install to execute the commands on the instances.  Needs to have full command access through sudo or be root.  For the classroom environment just use root
- **Yum Channels:** does yum repolist show the correct repos setup.  Do I have the 5 prereq packages installed.
- **IP addresses:** do I have a virtual hostname decided for brokers?  Do I know which broker's IP address I plan to give the broker virtual hostname.

oo-install is not shipped with the product, but instead maintained out of cycle
and distributed on openshift.com.  First we need to download and execute the
*oo-install* script.  If you were a customer and this was a shipping product you
would issue the following command as the **ec2-user** or **root** user on the
**broker** host:

	$ sh <(curl -s https://install.openshift.com/ose) -r -e -a

The "-e" tells it to run in OpenShift Enterprise mode as opposed to Origin mode.
The "-a" tells it to install the product in a highly available configuration
(advanced-mode).

The curl command will download the installer and create a .openshift directory
in the user's home directory that issued the command.  It will then execute the
installation program.  The first question that is asked is as follows:

    Checking for necessary tools...
    ...looks good.
    Downloading oo-install package...
    Extracting oo-install to temporary directory...
    Starting oo-install...
    OpenShift Installer (Build 20141003-2025)
    ----------------------------------------------------------------------

    Welcome to OpenShift.

    This installer will guide you through a basic system deployment, based
    on one of the scenarios below.

    Select from the following installation scenarios.
    You can also type '?' for Help or 'q' to Quit:
    1. Install OpenShift Enterprise
    2. Add a Node to OpenShift Enterprise
    Type a selection and press <return>: 1

    It looks like you are running oo-install for the first time on a new
    system. The installer will guide you through the process of defining
    your OpenShift deployment.


We are selecting "1" so we can install OpenShift Enterprise for the first time.
As you can see, oo-install can be used to add additional nodes to the
environment as well.

The next section of the oo-install will configure the DNS.  For this class we
will be leveraging an external DNS provider, Red Hat IDM.  All of your instances
have already been registered with the environments IDM server in a previous lab.
With that in mind, we will answer **no** to having OpenShift install a bind
instance for us.

For the next question, you must enter the domain that has been given to you by
the instructor.  Remember that your domain is in the format of
**userX.aussieshift.com where X is your student number**.

The hostname of the DNS server is **idm.aussieshift.com**   The IP address of the
DNS server will be written on the board in the classroom.

This next question is a tricky one.  Currently you must answer the last question
about the DNSSEC key.  This is a bug in oo-install, in some ways. It assumes you
will only use DNSSEC if you are using existing DNS environments. For this class,
we will be using kerberos, and not DNSSEC, for sending our DNS updates. In this
field, simply type **junk**.  These are example entries.

    ----------------------------------------------------------------------
    DNS Configuration
    ----------------------------------------------------------------------

    First off, we will configure some DNS information for this system.

    Do you want me to install a new DNS server for OpenShift-hosted
    applications, or do you want this system to use an existing DNS
    server? (Answer 'yes' to have me install a DNS server.) (y/n/q/?) n

    All of your hosted applications will have a DNS name of the form:

    <app_name>-<owner_namespace>.<all_applications_domain>

    What domain name should be used for all of the hosted apps in your
    OpenShift system? |aussieshift.com| user0.aussieshift.com

    What is the hostname of the existing DNS server? idm.aussieshift.com

    What is the IP address of the existing DNS server? 54.172.7.19

    What is the DNSSEC key value for nsupdates against this DNS server? junk

    That's all of the DNS information that we need right now. Next, we
    need to gather information about the hosts in your OpenShift
    deployment.


After configuring the DNS, we start answering questions about the brokers.
Remember, we will be using 2 brokers in this environment.  The format for the
broker hostname is brokerN.userX.aussieshift.com where N is the component number
and X is your user number.

**This is important.**  As we answer questions about the layer components, you
must remember to use the IDM registered hostname (i.e.
broker1.user0.aussieshift.com) with the AWS public IP address.  oo-install will
find and default to the AWS private IP address.  This will not work and will
cause problems with the installation.  In all cases during the question and
answer, use the public IP address.

For the broker we are on, enter localhost when asked since the ssh
private/public key has only been configure for the other servers.

    ----------------------------------------------------------------------
    Broker Configuration
    ----------------------------------------------------------------------

    Okay. I'm going to need you to tell me about the host where you want
    to install the Broker.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing): broker1.user0.aussieshift.com

    Hostname / IP address for SSH access to broker1.user0.aussieshift.com from
    the host where you are running oo-install. You can say 'localhost' if
    you are running oo-install from the system that you are describing:
    |broker1.user0.aussieshift.com| localhost
    Using current user (root) for local installation.

    Detected IP address 172.31.38.238 at interface eth0 for this host.
    Do you want Nodes to use this IP information to reach this host?
    (y/n/q/?) n

    Specify the IP address that other hosts will use to connect to this
    host (Detected 172.31.38.238): 54.88.178.4
    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

    That's everything we need to know right now for this Broker.

    Do you want to configure an additional Broker? (y/n/q) y

    ----------------------------------------------------------------------
    Broker Configuration
    ----------------------------------------------------------------------

    Okay, please provide information about this Broker host.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing): broker2.user0.aussieshift.com

    Hostname / IP address for SSH access to broker2.user0.aussieshift.com from
    the host where you are running oo-install. You can say 'localhost' if
    you are running oo-install from the system that you are describing:
    |broker2.user0.aussieshift.com|

    Username for SSH access to broker2.user0.aussieshift.com: |root|

    Validating root@broker2.user0.aussieshift.com... looks good.

    Detected IP address 172.31.41.90 at interface eth0 for this host.
    Do you want Nodes to use this IP information to reach this host?
    (y/n/q/?) n

    Specify the IP address that other hosts will use to connect to this
    host (Detected 172.31.41.90): 54.165.205.3
    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

    That's everything we need to know right now for this Broker.

    Currently you have described the following host system(s):
    * broker1.user0.aussieshift.com (Broker)
    * broker2.user0.aussieshift.com (Broker)

    Do you want to configure an additional Broker? (y/n/q) n

    Moving on to the next role.

We have successfully answered the questions about the broker, now we will move on to the message server (ActiveMQ).  We will be using 2 message servers.

    ----------------------------------------------------------------------
    MsgServer Configuration
    ----------------------------------------------------------------------

    Okay. I'm going to need you to tell me about the host where you want
    to install the MsgServer.

    Do you want to assign the MsgServer role to one of the hosts that
    you've already described? (y/n/q/?) n

    Okay, please provide information about this MsgServer host.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing): amq1.user0.aussieshift.com

    Hostname / IP address for SSH access to amq1.user0.aussieshift.com
    from the host where you are running oo-install. You can say
    'localhost' if you are running oo-install from the system that you are
    describing: |amq1.user0.aussieshift.com|

    Username for SSH access to amq1.user0.aussieshift.com: |root|

    Validating root@amq1.user0.aussieshift.com... looks good.

    Detected IP address 172.31.41.93 at interface eth0 for this host.
    Do you want Nodes to use this IP information to reach this host?
    (y/n/q/?) n

    Specify the IP address that other hosts will use to connect to this
    host (Detected 172.31.41.93): 54.172.35.33
    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

    That's everything we need to know right now for this MsgServer.

    Currently you have described the following host system(s):
    * broker1.user0.aussieshift.com (Broker)
    * broker2.user0.aussieshift.com (Broker)
    * amq1.user0.aussieshift.com (MsgServer)

    Do you want to configure an additional MsgServer? (y/n/q) y

    ----------------------------------------------------------------------
    MsgServer Configuration
    ----------------------------------------------------------------------

    Do you want to assign the MsgServer role to one of the hosts that
    you've already described? (y/n/q/?) n

    Okay, please provide information about this MsgServer host.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing): amq2.user0.aussieshift.com

    Hostname / IP address for SSH access to amq2.user0.aussieshift.com
    from the host where you are running oo-install. You can say
    'localhost' if you are running oo-install from the system that you are
    describing: |amq2.user0.aussieshift.com|

    Username for SSH access to amq2.user0.aussieshift.com: |root|

    Validating root@amq2.user0.aussieshift.com... looks good.

    Detected IP address 172.31.41.85 at interface eth0 for this host.
    Do you want Nodes to use this IP information to reach this host?
    (y/n/q/?) n

    Specify the IP address that other hosts will use to connect to this
    host (Detected 172.31.41.85): 54.172.27.211
    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

    That's everything we need to know right now for this MsgServer.

    Currently you have described the following host system(s):
    * broker1.user0.aussieshift.com (Broker)
    * broker2.user0.aussieshift.com (Broker)
    * amq1.user0.aussieshift.com (MsgServer)
    * amq2.user0.aussieshift.com (MsgServer)

    Do you want to configure an additional MsgServer? (y/n/q) n

    Moving on to the next role.


By now you are figuring out the pattern of questions.  First we always ask if you want to add the role to an host we have already described.  Remember, some people will want to run the broker and message server on the same host.  We are open to combinations.  For this class we will be answering **no**.  After that we want the **full hostname that is registered with IDM**.  Then the **default hostname choice for ssh** addressing.  Select root for the user (for this class).  Make sure you do not take the default IP address and instead type in the **public AWS IP address**.  We are using **eth0**.


After the message servers, we tackle the mongoDB instances.  In order to shard a mongo install, you must have 3 hosts.  This is why the database has 3 instances while the broker and amq have only 2.

    ----------------------------------------------------------------------
    DBServer Configuration
    ----------------------------------------------------------------------

    Okay. I'm going to need you to tell me about the host where you want
    to install the DBServer.

    Do you want to assign the DBServer role to one of the hosts that
    you've already described? (y/n/q/?) n

    Okay, please provide information about this DBServer host.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing): mongo1.user0.aussieshift.com

    Hostname / IP address for SSH access to mongo1.user0.aussieshift.com from
    the host where you are running oo-install. You can say 'localhost' if
    you are running oo-install from the system that you are describing:
    |mongo1.user0.aussieshift.com|

    Username for SSH access to mongo1.user0.aussieshift.com: |root|

    Validating root@mongo1.user0.aussieshift.com... looks good.

    Detected IP address 172.31.41.94 at interface eth0 for this host.
    Do you want Nodes to use this IP information to reach this host?
    (y/n/q/?) n

    Specify the IP address that other hosts will use to connect to this
    host (Detected 172.31.41.94): 54.164.137.98
    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

    That's everything we need to know right now for this DBServer.

    Currently you have described the following host system(s):
    * broker1.user0.aussieshift.com (Broker)
    * broker2.user0.aussieshift.com (Broker)
    * mongo1.user0.aussieshift.com (DBServer)
    * amq1.user0.aussieshift.com (MsgServer)
    * amq2.user0.aussieshift.com (MsgServer)

    Do you want to configure an additional DBServer? (y/n/q) y

    ----------------------------------------------------------------------
    DBServer Configuration
    ----------------------------------------------------------------------

    Do you want to assign the DBServer role to one of the hosts that
    you've already described? (y/n/q/?) n

    Okay, please provide information about this DBServer host.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing): mongo2.user0.aussieshift.com

    Hostname / IP address for SSH access to mongo2.user0.aussieshift.com from
    the host where you are running oo-install. You can say 'localhost' if
    you are running oo-install from the system that you are describing:
    |mongo2.user0.aussieshift.com|

    Username for SSH access to mongo2.user0.aussieshift.com: |root|

    Validating root@mongo2.user0.aussieshift.com... looks good.

    Detected IP address 172.31.41.89 at interface eth0 for this host.
    Do you want Nodes to use this IP information to reach this host?
    (y/n/q/?) n

    Specify the IP address that other hosts will use to connect to this
    host (Detected 172.31.41.89): 54.165.150.40
    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

    That's everything we need to know right now for this DBServer.

    Currently you have described the following host system(s):
    * broker1.user0.aussieshift.com (Broker)
    * broker2.user0.aussieshift.com (Broker)
    * mongo1.user0.aussieshift.com (DBServer)
    * mongo2.user0.aussieshift.com (DBServer)
    * amq1.user0.aussieshift.com (MsgServer)
    * amq2.user0.aussieshift.com (MsgServer)

    Do you want to configure an additional DBServer? (y/n/q) y

    ----------------------------------------------------------------------
    DBServer Configuration
    ----------------------------------------------------------------------

    Do you want to assign the DBServer role to one of the hosts that
    you've already described? (y/n/q/?) n

    Okay, please provide information about this DBServer host.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing): mongo3.user0.aussieshift.com

    Hostname / IP address for SSH access to mongo3.user0.aussieshift.com from
    the host where you are running oo-install. You can say 'localhost' if
    you are running oo-install from the system that you are describing:
    |mongo3.user0.aussieshift.com|

    Username for SSH access to mongo3.user0.aussieshift.com: |root|

    Validating root@mongo3.user0.aussieshift.com... looks good.

    Detected IP address 172.31.41.86 at interface eth0 for this host.
    Do you want Nodes to use this IP information to reach this host?
    (y/n/q/?) n

    Specify the IP address that other hosts will use to connect to this
    host (Detected 172.31.41.86): 54.172.33.128
    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

    That's everything we need to know right now for this DBServer.

    Currently you have described the following host system(s):
    * broker1.user0.aussieshift.com (Broker)
    * broker2.user0.aussieshift.com (Broker)
    * mongo1.user0.aussieshift.com (DBServer)
    * mongo2.user0.aussieshift.com (DBServer)
    * mongo3.user0.aussieshift.com (DBServer)
    * amq1.user0.aussieshift.com (MsgServer)
    * amq2.user0.aussieshift.com (MsgServer)

    Do you want to configure an additional DBServer? (y/n/q) n

    Moving on to the next role.

 The last Q&A section is on the nodes.  You will have to answer the questions about all 6 nodes.  In order to save space, I will only show the last node below.

    ----------------------------------------------------------------------
    Node Configuration
    ----------------------------------------------------------------------

    Do you want to assign the Node role to one of the hosts that you've
    already described? (y/n/q/?) n

    Okay, please provide information about this Node host.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing): node6.user0.aussieshift.com

    Hostname / IP address for SSH access to node6.user0.aussieshift.com from
    the host where you are running oo-install. You can say 'localhost' if
    you are running oo-install from the system that you are describing:
    |node6.user0.aussieshift.com|

    Username for SSH access to node6.user0.aussieshift.com: |root|

    Validating root@node6.user0.aussieshift.com... looks good.

    Detected IP address 172.31.41.82 at interface eth0 for this host.
    Do you want to use this as the public IP information for this Node?
    (y/n/q/?) n

    Specify the public IP address for this Node (Detected 172.31.41.82): 54.165.206.153
    Specify the network interface that this Node will use to route
    Application traffic (Detected 'eth0'): |eth0|

    That's everything we need to know right now for this Node.

    Currently you have described the following host system(s):
    * broker1.user0.aussieshift.com (Broker)
    * broker2.user0.aussieshift.com (Broker)
    * mongo1.user0.aussieshift.com (DBServer)
    * mongo2.user0.aussieshift.com (DBServer)
    * mongo3.user0.aussieshift.com (DBServer)
    * amq1.user0.aussieshift.com (MsgServer)
    * amq2.user0.aussieshift.com (MsgServer)
    * node.user0.aussieshift.com (Node)
    * node2.user0.aussieshift.com (Node)
    * node3.user0.aussieshift.com (Node)
    * node4.user0.aussieshift.com (Node)
    * node5.user0.aussieshift.com (Node)
    * node6.user0.aussieshift.com (Node)

    Do you want to configure an additional Node? (y/n/q) n


We have completed the Q&A regarding all 13 instances.  Now we will answer a few more questions about the environment.  Please allow OpenShift to autogenerate the passwords for the accounts in this development build.  And take the default anwsers for the replica set name, replica key value, and amq cluster password.  Lastly, leverage the DNS name you choose earlier for the broker cluster virtual hostname.  This is the DNS record that is using an IP address to one of your 2 brokers.

    Do you want to manually specify usernames and passwords for the
    various supporting service accounts? Answer 'N' to have the values
    generated for you (y/n/q) n

    It looks like you're setting up a High Availability OpenShift
    deployment. We're going to need to set some HA-related settings.

    Please enter a virtual hostname for the Broker cluster: broker-cluster

    What replica set name should the DB servers use? |openshift|

    What replica key value should the DB servers use?
    |Oad6aLIlFAcpQwmTbDX1Q|

    What password should the MsgServer cluster use for inter-cluster
    communication? |mZoy2f3uKyG7PCDVHJL5aA|

Lastly the oo-install will summarize the environment.  Pay attention to the passwords that are displayed in the boxes.  The demo user that we will login with later requires that password.

    Here are the details of your current deployment.

    DNS Settings
      * OpenShift will use existing DNS
      * Application Domain: user0.aussieshift.com
      * DNS Host IP: 54.172.7.19
      * DNSSEC key: junk

    Global Gear Settings
    +-------------------------+-------+
    | Valid Gear Sizes| small |
    | User Default Gear Sizes | small |
    | Default Gear Size   | small |
    +-------------------------+-------+

    Account Settings
    +----------------------------+------------------------------------------------------------------+
    | OpenShift Console User | demo |
    | OpenShift Console Password | UzdUFoIHwPuPNQkh6K2A |
    | Broker Session Secret  | 3hCLi9vvQMKXVEjsJ90A |
    | Console Session Secret | MEH3sKwiBz7vyYlezyWEpg   |
    | Broker Auth Salt   | vu4hHz2fmIhWAIwDQaaoJA   |

    | Broker Auth Private Key| -----BEGIN RSA PRIVATE KEY-----  |
    || MIIEowIBAAKCAQEAyer79FkJfGwSH4RdxFWMXEngs4HkMBd7DtSk5MZTG21dzo6q |
    || 0TkRUHmksVDMjc6d6QfgyEsyO+IcDR64KNCLpqcs95Owbyf6/e7kqkuOHYL+36vt |
    || VzYP2dtOUKBEwl3sBoAuqcpD6pHO2PgNPM4pzV/vZsuKACSujy+MFt3+7IrcGAnN |
    || qQFFuEkvXeern3fDRdI431426RgADooxUREdytoEzjDSfGLLAT+MdVs+oM606+AL |
    || /lRAdPzaE4qJ/jnVkbKxQEHDElhtuvk0iT6Dd/IODuw4K2m5zUwBcmIrODxGAPpS |
    || LubK+LUKPTN015y5BDXq6EMaD5ozQv+BsX+13QIDAQABAoIBABLiH/gFD6cMMFG0 |
    || PlSrL3o+Cn6fKij5OS/04QroJUOOYdR8cSsp7B2bkrRmewrUBN6TNwlkRulkxvzP |
    || H6fpgPXv8nug20I5+fYfjlECyeKmqpFecc7TJn5YTSWrJ2MKB5XADR0h5mIciryt |
    || zgcevLgRPcFeTaSfyZdoefB1iG60jyLdM/Jvq6YpmL68EcQcPoy7inNoV1KOyjqI |
    || bVu1ZCRXPuhU1sIa4JhBzXb68XhomO9q1O7NLUqV0/8ocCN3h5BCkH2RbPTi2Day |
    || R+xeR8TynIx+OX6Dk4nITSzrmvxj6R11sdytdOwyqLmjaooTxW82ns4TA+jJpsCz |
    || 0Hll5h0CgYEA9eVbfLetzA8EFXpgHb9Jc9Nn7oSi9GMymrorQQdB0ONx0/A4zEaP |
    || rQwNEhuZgPrQoewCEHamygZUqvnODTHcxBxwrzX7syNjYH7ZAICHeGo6yCl3bQZh |
    || BJjfwnfAosXSwqS4JH1lwM0p0sLc1tyokVzlXRkfuecghmbXpwsMDgcCgYEA0jcC |
    || vQ4UMTMYnxM8rxMDTYbLZRaoRDKwMQWRW6DIVM807i4iKkJJdcqo/Lg4bbBDxY9P |
    || dAw/JdQmV73z0Y0Drpm3bhcO8o6a89vUEy1BCoBFeXnpVB1t3opuYMK2QRCCwxV3 |
    || 9osQR/XYyMdP4os1qyRTb2oQygzCXzKCcWJlI/sCgYEA8CmwPlKD2976pSOeBs/S |
    || pN7hDrPbGIheX4LfRicZYDUU8uQYBWQRZfl0NrBgL/pIlS2WIpBQfNbMESXk2zxN |
    || G/mPEYHPMPqqUA/0UCo4piJTATaG3yQw07WgLiaaLiC6pcMN2w3iuPlpFOGfofdo |
    || aHlrx48HTqHwQXTmwc7nWjcCgYAHlFox7N8HgxshKTVn7pyQ4ApXY8C/bMBzlArQ |
    || rfRrMmlrKRisQ2WYrKz5J79JHTDkX61ytrpUJ9kWEtBGvvniAsLdYlF0p3Wo00VL |
    || R7dvpH5cyeuCz+jVPFKMhJjDsc+1LwH7TrpQjem6G42i0ngl6pJjkwR19I3RluWj |
    || JvQUnwKBgAKod+rq0wj09LA8FYNs8MxigvW72z2NuX4zkeCdbDbIG4gYNGhseSCx |
    || lvyKN8XHFH2nONFhgpSDh/BBxX4ROtRqzA6ScRbQZnVorDl26n9pEQgFQodComUI |
    || ER0DfjuRMSHHiZyNOpfCH/vPC8O/FTDHBOLt70WVhC4Y74kZwRcQ |
    || -----END RSA PRIVATE KEY-----|
    | MCollective User   | mcollective  |
    | MCollective Password   | 3GhRjr2eMHQANjfXJ9lGA|
    | MongoDB Broker User| openshift|
    | MongoDB Broker Password| 49l6D7LlZ4mwsTBv11cUDQ   |
    +----------------------------+------------------------------------------------------------------+

    Node Districts
    +----------+-----------+------------------------------------------------------------------------------------------------------------------------------------------------+
    | District | Gear Size | Nodes  |
    +----------+-----------+------------------------------------------------------------------------------------------------------------------------------------------------+
    | Default  | small | node.user0.aussieshift.com,node2.user0.aussieshift.com,node3.user0.aussieshift.com,node4.user0.aussieshift.com,node5.user0.aussieshift.com,node6.user0.aussieshift.com |
    +----------+-----------+------------------------------------------------------------------------------------------------------------------------------------------------+

    Role Assignments
    +-------------+------------------------------+
    | Brokers | broker1.user0.aussieshift.com |
    | | broker2.user0.aussieshift.com|
    | NameServers | -|
    | MsgServers  | amq1.user0.aussieshift.com  |
    | | amq2.user0.aussieshift.com |
    | DBServers   | mongo1.user0.aussieshift.com  |
    | | mongo2.user0.aussieshift.com |
    | | mongo3.user0.aussieshift.com |
    | Nodes   | node.user0.aussieshift.com   |
    | | node2.user0.aussieshift.com  |
    | | node3.user0.aussieshift.com  |
    | | node4.user0.aussieshift.com  |
    | | node5.user0.aussieshift.com  |
    | | node6.user0.aussieshift.com  |
    +-------------+------------------------------+

    High Availability Configuration
    +----------------------------+------------------------+
    | HA Broker DNS  | broker-cluster |
    | MongoDB Replica Set Name   | openshift  |
    | MongoDB Replica Key| Oad6aLIlFAcpQwmTbDX1Q  |
    | MsgServer Cluster Password | mZoy2f3uKyG7PCDVHJL5aA |
    +----------------------------+------------------------+

    Host Information
    +------------------------------+-----------+
    | Hostname | Roles |
    +------------------------------+-----------+
    | broker1.user0.aussieshift.com | Broker|
    | broker2.user0.aussieshift.com| Broker|
    | mongo1.user0.aussieshift.com  | DBServer  |
    | mongo2.user0.aussieshift.com | DBServer  |
    | mongo3.user0.aussieshift.com | DBServer  |
    | amq1.user0.aussieshift.com  | MsgServer |
    | amq2.user0.aussieshift.com | MsgServer |
    | node.user0.aussieshift.com   | Node  |
    | node2.user0.aussieshift.com  | Node  |
    | node3.user0.aussieshift.com  | Node  |
    | node4.user0.aussieshift.com  | Node  |
    | node5.user0.aussieshift.com  | Node  |
    | node6.user0.aussieshift.com  | Node  |
    +------------------------------+-----------+

Finally, oo-install will ask questions about your subscription.  Remember, due to the fact we are using a development channel, we have already setup the channels for the hosts.  So in this question, please answer no to both questions.

    Choose an action:
    1. Change the deployment configuration
    2. View the full host configuration details
    3. Proceed with deployment
    Type a selection and press <return>: 3

    Here is the subscription configuration that the installer will use for
    this deployment.
    +---------+-------+
    | Setting | Value |
    +---------+-------+
    | type| none  |
    +---------+-------+

    Do you want to make any changes to the subscription info in the
    configuration file? (y/n/q/?) n

    Do you want to set any temporary subscription settings for this
    installation only? (y/n/q/?) n

At this point, oo-install has all the information it needs and it will start installing and configuring the environment.

    Preflight check: verifying system and resource availability.

    Checking broker1.user0.aussieshift.com:
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking broker2.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking amq1.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking amq2.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking mongo1.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking mongo2.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking mongo3.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking node.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking node2.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking node3.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking node4.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking node5.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Checking node6.user0.aussieshift.com:
    * SSH connection succeeded
    * Target host is running Red Hat Enterprise Linux
    * Located getenforce
    * SELinux is running in enforcing mode
    * Located yum

    Deploying workflow 'enterprise_deploy'.

    broker1.user0.aussieshift.com : Configuring package repositories.
    broker2.user0.aussieshift.com : Configuring package repositories.
    amq1.user0.aussieshift.com : Configuring package repositories.
    amq2.user0.aussieshift.com : Configuring package repositories.
    mongo2.user0.aussieshift.com : Configuring package repositories.
    node.user0.aussieshift.com : Configuring package repositories.
    node2.user0.aussieshift.com : Configuring package repositories.
    node3.user0.aussieshift.com : Configuring package repositories.
    mongo1.user0.aussieshift.com : Configuring package repositories.
    mongo3.user0.aussieshift.com : Configuring package repositories.
    node4.user0.aussieshift.com : Configuring package repositories.
    node5.user0.aussieshift.com : Configuring package repositories.
    node6.user0.aussieshift.com : Configuring package repositories.
    broker1.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    broker1.user0.aussieshift.com : Completed configuring package repositories.
    amq1.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    amq1.user0.aussieshift.com : Completed configuring package repositories.
    mongo3.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    mongo3.user0.aussieshift.com : Completed configuring package repositories.
    node2.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    mongo2.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    node.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    mongo1.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    node2.user0.aussieshift.com : Completed configuring package repositories.
    mongo2.user0.aussieshift.com : Completed configuring package repositories.
    node.user0.aussieshift.com : Completed configuring package repositories.
    mongo1.user0.aussieshift.com : Completed configuring package repositories.
    node4.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    node6.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    broker2.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    node6.user0.aussieshift.com : Completed configuring package repositories.
    amq2.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    node4.user0.aussieshift.com : Completed configuring package repositories.
    broker2.user0.aussieshift.com : Completed configuring package repositories.
    amq2.user0.aussieshift.com : Completed configuring package repositories.
    node3.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    node5.user0.aussieshift.com:
    OpenShift: Begin preflight validation.
    OpenShift: Completed preflight validation.
    OpenShift: Begin configuring repos.
    OpenShift: Completed configuring repos.
    node3.user0.aussieshift.com : Completed configuring package repositories.
    node5.user0.aussieshift.com : Completed configuring package repositories.

    Step Prepare completed successfully.

    broker1.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    broker2.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    amq1.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    amq2.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    mongo1.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    mongo2.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    mongo3.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    node.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    node2.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    node3.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    node4.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    node5.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    node6.user0.aussieshift.com : Installing RPMs. This step can take up to an hour.
    amq1.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install activemq
    OpenShift: Completed installing RPMs.
    amq1.user0.aussieshift.com : Completed installing RPMs.
    amq2.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install activemq
    OpenShift: Completed installing RPMs.
    mongo3.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install mongodb-server mongodb
    OpenShift: Completed installing RPMs.
    amq2.user0.aussieshift.com : Completed installing RPMs.
    mongo3.user0.aussieshift.com : Completed installing RPMs.
    mongo1.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install mongodb-server mongodb
    OpenShift: Completed installing RPMs.
    mongo1.user0.aussieshift.com : Completed installing RPMs.
    mongo2.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install mongodb-server mongodb
    OpenShift: Completed installing RPMs.
    mongo2.user0.aussieshift.com : Completed installing RPMs.
    broker1.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install openshift-origin-broker openshift-origin-broker-util rubygem-openshift-origin-msg-broker-mcollective ruby193-mcollective-client rubygem-openshift-origin-auth-remote-user rubygem-openshift-origin-dns-nsupdate openshift-origin-console rubygem-openshift-origin-admin-console
    OpenShift: yum install rhc
    OpenShift: Completed installing RPMs.
    broker1.user0.aussieshift.com : Completed installing RPMs.
    broker2.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install openshift-origin-broker openshift-origin-broker-util rubygem-openshift-origin-msg-broker-mcollective ruby193-mcollective-client rubygem-openshift-origin-auth-remote-user rubygem-openshift-origin-dns-nsupdate openshift-origin-console rubygem-openshift-origin-admin-console
    OpenShift: yum install rhc
    OpenShift: Completed installing RPMs.
    broker2.user0.aussieshift.com : Completed installing RPMs.
    node2.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install rubygem-openshift-origin-node ruby193-rubygem-passenger-native openshift-origin-node-util ruby193-mcollective openshift-origin-msg-node-mcollective policycoreutils-python rubygem-openshift-origin-container-selinux rubygem-openshift-origin-frontend-nodejs-websocket rubygem-openshift-origin-frontend-haproxy-sni-proxy rsyslog7 rsyslog7-mmopenshift rubygem-openshift-origin-frontend-apache-mod-rewrite
    OpenShift: yum install openshift-origin-cartridge-cron openshift-origin-cartridge-dependencies-recommended-jbossews openshift-origin-cartridge-dependencies-recommended-nodejs openshift-origin-cartridge-dependencies-recommended-perl openshift-origin-cartridge-dependencies-recommended-php openshift-origin-cartridge-dependencies-recommended-python openshift-origin-cartridge-dependencies-recommended-ruby openshift-origin-cartridge-diy openshift-origin-cartridge-haproxy openshift-origin-cartridge-jbossews openshift-origin-cartridge-jenkins openshift-origin-cartridge-jenkins-client openshift-origin-cartridge-mongodb openshift-origin-cartridge-mysql openshift-origin-cartridge-nodejs openshift-origin-cartridge-perl openshift-origin-cartridge-php openshift-origin-cartridge-postgresql openshift-origin-cartridge-python openshift-origin-cartridge-ruby
    OpenShift: Completed installing RPMs.
    node2.user0.aussieshift.com : Completed installing RPMs.
    node4.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install rubygem-openshift-origin-node ruby193-rubygem-passenger-native openshift-origin-node-util ruby193-mcollective openshift-origin-msg-node-mcollective policycoreutils-python rubygem-openshift-origin-container-selinux rubygem-openshift-origin-frontend-nodejs-websocket rubygem-openshift-origin-frontend-haproxy-sni-proxy rsyslog7 rsyslog7-mmopenshift rubygem-openshift-origin-frontend-apache-mod-rewrite
    OpenShift: yum install openshift-origin-cartridge-cron openshift-origin-cartridge-dependencies-recommended-jbossews openshift-origin-cartridge-dependencies-recommended-nodejs openshift-origin-cartridge-dependencies-recommended-perl openshift-origin-cartridge-dependencies-recommended-php openshift-origin-cartridge-dependencies-recommended-python openshift-origin-cartridge-dependencies-recommended-ruby openshift-origin-cartridge-diy openshift-origin-cartridge-haproxy openshift-origin-cartridge-jbossews openshift-origin-cartridge-jenkins openshift-origin-cartridge-jenkins-client openshift-origin-cartridge-mongodb openshift-origin-cartridge-mysql openshift-origin-cartridge-nodejs openshift-origin-cartridge-perl openshift-origin-cartridge-php openshift-origin-cartridge-postgresql openshift-origin-cartridge-python openshift-origin-cartridge-ruby
    OpenShift: Completed installing RPMs.
    node4.user0.aussieshift.com : Completed installing RPMs.
    node6.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install rubygem-openshift-origin-node ruby193-rubygem-passenger-native openshift-origin-node-util ruby193-mcollective openshift-origin-msg-node-mcollective policycoreutils-python rubygem-openshift-origin-container-selinux rubygem-openshift-origin-frontend-nodejs-websocket rubygem-openshift-origin-frontend-haproxy-sni-proxy rsyslog7 rsyslog7-mmopenshift rubygem-openshift-origin-frontend-apache-mod-rewrite
    OpenShift: yum install openshift-origin-cartridge-cron openshift-origin-cartridge-dependencies-recommended-jbossews openshift-origin-cartridge-dependencies-recommended-nodejs openshift-origin-cartridge-dependencies-recommended-perl openshift-origin-cartridge-dependencies-recommended-php openshift-origin-cartridge-dependencies-recommended-python openshift-origin-cartridge-dependencies-recommended-ruby openshift-origin-cartridge-diy openshift-origin-cartridge-haproxy openshift-origin-cartridge-jbossews openshift-origin-cartridge-jenkins openshift-origin-cartridge-jenkins-client openshift-origin-cartridge-mongodb openshift-origin-cartridge-mysql openshift-origin-cartridge-nodejs openshift-origin-cartridge-perl openshift-origin-cartridge-php openshift-origin-cartridge-postgresql openshift-origin-cartridge-python openshift-origin-cartridge-ruby
    OpenShift: Completed installing RPMs.
    node6.user0.aussieshift.com : Completed installing RPMs.
    node5.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install rubygem-openshift-origin-node ruby193-rubygem-passenger-native openshift-origin-node-util ruby193-mcollective openshift-origin-msg-node-mcollective policycoreutils-python rubygem-openshift-origin-container-selinux rubygem-openshift-origin-frontend-nodejs-websocket rubygem-openshift-origin-frontend-haproxy-sni-proxy rsyslog7 rsyslog7-mmopenshift rubygem-openshift-origin-frontend-apache-mod-rewrite
    OpenShift: yum install openshift-origin-cartridge-cron openshift-origin-cartridge-dependencies-recommended-jbossews openshift-origin-cartridge-dependencies-recommended-nodejs openshift-origin-cartridge-dependencies-recommended-perl openshift-origin-cartridge-dependencies-recommended-php openshift-origin-cartridge-dependencies-recommended-python openshift-origin-cartridge-dependencies-recommended-ruby openshift-origin-cartridge-diy openshift-origin-cartridge-haproxy openshift-origin-cartridge-jbossews openshift-origin-cartridge-jenkins openshift-origin-cartridge-jenkins-client openshift-origin-cartridge-mongodb openshift-origin-cartridge-mysql openshift-origin-cartridge-nodejs openshift-origin-cartridge-perl openshift-origin-cartridge-php openshift-origin-cartridge-postgresql openshift-origin-cartridge-python openshift-origin-cartridge-ruby
    OpenShift: Completed installing RPMs.
    node5.user0.aussieshift.com : Completed installing RPMs.
    node.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install rubygem-openshift-origin-node ruby193-rubygem-passenger-native openshift-origin-node-util ruby193-mcollective openshift-origin-msg-node-mcollective policycoreutils-python rubygem-openshift-origin-container-selinux rubygem-openshift-origin-frontend-nodejs-websocket rubygem-openshift-origin-frontend-haproxy-sni-proxy rsyslog7 rsyslog7-mmopenshift rubygem-openshift-origin-frontend-apache-mod-rewrite
    OpenShift: yum install openshift-origin-cartridge-cron openshift-origin-cartridge-dependencies-recommended-jbossews openshift-origin-cartridge-dependencies-recommended-nodejs openshift-origin-cartridge-dependencies-recommended-perl openshift-origin-cartridge-dependencies-recommended-php openshift-origin-cartridge-dependencies-recommended-python openshift-origin-cartridge-dependencies-recommended-ruby openshift-origin-cartridge-diy openshift-origin-cartridge-haproxy openshift-origin-cartridge-jbossews openshift-origin-cartridge-jenkins openshift-origin-cartridge-jenkins-client openshift-origin-cartridge-mongodb openshift-origin-cartridge-mysql openshift-origin-cartridge-nodejs openshift-origin-cartridge-perl openshift-origin-cartridge-php openshift-origin-cartridge-postgresql openshift-origin-cartridge-python openshift-origin-cartridge-ruby
    OpenShift: Completed installing RPMs.
    node.user0.aussieshift.com : Completed installing RPMs.
    node3.user0.aussieshift.com:
    OpenShift: Begin installing RPMs.
    OpenShift: yum update
    OpenShift: yum install ntp ntpdate wget
    OpenShift: yum install rubygem-openshift-origin-node ruby193-rubygem-passenger-native openshift-origin-node-util ruby193-mcollective openshift-origin-msg-node-mcollective policycoreutils-python rubygem-openshift-origin-container-selinux rubygem-openshift-origin-frontend-nodejs-websocket rubygem-openshift-origin-frontend-haproxy-sni-proxy rsyslog7 rsyslog7-mmopenshift rubygem-openshift-origin-frontend-apache-mod-rewrite
    OpenShift: yum install openshift-origin-cartridge-cron openshift-origin-cartridge-dependencies-recommended-jbossews openshift-origin-cartridge-dependencies-recommended-nodejs openshift-origin-cartridge-dependencies-recommended-perl openshift-origin-cartridge-dependencies-recommended-php openshift-origin-cartridge-dependencies-recommended-python openshift-origin-cartridge-dependencies-recommended-ruby openshift-origin-cartridge-diy openshift-origin-cartridge-haproxy openshift-origin-cartridge-jbossews openshift-origin-cartridge-jenkins openshift-origin-cartridge-jenkins-client openshift-origin-cartridge-mongodb openshift-origin-cartridge-mysql openshift-origin-cartridge-nodejs openshift-origin-cartridge-perl openshift-origin-cartridge-php openshift-origin-cartridge-postgresql openshift-origin-cartridge-python openshift-origin-cartridge-ruby
    OpenShift: Completed installing RPMs.
    node3.user0.aussieshift.com : Completed installing RPMs.

    Step Install RPMs completed successfully.

    broker1.user0.aussieshift.com : Configuring host.
    broker2.user0.aussieshift.com : Configuring host.
    amq1.user0.aussieshift.com : Configuring host.
    amq2.user0.aussieshift.com : Configuring host.
    mongo1.user0.aussieshift.com : Configuring host.
    mongo2.user0.aussieshift.com : Configuring host.
    mongo3.user0.aussieshift.com : Configuring host.
    node.user0.aussieshift.com : Configuring host.
    node2.user0.aussieshift.com : Configuring host.
    node3.user0.aussieshift.com : Configuring host.
    node4.user0.aussieshift.com : Configuring host.
    node5.user0.aussieshift.com : Configuring host.
    OpenShift: Completed configuring host.
    broker1.user0.aussieshift.com : Completed configuring host.
    amq1.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    amq1.user0.aussieshift.com : Completed configuring host.
    broker2.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    amq2.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    broker2.user0.aussieshift.com : Completed configuring host.
    amq2.user0.aussieshift.com : Completed configuring host.
    node6.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    mongo2.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    node6.user0.aussieshift.com : Completed configuring host.
    node2.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    mongo2.user0.aussieshift.com : Completed configuring host.
    mongo1.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    node4.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    node2.user0.aussieshift.com : Completed configuring host.
    mongo3.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    node5.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    node3.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    mongo1.user0.aussieshift.com : Completed configuring host.
    node.user0.aussieshift.com:
    OpenShift: Begin configuring host.
    OpenShift: Completed configuring host.
    node4.user0.aussieshift.com : Completed configuring host.
    node5.user0.aussieshift.com : Completed configuring host.
    node.user0.aussieshift.com : Completed configuring host.
    node3.user0.aussieshift.com : Completed configuring host.
    mongo3.user0.aussieshift.com : Completed configuring host.

    Step Configure Host completed successfully.

    mongo1.user0.aussieshift.com : Configuring OpenShift.
    mongo2.user0.aussieshift.com : Configuring OpenShift.
    mongo3.user0.aussieshift.com : Configuring OpenShift.
    mongo2.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    mongo2.user0.aussieshift.com : Completed configuring OpenShift.
    mongo3.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    mongo3.user0.aussieshift.com : Completed configuring OpenShift.
    mongo1.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    mongo1.user0.aussieshift.com : Completed configuring OpenShift.

    Step Configure OpenShift completed successfully.

    mongo1.user0.aussieshift.com : Configuring MongoDB replica set.
    mongo1.user0.aussieshift.com:
    OpenShift: Waiting for MongoDB to start (19:45:01)...
    OpenShift: MongoDB is ready! (19:45:01)
    OpenShift: Waiting for MongoDB to start (19:45:09)...
    OpenShift: MongoDB is ready! (19:45:09)
    mongo1.user0.aussieshift.com : Completed configuring MongoDB replica set.

    Step Configure Replica Sets completed successfully.

    amq1.user0.aussieshift.com : Configuring OpenShift.
    amq2.user0.aussieshift.com : Configuring OpenShift.
    amq2.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    amq1.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    amq2.user0.aussieshift.com : Completed configuring OpenShift.
    amq1.user0.aussieshift.com : Completed configuring OpenShift.

    Step Configure OpenShift completed successfully.

    broker1.user0.aussieshift.com : Configuring OpenShift.
    broker2.user0.aussieshift.com : Configuring OpenShift.
    broker1.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    broker1.user0.aussieshift.com : Completed configuring OpenShift.
    broker2.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    broker2.user0.aussieshift.com : Completed configuring OpenShift.

    Step Configure OpenShift completed successfully.

    node.user0.aussieshift.com : Configuring OpenShift.
    node2.user0.aussieshift.com : Configuring OpenShift.
    node3.user0.aussieshift.com : Configuring OpenShift.
    node4.user0.aussieshift.com : Configuring OpenShift.
    node5.user0.aussieshift.com : Configuring OpenShift.
    node6.user0.aussieshift.com : Configuring OpenShift.

    node.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    node.user0.aussieshift.com : Completed configuring OpenShift.
    node4.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    node2.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    node4.user0.aussieshift.com : Completed configuring OpenShift.
    node2.user0.aussieshift.com : Completed configuring OpenShift.
    node6.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    node6.user0.aussieshift.com : Completed configuring OpenShift.
    node5.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    node5.user0.aussieshift.com : Completed configuring OpenShift.
    node3.user0.aussieshift.com:
    OpenShift: Begin configuring OpenShift.
    OpenShift: Completed configuring OpenShift.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    node3.user0.aussieshift.com : Completed configuring OpenShift.

    Step Configure OpenShift completed successfully.

    broker1.user0.aussieshift.com : Performing post deploy steps.
    broker1.user0.aussieshift.com:
    OpenShift: Begin post deployment steps.
    OpenShift: Begin restarting services.
    OpenShift: Completed restarting services.
    OpenShift: Configuring districts.
    OpenShift: Adding nodes: node.user0.aussieshift.com,node2.user0.aussieshift.com,node3.user0.aussieshift.com,node4.user0.aussieshift.com,node5.user0.aussieshift.com,node6.user0.aussieshift.com with profile: small to district: Default.
    OpenShift: Completed configuring districts.
    OpenShift: Completed post deployment steps.
    broker1.user0.aussieshift.com : Completed post deploy steps.

    Step Post Deploy completed successfully.

    broker1.user0.aussieshift.com : Validating installation.
    broker2.user0.aussieshift.com : Validating installation.
    node.user0.aussieshift.com : Validating installation.
    node2.user0.aussieshift.com : Validating installation.
    node3.user0.aussieshift.com : Validating installation.
    node4.user0.aussieshift.com : Validating installation.
    node5.user0.aussieshift.com : Validating installation.
    node6.user0.aussieshift.com : Validating installation.

During the validating section of the oo-install at the end, you will see output from oo-diagnostic on the screen.  Remember above when we entered a bogus junk entry for the DNSSEC question.  This is going to cause error to be displayed.  Please do not worry about them.  Before we are about to do anything with the environment, we will need to configure OpenShift to authenticate against the IDM kerberos service.  The DNSSEC error will look like this:

    broker2.user0.aussieshift.com:
    OpenShift: Begin running oo-diagnostics.
    OpenShift: oo-diagnostics output - FAIL: run_script
    OpenShift: oo-diagnostics output - oo-accept-broker had errors:
    OpenShift: oo-diagnostics output - --BEGIN OUTPUT--
    OpenShift: oo-diagnostics output - could not create key from j: bad base64 encoding
    OpenShift: oo-diagnostics output - syntax error
    OpenShift: oo-diagnostics output - FAIL: error adding txt record name testrecord.user0.aussieshift.com to server 54.172.7.19: this_is_a_test
    OpenShift: oo-diagnostics output - -- is the nameserver running, reachable, and key auth working?
    OpenShift: oo-diagnostics output - FAIL: txt record testrecord.user0.aussieshift.com does not resolve on server 54.172.7.19
    OpenShift: oo-diagnostics output - could not create key from j: bad base64 encoding
    OpenShift: oo-diagnostics output - syntax error
    OpenShift: oo-diagnostics output - FAIL: error deleteing txt record name testrecord.user0.aussieshift.com to server 54.172.7.19:
    OpenShift: oo-diagnostics output - -- is the nameserver running, reachable, and key auth working?
    OpenShift: oo-diagnostics output - 3 ERRORS
    OpenShift: oo-diagnostics output -
    OpenShift: oo-diagnostics output - --END oo-accept-broker OUTPUT--


You will also see some oo-diagnostic errors about oo-admin-yum-validator, self signd certs, and subscription types.  Those are because we are using a development channel and can be ignored as well.

The product has been completely installed in an HA configuration.  Congratulations!


##**Verifying the install**

Once the installation has completed, you can verify that the node have been added to your infrastructure by running the following command on the **broker host**:

	$ sudo oo-mco ping

You should see the following results:

	node2.user0.aussieshift.com                  time=206.40 ms
	node1.user1.aussieshift.com                  time=277.37 ms


	---- ping statistics ----
	2 replies max: 277.37 min: 206.40 avg: 241.89


**Lab 4 Complete!**

<!--BREAK-->