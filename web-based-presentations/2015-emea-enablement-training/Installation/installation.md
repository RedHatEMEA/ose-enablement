# Introduction

## Virtual Machines
Each user has 3 virtual machines with 4 GB of RAM, registered to the AMS2 Lab
Satellite, and subscribed to all the OpenShift channels.

The names of the vms per user are:

    OSE150209-st01-01 to OSE150209-st01-03 for student 1
    .
    .
    .
    OSE150209-st17-01 to OSE150209-st17-03 for student 17

## DNS Domain
The DNS Domain for the Virtual Machines is gps.hst.ams2.redhat.com
All Machines are registered to DNS. They names follow the convention

- OSE150209-st01-01.gps.hst.ams2.redhat.com **Broker**
- OSE150209-st01-02.gps.hst.ams2.redhat.com **Node 1**
- OSE150209-st01-03.gps.hst.ams2.redhat.com **Node 2**

## DHCP, DNS:
All VMs are registered in the DHCP server and will receive the same IP
in each boot. The DNS configuration is a work in progress (I don't have
writing permissions on the DNS server). In the meantime please use the
HOSTS configuration in the /etc/hosts of your system. See the
configuration attached for some copy+paste.


# Setup

1. Login to https://ipa.gps.hst.ams2.redhat.com/ or SSH to `ipa.gps.hst.ams2.redhat.com` to change the user password. (This is a mandatory step. To know why refer to http://www.freeipa.org/page/New_Passwords_Expired)
2. Go to https://rhev-manager.gps.hst.ams2.redhat.com/ovirt-engine/userportal/ and login with the login from 1. Use the domain gps.hst.ams2.redhat.com`. Note that the access is made through the user portal.

#Lab: Installing OpenShift Enterprise with oo-install


During this lab you will stand up your OpenShift Enterprise (OSE) environment.
Our environment will be modeled after one of the world's most popular public
PaaS offerings.  Some configuration choices have been made in order to save
money (AWS bill concerns), but we have stayed as true to a production
configuration as possible.  We will deploy the following:

* (1) Broker
* (2) OSE nodes

As you can see, this will consume 2 AWS EC2 instances. Later we will add another
node bringing the total to 3 EC2 instances.

#Puppet or openshift.sh?

Many OSE customers have been leveraging puppet to deploy a highly available OSE.
In this lab we will leverage the new `oo-install` command in OSE 2.2. OSE 2.2
was to be released to the general public this month (Nov, 2014).  `oo-install`
cannot make the assumption that the customer knows or wants puppet and so it
drives automation via the openshift.sh installation script.  `oo-install` will
interrogate the user with questions, generate a YAML file based on the
responses, and feed that file to openshift.sh as proper variables.  This offers
a cleaner and more reproducible installation.

This does not mean we are not also investing in puppet.  Shortly after the
release of OSE 2.2 we will also have completed the porting of the Origin puppet
modules to work with OSE 2.2.  Customers will eventually have a choice between
puppet or `oo-install`.

# **What Has Already Been Done to the Environment** #

**Packaging:**

As mentioned, this is OSE 2.2.  OSE 2.2 requires RHEL 6.6. Check out the yum
repository configuration on your boxes.


    [root@broker ~]# yum repolist
    Loaded plugins: amazon-id, priorities, rhui-lb, security
    45 packages excluded due to repository priority protections
    repo id                                          repo name                                                                             status
    jb-eap-6-for-rhel-6-server-rpms                  JBoss Enterprise Application Platform 6 (RHEL 6 Server) (RPMs)                          1,818+27
    jb-ews-2-for-rhel-6-server-rpms                  JBoss Enterprise Web Server 2 (RHEL 6 Server) (RPMs)                                      255+25
    rhel-6-server-ose-2.2-infra-rpms                 Red Hat OpenShift Enterprise 2.2 Infrastructure (RPMs)                                       150
    rhel-6-server-ose-2.2-jbosseap-rpms              Red Hat OpenShift Enterprise 2.2 JBoss EAP add-on (RPMs)                                       3
    rhel-6-server-ose-2.2-node-rpms                  Red Hat OpenShift Enterprise 2.2 Application Node (RPMs)                                     331
    rhel-6-server-ose-2.2-rhc-rpms                   Red Hat OpenShift Enterprise 2.2 Client Tools (RPMs)                                          14
    rhel-6-server-rpms                               Red Hat Enterprise Linux 6 Server (RPMs)                                              14,310+110
    rhel-server-rhscl-6-rpms                         Red Hat Software Collections RPMs for Red Hat Enterprise Linux 6 Server                    2,525
    rhui-REGION-client-config-server-6               Red Hat Update Infrastructure 2.0 Client Configuration Server 6                                3
    repolist: 19,409

