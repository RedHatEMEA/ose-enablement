# Lab: Managing Applications

###Server used:

* localhost
* node host

###Tools used:

* `rhc`

##Start/Stop/Restart OpenShift Enterprise applications

OpenShift Enterprise provides commands to start, stop, and restart an
application. If at any point in the future you decide that an application should
be stopped for some maintenance, you can stop the application using the *rhc app
stop* command. After making necessary maintenance tasks, you can start the
application again using the `rhc app start` command.

To stop an application, execute the following command:

	$ rhc app stop firstphp
	
	RESULT:
	firstphp stopped

Verify that your application has been stopped by going to the URL:
http://firstphp-ose.userX.onopenshift.com. You should get a 503 response.
	
	<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
	<html><head>
	<title>503 Service Temporarily Unavailable</title>
	</head><body>
	<h1>Service Temporarily Unavailable</h1>
	<p>The server is temporarily unable to service your
	request due to maintenance downtime or capacity
	problems. Please try again later.</p>
	<hr>
	<address>Apache/2.2.15 (Red Hat) Server at firstphp-ose.userX.onopenshift.com Port 80</address>
	</body></html>

To start the application back up, execute the following command:

	$ rhc app start firstphp

	RESULT:
	firstphp started

Verify that your application has been started by refreshing the URL.

You can also stop and start the application in one command as shown below.

	$ rhc app restart firstphp

	RESULT:
	firstphp restarted


##**Viewing application details**

All of the details about an application can be viewed by the `rhc app show`
command. This command will list when the application was created, the unique
identifier of the application, Git URL, SSH URL, and other details as shown
below:

	$ rhc app show firstphp

	firstphp @ http://firstphp-ose.userX.onopenshift.com/ (uuid: 53727d3e201f1b7288000042)
	-------------------------------------------------------------------------------------------
	  Domain:     ose
	  Created:    1:14 PM
	  Gears:      1 (defaults to small)
	  Git URL:    ssh://53727d3e201f1b7288000042@firstphp-ose.userX.onopenshift.com/~/git/firstphp.git/
	  SSH:        53727d3e201f1b7288000042@firstphp-ose.userX.onopenshift.com
	  Deployment: auto (on git push)

	  php-5.4 (PHP 5.4)
	  -----------------
	    Gears: 1 small


##**Viewing application status**

The state of application gears can be viewed by passing the `state` switch to
the `rhc app show` command, as shown below:

	$ rhc app show --state firstphp
	Cartridge php-5.4 is started


##**Cleaning up an application**

As a user starts developing an application and deploying changes to OpenShift
Enterprise, the application will start consuming some of the available disk
space that is part of their quota. This space is consumed by the Git repository,
log files, temporary files, and unused application libraries. OpenShift
Enterprise provides a disk-space cleanup tool, `tidy` to help users manage the
application disk space. This command is also available under `rhc app` and
performs the following functions:

* Runs the `git gc` command on the application's remote Git repository.
* Clears the application's `/tmp/` and log file directories. These are specified
  by the application's `OPENSHIFT_LOG_DIR` and `OPENSHIFT_TMP_DIR` environment
  variables.
* Clears unused application libraries. This means that any library files
  previously installed by a `git push` command are removed.

To clean up the disk space on your application gear, run the following command:

	$ rhc app tidy firstphp

After running this command you should see the following output:

	RESULT:
	firstphp cleaned up

##**SSH to application gear**

