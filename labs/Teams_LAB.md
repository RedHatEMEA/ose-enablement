#**Lab: Working with Development Teams**


**Server used:**

* localhost

**Tools used:**

* `rhc` client tools
* `git`

During this lab we are going to explore how to use the OpenShift Enterprise
platform to manage a team of developers that may be working on the same
application.  Learning how to utilize the access controls and permission system
that is built into OpenShift Enterprise will provide the account owner the
authority to grant specific rights to an individual developer.  The current
access permissions that can be assigned by the account owner are **view**,
**edit**, and **administer**.

The **view** permission level only allows the user to view the details of an
application for the domain, including the name, add-on cartridges, number of
gears, amount of storage, etc.  A user with the **view** access role will not be
able to clone the Git repository, SSH to the gear, stop or start the
application, or embed add-on cartridges.  This role is for viewing information
about the running applications under the domain.

The **edit** permission level will allow the user to perform all functions on the
domain such as adding cartridges, creating new applications, deleting
applications, viewing and changing source code and triggering new deployments.
However, this role does not grant the permission to modify, add or delete users
from the domain access list.

The **administer** permission level allows you to give full control of the domain
to another user. Caution should be observed when granting this permission level,
as the added user will be able to modify any setting for the specified domain.


##**Setting up multiple domains**

OpenShift Enterprise allows a user to create multiple domains that are
associated with their account.  Keep in mind that the domain in this context
does not refer to the domain name of the application, but rather to a unique
identifier for grouping your applications.  For instance, you may have a domain
for different environments that your application will go through before reaching
production.  These domains could be named **dev**, **qa**, **stage**, and
**production**.  You can create additional domains for your account by using the
command line or the web console.  

###**Creating a domain with the command line**

To create a new domain called **osedev** using the command line, enter in the
following command:

	$ rhc domain create osedev

If the command was successful, you should see the following confirmation message:

	Creating domain 'osedev' ... done
	You may now create an application using the 'rhc create-app' command

Create additional domains called **osetest** and **oseprod**.

##**Adding members to a domain**

Now that we understand the different roles that can be assigned to a user for a
specific domain, it's time to learn how to add new members to collaborate with.
The OpenShift Enterprise Platform allows you to add additional members to your
domain as long as the user you are adding has an existing account on the
platform.  

Add a new user to the **osedev** domain with the following command:

	$ rhc member add demodev@redhat.com -n osedev

You will see the following error message:

	Adding 1 editor to domain ... There is no account with login demodev@redhat.com.

This is because the **demodev** user has not been created it.  In order to create
this user, log in to your **broker host** and issue the following command:

	# htpasswd /etc/openshift/htpasswd demodev@redhat.com

Give the **demodev@redhat.com** user a default password of **changeme**.

Continue to add accounts for both a **demotest@redhat.com** and a
**demoprod@redhat.com** user.

Now that we have added some additional users to the OpenShift Enterprise PaaS,
let's add the **demodev@redhat.com** user to our domain.

	$ rhc member add demodev@redhat.com -n osedev

You will again see an error:

	Adding 1 editor to domain ... There is no account with login demodev@redhat.com.

Why is that? Even if valid **credentials** exist for a user, until that user
actually attempts to log into OpenShift (or an administrator manually creates
the user account), no account exists. In other words, only two mechanisms cause
accounts to be created in OpenShift:

1. A user's first login/interaction
2. Manual action by an administrator

One easy way to get the account created is to use the command line tool as the
desired user and perform some simple action. For example, specify your
**demodev@redhat.com** user and try to list domains:

	$ rhc domain list -l demodev@redhat.com

You will be asked for a password, and then likely see something like the
following output:

    In order to deploy applications, you must create a domain with 'rhc setup'
    or 'rhc create-domain'.

Completing this login has successfully created the **demodev@redhat.com**
account. Now you can add the **demodev@redhat.com** user to the domain
with the following command:

	$ rhc member add demodev@redhat.com -n osedev

This time, you should see a confirmation message:

	Adding 1 editor to domain ... done


##**Modifying members of a domain**

An interesting thing that you may have noticed is that we did not specify the
role that we wanted to be applied to the member.  In the case where a role is
not specified, the OpenShift platform will default the membership to the
**edit** role.  Let's imagine that we made a mistake and actually wanted to add
the new member under the view only role.  In order to change a role for a
member, you can simply execute the `rhc member add` command again while also
specifying the role to be applied to the member.  For example, to update the
role for the **demodev@redhat.com** member, we could simply in the following
command:

	$ rhc member add demodev@redhat.com -n osedev --role view

##**Listing and removing members of a domain**

In order to list all of the members for a particular domain, you can use the
`member list` command as shown below:

	$ rhc member list -n osedev

You should see the following members:

	Login              Role          Type
	------------------ ------------- ----
	demo               admin (owner) user
	demodev@redhat.com view          user

To remove a member from a domain, enter in the following command:

	$ rhc member remove demodev@redhat.com -n osedev


##**Promoting code between environments**

One of the most common questions customers have about the OpenShift Enterprise
platform is about how to perform code promotion between different environments.

Organizations want to allow developers to have full control over the development
environment where they can create gears on demand in order to take advantage of
the speed in which they can develop the software.

The QA team wants full access to the QA environment as well as being the only
ones who can provision servers and deploy stable builds from the development
environment.  Once the QA team has deployed the application to their
environment, they don't want pesky developers to be able to login to the gears
and modify configuration settings or code that may invalidate their test case.

In the production environment, the system administrators and release engineers
need full control over the environment while locking out both the development
and quality assurance teams.

Fortunately, using the information you have learned in this chapter, you should
have the tools and knowledge to accomplish this common scenario by creating
additional domains and assigning proper membership and roles to each domain.

For this scenario to work properly, you would need to create three domains:
**osedev**, **osetest**, and **oseprod**.  Once the domains have been created, you
can then begin assigning access permissions to each member on the corresponding
team that needs access to that environment.  The following diagram shows how you
could configure your account for this scenario:

![](http://training.runcloudrun.com/ose2/promote.png)


##**Time to think**

Given what you have learned in this chapter, create a scenario where code
promotion can happen as described in the image above.  Once you have created
your users and assigned the correct permissions to each domain, you can promote
code using the following syntax:

	$ rhc app create firstphp -n osetest --from-app osedev/firstphp -l \
        demotest@redhat.com

What is the significance of this command?

* You are creating an application
* As the user **osetest**
* In the **osetest** domain
* But sourcing code from the **osedev** domain

Can you see how this relates to code promotion between environments?

**Hint: You will have to perform a few actions *before* you can execute the
above command.**

**Hint: if you see an error that says "Specifying a region on application
creation has been disabled. A region may be automatically assigned for you.",
make sure to update your `rhc` gem/installation.**

This may take a while for you to get correct.

**Lab Complete!**
<!--BREAK-->