Your EC2 image has already been registered and subscribed and had the correct
repos enabled by using `subscription-manager`.

`oo-install`  requires that the following packages additionally be installed
before you start:

    # yum -y install ruby unzip curl ruby193-ruby yum-plugin-priorities

**CPU/MEM/NET**

We will be using an AWS t2.medium instance.  That does not mean we recommend the
t2.medium for production use by customers.  m3.mediums or higher would be better
suited for production.  We have also already configured the AWS security groups
to allow for the required port access.

**Password-less Access**

`oo-install` will be executed from your broker node and SSH-es into the other
EC2 instances to install and configure OpenShift.  In order to allow that to
occur, `oo-install` requires that an SSH private/public key exchange be used in
place of a normal password challenge.  This has already been set up for you in
the images provided.

The `oo-install` program will default to using the root user.  You can run it as
any user.  Should you wish to use another user, you need to ensure that full
access has been granted via sudo and that sudo has been configured to be
password-less as well.  For the purposes of this class, please use the root user
when asked during the `oo-install`.  We have modified /etc/ssh/sshd_config to
allow for root login access.

# Download and Run oo-install #

Now that we have done our pre-work, we can actually issue the `oo-install`
command. The pre-install check list you should run through is:


* **Machine Roles:** Do I know what my boxes are and what role I'm going to ask
them to play (broker, msgserver, mongo, node)?
* **Password-less SSH:** Do I have a password-less SSH login to the instances
from the broker where I will be issuing the `oo-install` command?
* **Power User:** Do I know what username I will be giving to `oo-install` to
execute the commands on the instances? It needs to have full command access
through sudo or be root. For the classroom environment, just use root.
* **Yum Channels:** Does `yum repolist` show the correct repo setup? Do I have
the five prereq packages installed?
* **IP Addresses:** Have I decided on a virtual hostname for the broker? Do I
know which IP address I plan to give the broker?

`oo-install` is not shipped with the product, but instead maintained out of
cycle and distributed on OpenShift.com.  We need to download and execute the
`oo-install` script.

It is recommended you use the `screen` command (or another terminal emulator if
you prefer) so that you can safely disconnect from your SSH session without
disrupting the install. `screen` has been preinstalled on the lab machines.
Start a session with the command `screen`:

    # screen

**Note:** To detach from a `screen` session, press 'Ctrl' and 'a', and then hit
the 'd' key. To reattach to a `screen` session, use the command `screen -r`.

In your screen session on the **broker** host, issue the following command:

    # sh <(curl -s https://install.openshift.com/ose-2.2)

The curl command will download the installer and create a `.openshift` directory
in the home directory of the user who issued the command.  It will then execute
the installation program.  The first question that is asked is as follows:

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


We are selecting **1** so we can install OpenShift Enterprise for the first time.
As you can see, `oo-install` can be used to add additional nodes to the
environment as well.

The next thing we need to do is install a DNS server for usage during this
class.  For the purposes of this class, we are going to be using BIND, which we
will install on the broker host provided to you by the instructor.  The
`oo-install` program will display the following information:

It looks like you are running oo-install for the first time on a new
system. The installer will guide you through the process of defining
your OpenShift deployment.

    ----------------------------------------------------------------------
    DNS Configuration
    ----------------------------------------------------------------------

    First off, we will configure some DNS information for this system.

    Do you want me to install a new DNS server for OpenShift-hosted
    applications, or do you want this system to use an existing DNS
    server? (Answer 'yes' to have me install a DNS server.) (y/n/q/?)