OpenShift allows remote access to the application gear by using the Secure Shell
protocol (SSH). [Secure Shell (SSH)](http://en.wikipedia.org/wiki/Secure_Shell)
is a network protocol for securely getting access to a remote computer.  SSH
uses RSA public key cryptography for both the connection and authentication. SSH
provides direct access to the command line of your application gear on the
remote server. After you are logged in on the remote server, you can use the
command line to directly manage the server, check logs, and test quick changes.
OpenShift Enterprise uses SSH for the following:

* Performing Git operations.
* Remote access your application gear.

The SSH keys were generated and uploaded to OpenShift Enterprise by `rhc setup`
command we executed in a previous lab. You can verify that SSH keys are uploaded
by logging into the OpenShift Enterprise web console and clicking on the
"Settings" tab, as shown below.

![](http://training.runcloudrun.com/ose2/sshKeys-webconsole.png)

**Note:** If you don't see an entry under "Public Keys," then you can either
upload the SSH key by clicking on "Add a new key" or run the `rhc setup` command
again. This will create a SSH key pair in `<User.Home>/.ssh/` folder and upload
the public key to the OpenShift Enterprise PaaS.

After the SSH keys are uploaded, you can SSH into the application gear as shown
below.  SSH is installed by default on most UNIX-like platforms, such as Mac OS
X and Linux.  For Windows, you can use
[PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/).  Instructions for
installing PuTTY can be found [on the OpenShift
website](https://openshift.redhat.com/community/page/install-and-setup-putty-ssh-client-for-windows).

Although you can SSH in by using the standard `ssh` command line utility, the
OpenShift client tools includes a SSH command that makes the process of logging
in to your application even easier.  To SSH to your gear, execute the following
command:

	$ rhc app ssh firstphp

**Note:** The `rhc` client tools are context aware.  If you are in the directory
of your application, you do not need to specify the application's name to `rhc`.

If you want to use the `ssh` command line utility, execute the following
command, remembering to replace X with your user number:


	$ ssh UUID@appname-namespace.userX.onopenshift.com

You can get the SSH URL by running `rhc app show` command as shown below:

	$ rhc app show firstphp

	firstphp @ http://firstphp-ose.userX.onopenshift.com/ (uuid: 53727d3e201f1b7288000042)
	-------------------------------------------------------------------------------------------
	  Domain:     ose
	  Created:    1:14 PM
	  Gears:      1 (defaults to small)
	  Git URL:    ssh://53727d3e201f1b7288000042@firstphp-ose.userX.onopenshift.com/~/git/firstphp.git/
	  SSH:        53727d3e201f1b7288000042@firstphp-ose.userX.onopenshift.com
	  Deployment: auto (on git push)

	  php-5.4 (PHP 5.4)
	  -----------------
	    Gears: 1 small

Now you can `ssh` into the application gear using the SSH URL shown above:

	$ ssh 53727d3e201f1b7288000042@firstphp-ose.userX.onopenshift.com
	
	    *********************************************************************
	
	    You are accessing a service that is for use only by authorized users.  
	    If you do not have authorization, discontinue use at once. 
	    Any use of the services is subject to the applicable terms of the 
	    agreement which can be found at: 
	    https://openshift.redhat.com/app/legal
	
	    *********************************************************************
	
	    Welcome to OpenShift shell
	
	    This shell will assist you in managing OpenShift applications.
	
	    !!! IMPORTANT !!! IMPORTANT !!! IMPORTANT !!!
	    Shell access is quite powerful and it is possible for you to
	    accidentally damage your application.  Proceed with care!
	    If worse comes to worst, destroy your application with 'rhc app destroy'
	    and recreate it
	    !!! IMPORTANT !!! IMPORTANT !!! IMPORTANT !!!
	
	    Type "help" for more info.
	


You can also view all of the commands available on the application gear shell by
running the help command as shown below:

	[firstphp-ose.userX.onopenshift.com 5488ba3e7990081164000007]\> help
	Help menu: The following commands are available to help control your openshift application and environment.

	gear            control your application (start, stop, restart, etc)
	                or deps with --cart      (gear start --cart mysql-5.1)
	tail_all        tail all log files
	export          list available environment variables
	rm              remove files / directories
	ls              list files / directories
	ps              list running applications
	kill            kill running applications
	mysql           interactive MySQL shell
	mongo           interactive MongoDB shell
	psql            interactive PostgreSQL shell
	quota           list disk usage

	Deprecated:
	ctl_app         control your application (start, stop, restart, etc)
	ctl_all         control application and deps like mysql in one command
		
##**Viewing disk quota for an application**

When we used `oo-install`, OpenShift automatically configured the application
gears to have a disk-usage quota.  You can view the quota of your currently
running gear by connecting to the gear node host via SSH as discussed previously
in this lab.  Once you are connected to your application gear, enter the
following command:

	[firstphp-ose.userX.onopenshift.com 5488ba3e7990081164000007]\>  quota -s

If the quota information that we configured earlier is correct, you should see
something like the following:

	Disk quotas for user e9e92282a16b49e7b78d69822ac53e1d (uid 1000): 
	     Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
	/dev/mapper/VolGroup-lv_root
	                  22540       0   1024M             338       0   40000        

To view how much disk space your gear is actually using, you can also enter in
the following:

	[firstphp-ose.userX.onopenshift.com 5488ba3e7990081164000007]\> du -h

Go ahead and exit from your SSH session with `exit`.

##**Viewing log files for an application**

Logs are very important when you want to find out why an error is happening or
if you want to check the health of your application. OpenShift Enterprise
provides the `rhc tail` command to display the contents of your log files. To
view all the options available for the `rhc tail` command, issue the following:

    $ rhc tail --help

And you should see:

	Usage: rhc tail <application>

	Tail the logs of an application


	Options
	  -n, --namespace NAME      Name of a domain
	  -o, --opts options        Options to pass to the server-side (linux based) tail command (applicable to tail command only) (-f is implicit.  See the linux tail man page full list of options.) (Ex: --opts '-n
	                            100')
	  -f, --files files         File glob relative to app (default <application_name>/logs/*) (optional)
	  -g, --gear ID             Tail only a specific gear
	  -a, --app NAME            Name of an application

	Global Options
	  -l, --rhlogin LOGIN       OpenShift login
	  -p, --password PASSWORD   OpenShift password
	  --token TOKEN             An authorization token for accessing your account.
	  --server NAME             An OpenShift server hostname (default: openshift.redhat.com)
	  --timeout SECONDS         The timeout for operations

	  See 'rhc help options' for a full list of global options.


The `rhc tail` command requires that you provide the application name of the
logs you would like to view.  To view the log files of our **firstphp**
application, use the following command:

	$ rhc tail firstphp

You should see information for both the access and error logs.  While you have
the `rhc tail` command open, issue a HTTP `GET` request by pointing your web
browser to **http://firstphp-ose.userX.onopenshift.com**.  You should see a new
entry in the log files that looks similar to this:

	10.10.56.204 - - [22/Jan/2013:18:39:27 -0500] "GET / HTTP/1.1" 200 5242 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:19.0) Gecko/20100101 Firefox/19.0"

The log files are also available if you are SSHed into your gear in the
`php-5.4/logs/` directory.

Now that you know how to view log files by using the `rhc tail` command, it is
also important to know how to view logs during an SSH session on the gear.  SSH
into the gear and use the knowledge you have learned in the lab to tail the logs
files on the gear.

##**Adding a custom domain to an application using the command line**

OpenShift Enterprise supports the use of custom domain names for an application.
For example, suppose we want to use `http://www.somesupercooldomain.com` for the
application **firstphp** that we created in a previous lab. The first thing you
need to do before setting up a custom domain name is to buy the domain name from
domain registration provider.

After buying the domain name, you have to add a [CNAME
record](http://en.wikipedia.org/wiki/CNAME_record) for the custom domain name.
Once you have created the `CNAME` record, you can let OpenShift Enterprise know
about the `CNAME` by using the `rhc alias` command.

	$ rhc alias add firstphp www.mycustomdomainname.com

Technically, what OpenShift Enterprise has done under the covers is set up a
`vhost` in Apache to handle the custom URL.

##**Adding a custom domain to an application using the web console**

The OpenShift web console now allows you to customize your application's domain
host URL without having to use the command line tools.  Point your browser to
**broker.hosts.userX.onopenshift.com** and authenticate.  After you have
authenticated to the management console, click on the **Applications** tab at the
top of the screen.

You should see the **firstphp** application listed:

![](http://training.runcloudrun.com/ose2/domainName.png)

Click on the **change** link next to your application name.  On the following
page, you can specify the custom domain name for your application.  Go ahead and
add a custom domain name of **www.openshiftrocks.com** and click the save button.

![](http://training.runcloudrun.com/ose2/customDomainName.png)

You should now see the new domain name listed in the console.  You can also
verify the alias was added by running the following command at your terminal
prompt:

	$ rhc app show firstphp

If the alias was added correctly, you should see the following output:

	firstphp @ http://firstphp-ose.userX.onopenshift.com/ (uuid: 53727d3e201f1b7288000042)
	-------------------------------------------------------------------------------------------
	  Domain:     ose
	  Created:    1:14 PM
	  Gears:      1 (defaults to small)
	  Git URL:    ssh://53727d3e201f1b7288000042@firstphp-ose.userX.onopenshift.com/~/git/firstphp.git/
	  SSH:        53727d3e201f1b7288000042@firstphp-ose.userX.onopenshift.com
	  Deployment: auto (on git push)
	  Aliases:    www.mycustomdomainname.com

	  php-5.4 (PHP 5.4)
	  -----------------
	    Gears: 1 small


If you point your web browser to **www.openshiftrocks.com**, you will notice that
it does not work.  This is because the domain name has no DNS entry pointing to
your application.  You can verify this with the `dig` command:

    ; <<>> DiG 9.9.4-P2-RedHat-9.9.4-16.P2.fc20 <<>> www.openshiftrocks.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 10118
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
    
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 4096
    ;; QUESTION SECTION:
    ;www.openshiftrocks.com.		IN	A
    
    ;; AUTHORITY SECTION:
    com.			854	IN	SOA	a.gtld-servers.net. nstld.verisign-grs.com. 1418308070 1800 900 604800 86400
    
    ;; Query time: 33 msec
    ;; SERVER: 10.11.5.19#53(10.11.5.19)
    ;; WHEN: Thu Dec 11 09:28:55 EST 2014
    ;; MSG SIZE  rcvd: 124

The `NXDOMAIN` status means that the domain is **N**one**x**istant - it's not
registered.

In order to verify that OpenShift added the `vhost`, add an entry in your
`/etc/hosts` file on your local machine.

	$ sudo vi /etc/hosts

Add the following entry, replacing the IP address with the address of your node host.

	54.x.x.x  www.openshiftrocks.com

Once you have edited and saved the file, open your browser and go to your custom
domain name.  You should see the application.

Once you have verified that the `vhost` was added correctly by viewing the site
in your web browser, delete the line from the `/etc/hosts` file.

##**Backing up an application**

Use the `rhc snapshot save` command to create backups of your OpenShift
Enterprise application. This command creates a gzipped tar file of your
application and of any locally-created log and data files.  This snapshot is
downloaded to your local machine.  The directory structure that exists on the
server is maintained in the downloaded archive.

	$ rhc snapshot save firstphp
	Pulling down a snapshot to firstphp.tar.gz...
	Waiting for stop to finish
	Done
	Creating and sending tar.gz
	Done
	
After the command successfully finishes, you will see a file named
`firstphp.tar.gz` in the directory where you executed the command. The default
filename for the snapshot is **$Application_Name.tar.gz**. You can override this
path and filename with the `-f` or `--filepath` option.

**NOTE**: This command will stop your application for the duration of the backup
process.

Now that we have our application snapshot saved, edit the `index.php` file in
your `firstphp` application and change the **"Welcome to OpenShift Enterprise"**
`h1` tag to say **"Welcome to OpenShift Enterprise before restore"**.

Once you have made this change, perform the following command to push your
changes to your application gear:

	$ git commit -am "Added message"
	$ git push

Verify that changes are reflected in your web browser.

##**Restoring a backup**

Not only you can take a backup of an application, but you can also restore a
previously saved snapshot.  This form of the `rhc` command restores the Git
repository, as well as the application data directories and the log files found
in the specified archive. When the restoration is complete, OpenShift Enterprise
runs the deployment script on the newly restored repository.  

**Note:** Restoring a backup, while still valid, is no longer the recommended
way to restore a previous version of an application.  The correct way to restore
a previous deployment is to use the new rollback capabilities. We will be
discussing rollbacks later during this training class.

Given that, you may skip the following restore.

**OPTIONAL**

To restore an application snapshot, run the following command:

	$ rhc snapshot restore firstphp -f firstphp.tar.gz

You will see the following confirmation message:

	Restoring from snapshot firstphp.tar.gz...
	Removing old data dir: ~/app-root/data/*
	Restoring ~/app-root/data

**NOTE**: This command will stop your application for the duration of the restore process.

##**Verify application has been restored**

Open up a web browser and point to the following URL:

	http://firstphp-ose.userX.onopenshift.com

If the restore process worked correctly, you should see the restored application
running just as it was before.

##**Deleting an application**

You can delete an OpenShift Enterprise application by executing the *rhc app
delete* command. This command deletes your application and all of its data on
the OpenShift Enterprise server but leaves your local directory with the Git
clone of the application intact. This operation cannot be undone, so use it with
caution.

	$ rhc app delete someAppToDelete
	
	Are you sure you wish to delete the 'someAppToDelete' application? (yes/no)
	yes 
	
	Deleting application 'someAppToDelete'
	
	RESULT:
	Application 'someAppToDelete' successfully deleted

There is another variant of this command which does not require the user to
confirm the delete operation.  To use this variant, pass the `--confirm` flag:

	$ rhc app delete --confirm someAppToDelete
	
	Deleting application 'someAppToDelete'
	
	RESULT:
	Application 'someAppToDelete' successfully deleted


#**Deployment History and Rollbacks**

OpenShift supports deployment history complete with the ability to rollback to a
previous version of your application. Why is this important? There are often
times when new changes are deployed to production only to find that it
introduces a new critical bug that must be fixed immediately. The most obvious
way to handle this is to rollback the application code to the last known working
state while developers work to resolve the defect in the software. There are
various methods to solve this problem including versioning RPMs and keeping them
in a repository to backing up the gold build of the `.war` or `.ear` file in
Javaland. During this lab, we will learn how to the new deployment and rollback
strategy that has been implemented as part of OpenShift Enterprise can be
utilized.

##**Create a new application using the RHC command line tools**

In order for the exercises in this lab to work correctly, we need to create a
new application called `prodapp` using the `php-5.4` cartridge:

	$ rhc app create prodapp php-5.4

##**Specifying the number of deployments to keep**

By default, OpenShift Enterprise will only track the last deployment of your
application. In order to be able to rollback to a previous version of your
application code, we need to configure the demo application we created in the
previous step to track multiple deployments of our code base. To accomplish
this, change to your application directory and modify the configuration with the
following commands:

	$ cd prodapp
	$ rhc app-configure --keep-deployments 10

The above command will alert OpenShift that we want to track the last 10
deployments of our application.

**Note:** If you get an error message saying `invalid syntax`, this is because
you are running an older version of the client tools on your machine. The ~rhc~
gem that contains the deployment functionality was released on November 6, 2013.
If your `rhc` gem install is older than that, simply run a `gem update rhc` to
get the latest version.

##**Listing the deployments**

To view a list of your application deployment history, issue the following
command:

	$ rhc deployment-list

If the above command is executed correctly, you should see one deployment in
your history.

	3:58 PM, deployment 9d8f96ac

The information you see will look slightly different for you, specifically the
time of the deployment and the deployment hash. The reason you have a deployment
already in your history is that, when you created the application previously in
this lab, a template application was deployed to your OpenShift Enterprise gear.
If you view your application in a browser at this point, you will see this
deployment, which includes the "getting started" page at your domain.

##**Creating a new deployment**

In order to see deployments in actions, let's create a new file and deploy it.
Ensure that you are in your application's root directory and enter in the
following commands:

	$ echo "Hello deployment 1" > hello.html
	$ git add .
	$ git commit -am "Creating hello text file";
	$ git push

At this point, you will notice a few new bits of information provided to you
when you performed the `git push`. Namely, the platform will display the
deployment ID as well activating the deployment. The output should look similar
to this:

	Counting objects: 4, done.
	Compressing objects: 100% (2/2), done.
	Writing objects: 100% (3/3), 351 bytes, done.
	Total 3 (delta 0), reused 0 (delta 0)
	remote: Stopping PHP 5.4 cartridge (Apache+mod_php)
	remote: Waiting for stop to finish
	remote: Waiting for stop to finish
	remote: Building git ref 'master', commit 318869b
	remote: Checking .openshift/pear.txt for PEAR dependency...
	remote: Preparing build for deployment
	remote: Deployment id is cc200f43
	remote: Activating deployment
	remote: Starting PHP 5.4 cartridge (Apache+mod_php)
	remote: Application directory "/" selected as DocumentRoot
	remote: -------------------------
	remote: Git Post-Receive Result: success
	remote: Activation status: success
	remote: Deployment completed with status: success
	To ssh://5372a37e201f1bf35f0000de@prodapp-ose.user1.onopenshift.com/~/git/prodapp.git/
	   c1fb528..318869b  master -> master

Verify that your application code was pushed to the production gear by visiting
the URL for your application. Open your browser and point the location to to
following URL. Remember to replace X with your user number.

    http://prodapp-ose.userX.onopenshift.com/hello.html

If the deployment was successful, you should see the following displayed in your
web browser:

	Hello deployment 1

Let's create one more deployment so that we can demonstrate how to properly roll
back. Ensure that you are still in the root directory of your application and
issue the following commands:

	$ echo "Hello deployment 3" > hello.html
	$ git commit -am "Modify deployment number"
	$ git push

Verify that your application code has been pushed to the production gear by
visiting the URL for your application. Open your browser and point the location
to the following URL:

    http://prodapp-ose.userX.onopenshift.com/hello.html

If the deployment was successful, you should see the following displayed in your
web browser:

    Hello deployment 3

##**Rolling back**

Oops! In the previous we made a mistake and accidentally printed "Hello
deployment 3" to the user when we meant to say "Hello deployment 2". Because
this is a mission critical bug (and they all are -- am I right?), the powers
that be require that we immediately fix the situation. To roll back to our last
known good state, we need to list all of our deployments and then reactivate the
last known working one while we address the issue. Let's list all of our
deployments with the following command:

	$ rhc deployment-list

You should see three deployments of your application as follows (with different
timestamps and hashes):

	3:58 PM, deployment 9d8f96ac
	4:04 PM, deployment cc200f43
	4:07 PM, deployment a9b1f5bf

To better understand these deployments, let's explain each one.

"3:58 PM, deployment 9d8f96ac": This is the initial deployment that happened
when we created our OpenShift application. It included the template web
application that is deployed by default.

"4:04 PM, deployment cc200f43": This deployment contains the addition of the
`hello.html` file that displays "Hello deployment 1" to the user.

"4:07 PM, deployment a9b1f5bf": This deployment contains the mission critical
bug that displays "Hello deployment 3" to the user instead of "Hello deployment
2".

At this point, we want to rollback our application code to hash **cc200f43**
that was deployed at 4:04 PM. To roll our code back, enter the following
command:

	$ rhc deployment-activate cc200f43

You should see the following output:

	Activating deployment 'cc200f43' on application prodapp ...
	Activating deployment
	Stopping PHP 5.4 cartridge (Apache+mod_php)
	Waiting for stop to finish
	Waiting for stop to finish
	Starting PHP 5.4 cartridge (Apache+mod_php)
	Application directory "/" selected as DocumentRoot
	Success

We just rolled back our application code to the last known good state. At this
point, you can also fast-forward to the last deployment with a hash of
**a9b1f5bf** if you desire. If you run the following command:

	$ rhc deployment-list

You will see information about the rollback that happened. It should look
similar to this:

	3:58 PM, deployment 9d8f96ac
	4:04 PM, deployment cc200f43
	4:07 PM, deployment a9b1f5bf (rolled back)
	4:11 PM, deployment cc200f43 (rollback to 4:04 PM)

##**Deploying from a branch**

OpenShift Enterprise also supports the ability to deploy code from a Git branch
other than `master`. Using the `rhc app-configure` command, you can list the
current branch from which deployments will be performed.

	$ rhc app-configure

You should see the following output:

	prodapp @ http://prodapp-ose.user1.onopenshift.com/ (uuid: 5372a37e201f1bf35f0000de)
	-----------------------------------------------------------------------------------------
	  Deployment:        auto (on git push)
	  Keep Deployments:  10
	  Deployment Type:   git
	  Deployment Branch: master
	
	Your application 'prodapp' is configured as listed above.

In order change the branch we want to deploy from, issue the following command:

	$ rhc app-configure --deployment-branch mybranch

Verify the change with the following command and notice the new **Deployment
Branch** listed:

	$ rhc app-configure
	
	prodapp @ http://prodapp-ose.user1.onopenshift.com/ (uuid: 5372a37e201f1bf35f0000de)
	-----------------------------------------------------------------------------------------
	  Deployment:        auto (on git push)
	  Keep Deployments:  10
	  Deployment Type:   git
	  Deployment Branch: mybranch
	
	Your application 'prodapp' is configured as listed above.

##**Disabling automatic deployment**

What if you want to commit and push code to your OpenShift Enterprise repository
but not actually deploy it? No problem, we support that now as well. Just turn
off automatic deployment with the following command:

	$ rhc app-configure --no-auto-deploy

Verify your new configuration:

	$ rhc app-configure
	
	You should see the following output:
	prodapp @ http://prodapp-ose.user1.onopenshift.com/ (uuid: 5372a37e201f1bf35f0000de)
	-----------------------------------------------------------------------------------------
	  Deployment:        manual (use 'rhc deploy')
	  Keep Deployments:  10
	  Deployment Type:   git
	  Deployment Branch: mybranch
	
	Your application 'prodapp' is configured as listed above.

You can then use the `rhc app-deploy` command to deploy the changes to your
OpenShift Enterprise gear.

<!--BREAK-->