Answer **y** to have BIND installed on the broker host.

The next thing we need to do is set the hostname of the environment that we will
be using.  For this class we will be using userX.onopenshift.com, where X is the
user number provided to you. Please substitute your user number wherever you see
**userX**.

    All of your hosted applications will have a DNS name of the form:

    <app_name>-<owner_namespace>.<all_applications_domain>

    What domain name should be used for all of the hosted apps in your
    OpenShift system? |example.com|

Type in your **userX.onopenshift.com** domain and press the enter key.

    Do you want to register DNS entries for your OpenShift hosts with the
    same OpenShift DNS service that will be managing DNS records for the
    hosted applications? (y/n/q)

    Answer **y** as we do want to add DNS records for hosted applications.

    What domain do you want to use for the OpenShift hosts?
    |hosts.userX.onopenshift.com|

    Select the default domain of hosts.userX.onopenshift.com by pressing the enter key.

We then need to specify the hostname for the host we are describing:

    You have indicated that you want the installer to deploy DNS. Please configure a host to use as the nameserver.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing):

Enter in **broker.hosts.userX.onopenshift.com** and press the enter key. Remember
to replace X with your user number.

    Hostname / IP address for SSH access to broker.hosts.user1.onopenshift.com from
    the host where you are running oo-install. You can say 'localhost' if
    you are running oo-install from the system that you are describing:
    |broker.hosts.user1.onopenshift.com|

Input the 54.x.x.x address provided to you by the instructor.

Once you enter in the IP address of the host, you will then need to provide the
username that will be used to install the product.  For this class, you can use
the default of **root**:

    Username for SSH access to 54.65.64.139: |root|

Use the default of **root**.

You should see the following output indicating that `oo-install` was able to authenticate with the provided username using a password-less login via SSH keys.  You will then need to specify the IP address of the host:

    Validating root@54.65.64.139... looks good.

    Detected IP address 172.31.29.241 at interface eth0 for this host.
    Do you want Nodes to use this IP information to reach this host?
    (y/n/q/?)

Answer **n** and then press the enter key.

    Specify the IP address that other hosts will use to connect to this
    host (Detected 172.31.29.241): |54.65.64.139|

Once you enter in **n**, you will be asked to provide the IP address that you
would like to use.  Use the 54.x.x.x one provided by pressing the enter key.

You will then be asked to specify the network interface that should be used to
send packets across the network:

    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

Use the default of **eth0** by pressing the enter key.

You will then be presented with the following information:

    Normally, the BIND DNS server that is installed on this host will be
    reachable from other OpenShift components using the host's configured
    IP address (54.65.64.139).

    If that will work in your deployment, press <enter> to accept the
    default value. Otherwise, provide an alternate IP address that will
    enable other OpenShift components to reach the BIND DNS service on
    this host: |54.65.64.139|

Use the default 54.x.x.x IP address by pressing the enter key.

# Broker Configuration #

Now that we have our DNS server installed on our host, we can also assign the
**broker** role to the host so that the host will act as both the DNS server and
the OpenShift Enterprise Broker.

    ----------------------------------------------------------------------
    Broker Configuration
    ----------------------------------------------------------------------

    Okay. I'm going to need you to tell me about the host where you want
    to install the Broker.

    Do you want to assign the Broker role to broker.hosts.userX.onopenshift.com?
    (y/n/q/?)

Enter in **y** and press the enter key.

You will then be asked if you want to configure an additional broker:

    Do you want to configure an additional Broker? (y/n/q)

Enter in **n** and press enter.

# Node Configuration #

At this point in the installation we have configured a DNS server, MongoDB
server, and a Broker host.  We now need to configure a host for a node that our
applications will be deployed on.

    ----------------------------------------------------------------------
    Node Configuration
    ----------------------------------------------------------------------

    Okay. I'm going to need you to tell me about the host where you want
    to install the Node.

    Do you want to assign the Node role to broker.hosts.userX.onopenshift.com?
    (y/n/q/?)

OpenShift allows you to assign the **node** role to the same machine as the
broker.  Doing this would mean that your applications will be deployed on the
same host as the broker.  This is a bad architecture and should never be used in
a production environment.  However, it may be suitable for a proof of concept
where only a single machine is available.

For this class, we are going to install a multi-node configuration.  For that
reason, answer **n** and press the enter key.

Once you say not to add the node role to the broker host, you will be asked to
provide information about the host that you will be using for your node:

    Okay, please provide information about this Node host.

    Hostname (the FQDN that other OpenShift hosts will use to connect to
    the host that you are describing):

Enter in **node1.hosts.userX.onopenshift.com** and press the enter key. Remember
to replace X with your user number.

You will then be asked to provide the IP address of the node1 host.

    Hostname / IP address for SSH access to node1.hosts.userX.onopenshift.com from
    the host where you are running oo-install. You can say 'localhost' if
    you are running oo-install from the system that you are describing:
    |node1.hosts.userX.onopenshift.com|

Enter in the **54.x.x.x** IP address of your node1 host and press the enter key.

    Username for SSH access to 54.65.70.239: |root|

Use the default username that is provided (**root**) and press the enter key.

    Validating root@54.65.70.239... looks good.

    Detected IP address 172.31.29.240 at interface eth0 for this host.
    Do you want to use this as the public IP information for this Node?
    (y/n/q/?)

We then need to provide the IP address of the node host.  By default, the
installation program will ask you to use the 172.x.x.x address.  This is **not**
what we want to use.  Enter in **n** and press the enter key.

    Specify the public IP address for this Node (Detected 172.31.29.240):
    |54.65.70.239|

Use the provided public 54.x.x.x IP address and press the enter key.

You will then be asked to specify the network interface that should be used to
send packets across the network:

    Specify the network interface that other hosts will use to connect to
    this host (Detected 'eth0'): |eth0|

Use the default of **eth0** by pressing the enter key.

You will then see the following output:

    That's everything we need to know right now for this Node.

    Currently you have described the following host system(s):
    * broker.hosts.user1.onopenshift.com (Broker, NameServer)
    * node1.hosts.user1.onopenshift.com (Node)

    Do you want to configure an additional Node? (y/n/q)

As we will demonstrate adding a node later, we can simply select **n** for now,
and press enter.

You will then be asked if you want to manually provide username/password
combinations for the required services such as ActiveMQ etc.

    Do you want to manually specify usernames and passwords for the
    various supporting service accounts? Answer 'N' to have the values
    generated for you (y/n/q)

Enter in **n** and press the enter key.

You will then see an overview of the installation:

    Here are the details of your current deployment.

    Note: ActiveMQ and MongoDB will be installed on all Broker instances.
    For more flexibility, rerun the installer in advanced mode (-a).

    DNS Settings
    * Installer will deploy DNS
    * Application Domain: user1.onopenshift.com
    * Register OpenShift hosts with DNS? Yes
    * Component Domain: hosts.user1.onopenshift.com

    Global Gear Settings
    +-------------------------+-------+
    | Valid Gear Sizes        | small |
    | User Default Gear Sizes | small |
    | Default Gear Size       | small |
    +-------------------------+-------+

    Account Settings
    +----------------------------+------------------------------------------------------------------+
    | OpenShift Console User     | demo                                                             |
    | OpenShift Console Password | j8AFZ4yoQrulFhQdILD6Xg                                           |
    | Broker Session Secret      | advO54MMmtw6r0oehHyRQ                                            |
    | Console Session Secret     | g1lSvmvaBop8JRdnYlCdOQ                                           |
    | Broker Auth Salt           | a0YA15nc1E53IfoLLwl1Kw                                           |
    | Broker Auth Private Key    | -----BEGIN RSA PRIVATE KEY-----                                  |
    |                            | MIIEowIBAAKCAQEA9CLW0CxxY824ALMY9jfq+aPGI2QMg5s7xTt11SyEb+XpUnjX |
    |                            | 42Kt+W04ghtQk/8Fmu/LreCQlEgS/hKEP0O0C8LvDkaZ0E2gUUjSKdkGZNiRsvq6 |
    |                            | PbXadZ0BSsnOEe8uA3J6AuPYI4KZw4B6oEdysKx2M1plYm5xdJczFPkUNs8rLTEB |
    |                            | IZMgFErzaywvju2cOrI32t/SySZstfXYHwCi8IdwM6bAcwoiQ9vCQz8kaIWdyenc |
    |                            | UGRaCMLdAph7LLyTUB13qO1jsgxcYPYQdmxDdCbxm5eLm0IWZkTZCH20CXMHF+VK |
    |                            | /M+ElHefccQgMBura6HC1pUOoEhpk55uNvT0TQIDAQABAoIBAQC1lFQBcYzElnWM |
    |                            | z6h5OQ3jrxPnrrpACG1kPN1fOEUolO/9DzRDQ1nycnHdE0PTT5JzsnbjVGs0XocB |
    |                            | wfPqughn1wzGqWwtqg7bZjYqOeiviQSVAjcTPvbFE4mqfn5uiF7I4ZQuIhjYEIMd |
    |                            | DaonG/0JurwPZeSSWWK5PNwZdUi7mdafERDaxvn43eDsOUUTDMnKoOTSNGCd+yWB |
    |                            | rxkHJfXP/TxfFPNMDKkYJTjEX3LHUYd1vSHe+Ak97vSBCMIbkY24XGbFjdudR0yu |
    |                            | Zb/qoxy9inzDOK4eQqzn8n+7O9McgjX0nauQlh2s/LsTToqNBLSdHIuHUiONLphX |
    |                            | PVbnWf4BAoGBAP/mzq1PmFkudTBs99+iLQLaYnQVu+sL9vwcQnhjs8wwjOxTsrG/ |
    |                            | uGTGzfrkdYxhtpN1D3JeTDZgscZyvl8my6Tz8+DVJLhbEr5tNEUMOefjffHWQg/U |
    |                            | dzulCCgQ4Mx3JC3/Cxle0TmRUGiSolCKW6yXRmbEaT99ey3N/7yYokg9AoGBAPQ6 |
    |                            | 354q19ZRerRx0by42n9JprKRyoYKbpIadCsjNamNVX9mBIPVj+VgvohsOzVBOsY6 |
    |                            | cYDdVCLebwWWdnlxWviky8X1VHLQAZbynB2YUFFk15UFjAMypE9KtvU2UsGLucOs |
    |                            | 5Ujx46oTsdKqJGKkCABVIMlqtbsOXvS8OHrjxQ1RAoGABJ+G3Fqzxeiw9U8Cq2ei |
    |                            | qIqJfM9ntbdhnuxjxwkGFopKAXsBn3R3QFrXHdFCzmZ1hfR3cvmBJvpYO92W0uFA |
    |                            | jJpbrZQsNahvjkEq0JSH90iE3fmg9+g+vzUcEJ09cnQ0kyAocyzjWsblTP5ZMFtP |
    |                            | jK6u9uxVenAp6YnvNNkNFYECgYAigDaat16qLfRxjSqdyFdFZ/gefa3oZYzdItOK |
    |                            | TH0GKKsNRjIZFZAwTQxdZTyv9zkAS71BAQMjsdxpI6o02aiKO21114RIe83drwQS |
    |                            | wjOGbAJwUMpIoVzIvrs9xKDIKp7hX4k8Vr9chU+3fMWLEbT3pw7spSBq/kq3s+ce |
    |                            | pRJvIQKBgFEVxC98gDAIQn2Tmsoqu5BMleGqnN2U1RH9DCZ1q8+sK6M8LBzyhS5V |
    |                            | 5ydPyNnPYsBCt0luvTEyLqPjdCbUdff23/wl3ItD9oCcEdpcb4OWHU+phwf9wO36 |
    |                            | PY4z4aBQehzVv96VnFtvRFrSrBE6C+/gFTCASvwiDsBvvcGm4r3H             |
    |                            | -----END RSA PRIVATE KEY-----                                    |
    | MCollective User           | mcollective                                                      |
    | MCollective Password       | 5agqqDiRInHpkulFta5C3w                                           |
    | MongoDB Admin User         | admin                                                            |
    | MongoDB Admin Password     | z6sys4iOzddWMe0GXesmQ                                            |
    | MongoDB Broker User        | openshift                                                        |
    | MongoDB Broker Password    | 7JmWlau3VzY3eDlotxLvA                                            |
    +----------------------------+------------------------------------------------------------------+

    Node Districts
    +----------+-----------+---------------------------------------------------------------------+
    | District | Gear Size | Nodes                                                               |
    +----------+-----------+---------------------------------------------------------------------+
    | Default  | small     | node1.hosts.user1.onopenshift.com                                   |
    +----------+-----------+---------------------------------------------------------------------+

    Role Assignments
    +------------+------------------------------------+
    | Broker     | broker.hosts.user1.onopenshift.com |
    | NameServer | broker.hosts.user1.onopenshift.com |
    | Nodes      | node1.hosts.user1.onopenshift.com  |
    +------------+------------------------------------+

    Host Information
    +------------------------------------+------------+
    | Hostname                           | Roles      |
    +------------------------------------+------------+
    | broker.hosts.user1.onopenshift.com | Broker     |
    |                                    | NameServer |
    | node1.hosts.user1.onopenshift.com  | Node       |
    +------------------------------------+------------+

    Choose an action:
    1. Change the deployment configuration
    2. View the full host configuration details
    3. Proceed with deployment
    Type a selection and press <return>:

**Note:** You may wish to copy the OpenShift Console Password from the output
and save it somewhere, as we will be using it in later labs.

If everything looks correct, select **3** to proceed with the deployment.

`oo-install` also gives the user the ability to specify additional Red Hat
Subscription information.  This has been preconfigured for you in this class.

    Do you want to make any changes to the subscription info in the
    configuration file? (y/n/q/?)

Enter in **n** and press the enter key.

    Do you want to set any temporary subscription settings for this
    installation only? (y/n/q/?)

Enter in **n** and press the enter key.

During the installation you may see several errors that look like:

    Address 54.175.14.24 maps to ec2-54-175-14-24.compute-1.amazonaws.com, but
    this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!

This is because the systems have their hostnames configured to match their
public DNS names but Amazon owns the reverse DNS for the public IPs and it does
not match our hostnames. It's OK - don't worry!

At the end of this process, OpenShift Enterprise 2.2 will be installed on your
broker and your single node.

# Troubleshooting

## Distributing SSH Keys

Distributing SSH-Keys worked for me with: (Change XX with your user-number and maybe leave XX-03 node for now)

    for host in OSE150209-stXX-02 OSE150209-stXX-03; do ssh-copy-id -i ~/.ssh/id_rsa.pub "root@$host"; done

## Later Node Installation

If you install one of the nodes at a later time, you need to register its DNS Name manually using the following command:

    oo-register-dns -h node -d hosts.domain.com -n [ip_address]

## Strange oo-accept-node errors

If you see the following output after the installation:

    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/diy/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/ruby/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/mongodb/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/nodejs/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/mysql/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/cron/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/postgresql/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/haproxy/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/perl/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/python/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/jbossews/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/jenkins-client/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/jenkins/metadata/manifest.yml
    FAIL: cart repo version mismatch for /usr/libexec/openshift/cartridges/php/metadata/manifest.yml

run the following commands on your nodes:

    node# yum -y reinstall $(rpm -qa | grep cartridge)

after that is done run:

    node# oo-admin-cartridge -a install -R -s /usr/libexec/openshift/cartridges -f -d

Then the error should be gone. The problem is that the systems were running ahead of time when we installed them.
Now their time is configured with ntpd



