
# Cartridge Management

--

# New and Noteworthy

- since OpenShift Enterprise 2.1 we have `oo-admin-ctl-cartridge`
- Cartridges are now managed on the broker
- Stores Metadata about cartridges in MongoDB
- New Cartridges need to be activated before they can be consumed
- Empowers broker to track deployed cartridge versions and capabilities to applications
  - This information is important for (starting, stopping, scaling, and deleting)
  - which node host can receive application scale events, etc.
  - broker creates unique Cartridge ID's for each imported Cartridge
    - Timestamp
    - Cartridge ID
- Allows multimple cartridge versions to share cartridge name

--

# Software Versions vs. Cartridge Versions

- Cartridges can be installed via RPM or source directories (git repositories, etc.)
- Cartridges might provide more than one version (i.e. `openshift-origin-cartridge-ruby`)
  - ruby 1.8 and ruby 1.9
- Cartridges provide a manifest
  - Parts of manifest information is imported to broker by `oo-admin-ctl-cartridge`
- Each Cartridge manifest provides a _cartridge version_
- Cartridge Versions are different from Software Versions (i.e. RPM versions, etc.)
- Deactivated cartridges remain active for running applications (scaling, move) but can not be chosen for new applications

--

# Activating, Deactivating, Importing Cartridges

Cartridges can be _activated_ automatically after an import

import profiles from a random node in each gear profile

```
broker# oo-admin-ctl-cartridge -c import-profile --activate
```

Importing Manifests from downloadable Cartridges

```
broker# oo-admin-ctl-cartridge -c import --url URL_to_Cartridge_Manifest --activate
```

List all imported cartridges

```
broker# oo-admin-ctl-cartridge -c list
* cron-1.4         plugin    Cron 1.4              2014/06/16 22:09:55 UTC
* jenkins-client-1 plugin    Jenkins Client        2014/06/16 22:09:55 UTC
  mongodb-2.4      service   MongoDB 2.4           2014/06/16 22:09:55 UTC
* mysql-5.1        service   MySQL 5.1             2014/06/16 22:09:55 UTC
* mysql-5.5        service   MySQL 5.5             2014/06/16 22:09:55 UTC
  ruby-1.8         web       Ruby 1.8              2014/06/16 22:09:55 UTC
* ruby-1.9         web       Ruby 1.9              2014/06/16 22:09:55 UTC
* haproxy-1.4      web_proxy Web Load Balancer     2014/06/16 22:09:55 UTC
```

--

# Activating, Deactivating, Importing Cartridges (cont.)

Cartridges can be activated and deactivated manually

```
broker# oo-admin-ctl-cartridge -c activate --name Cart_Name1,Cart_Name2,Cart_Name3
broker# oo-admin-ctl-cartridge -c deactivate --name Cart_Name1,Cart_Name2,Cart_Name3
```

Existing Applications and scaled gears remain with old (inactive) cartridge versions

```
broker# oo-admin-ctl-cartridge -c migrate
```

Updates Application in MongoDB Datastore to use newly activated cartridges during scaling events.
_Existing_ gears are left __untouched__! To update those use

```
broker# oo-admin-upgrade
```

can be used to upgrade downloadable cartridges as well

--

# Removing Unused Inactive Cartridges

```
broker# oo-admin-ctl-cartridge -c list

* cron-1.4         plugin    Cron 1.4              2014/06/16 22:09:55 UTC
* jenkins-client-1 plugin    Jenkins Client        2014/06/16 22:09:55 UTC
  mongodb-2.4      service   MongoDB 2.4           2014/06/16 22:09:55 UTC
* mysql-5.1        service   MySQL 5.1             2014/06/16 22:09:55 UTC
* mysql-5.5        service   MySQL 5.5             2014/06/16 22:09:55 UTC
  ruby-1.8         web       Ruby 1.8              2014/06/16 22:09:55 UTC
* ruby-1.9         web       Ruby 1.9              2014/06/16 22:09:55 UTC
* haproxy-1.4      web_proxy Web Load Balancer     2014/06/16 22:09:55 UTC

broker# oo-admin-ctl-cartridge -c clean

Deleting all unused cartridges from the broker ...
539f6b336892dff17900000f # ruby-1.8
# 539f6b336892dff179000012 mongodb-2.4                         1
```

Removes unused inactive cartridges from MongoDB Datastore. Inactive Cartridges that cannot be removed
because there are still existing applications/gears using it are listed with a leading _#_. At the end of the line
you see the number of applications using it.

--

# Cartridge Types

Type      | Description                       | RH Supported?
----------|-----------------------------------|--------------
Standard  | Shipped with OpenShift Enterprise | Yes, with OSE Entitlement
Premium   | Shipped with OpenShift Enterprise | Yes, requires OSE premium Entitlement
Custom    | Developed by Users                | No
Community | <!-- .element: style="font-size: 0.7em" --> These cartridges are contributed by the community. See the OpenShift Origin Index at http://origin.ly to browse and search for many community cartridges.  | No
Partner   | These cartridges are developed by third-party partners. | No, but could be supported through a partner

--

# Custom and Community Cartridges vs. Downloadable Cartridges

- Custom and Community Cartridges are installed locally on OpenShift Enterprise Nodes and imported into the Broker
- Downloadable Cartridges are usually downloaded through application creation
- Since 2.1 you can download manifest metadata into MongoDB datastore using
```
oo-admin-ctl-cartridge -c import --url URL_to_Cartridge_Manifest
```
  - Makes downloadable cartridges available during app creation
  - Cartridge Sources are still hosted externally

--

# Installing Custom and Community Cartridges

Run `oo-admin-cartridge` on each node host

```
node# oo-admin-cartridge --action install --source /path/to/cartridge/
```

Verify everything went correct

```
node# oo-admin-cartridge --list
    (redhat, jenkins-client, 1.4, 0.0.1)
    (redhat, haproxy, 1.4, 0.0.1)
    (redhat, jenkins, 1.4, 0.0.1)
    (redhat, mock, 0.1, 0.0.1)
    (redhat, tomcat, 8.0, 0.0.1)
    (redhat, cron, 1.4, 0.0.1)
    (redhat, php, 5.3, 0.0.1)
    (myvendor, mycart, 1.1, 0.0.1)
    (redhat, ruby, 1.9, 0.0.1)
    (redhat, perl, 5.10, 0.0.1)
    (redhat, diy, 0.1, 0.0.1)
    (redhat, mysql, 5.1, 0.2.0)
```

this displays _vendor name_, _cartridge name_ and available _version(s)_

--

# Installing Custom and Community Cartridges (cont.)

Restart mcollective service on each node host

```
node# service ruby193-mcollective restart
```

Update the cartridge lists on the broker

OSE < 2.1 | OSE 2.1+
----------|---------
`broker# oo-admin-broker-cache --clear --console` | `broker# oo-admin-ctl-cartridge -c import-profile --activate`
                                                  | `broker# oo-admin-console-cache --clear`

--

# Removing Custom and Community Cartridges

For OpenShift Enterprise 2.1+ run

```
broker# oo-admin-ctl-cartridge -c deactivate --name Cart_Name
```

List the cartridges to obtain the cartridge version and the software version and the cartridge name

```
broker# oo-admin-cartridge --list
```

Migrate all applications to the new version

```
broker# oo-admin-upgrade
```

--

# Removing Custom and Community Cartridges (cont.)

Run the following command on each node to remove the cartridge from the cartridge repository

```
node# oo-admin-cartridge --action erase \
                         --name Cart_Name \
                         --version Software_Version_Number \
                         --cartridge_version Cart_Version_Number
```

OSE < 2.1 | OSE 2.1+
----------|---------
`broker# oo-admin-broker-cache --clear --console` | `broker# oo-admin-console-cache --clear`

--

# Upgrading Custom and Community Cartridges using `oo-admin-upgrade`

oo-admin-upgrade can be used to upgrade:

- All gears in an OpenShift Enterprise Environment
```
broker# oo-admin-upgrade upgrade-node --version=<version>
```
- All gears on a single Node
```
broker# oo-admin-upgrade upgrade-node <node.example.com> \
                 --version=<version>
```
- Or a single gear/application
```
broker# oo-admin-upgrade upgrade-gear --app-name=<app_name>
```

Uses Information from the _broker_ host and instruments _mcollective_ to trigger the upgrade of a gear.

--

# `oo-admin-upgrade` Upgrade Process Overview

1. Load the gear upgrade extension, if configured.
2. Inspect the gear state.
3. Run the gear extension's pre-upgrade script, if it exists.
4. Compute the upgrade itinerary for the gear.
5. If the itinerary contains an incompatible upgrade, stop the gear.
6. Upgrade the cartridges in the gear according to the itinerary.
7. Run the gear extension's post-upgrade script, if it exists.
8. If the itinerary contains an incompatible upgrade, restart and validate the gear.
9. Clean up after the upgrade by deleting pre-upgrade state and upgrade metadata.

--

# Quickstarts

- Not available in OpenShift Enterprise by default
- See [OpenShift QuickStart Developer's Guide](https://developers.openshift.com/en/get-involved-extend-openshift.html#create-a-quickstart) on how to create your own
- You can add all Community Quickstarts _or_ enable them one-by-one using the JSON definition

--

# Adding Quickstarts

Edit `/etc/openshift/quickstarts.json`

```
[
   {
      "updated_at": "2014-06-06T06:17:45Z",
      "owner": "https://github.com/juhoffma",
      "id": 20253423,
      "description": "Downloadable Cartridge for OpenShift Online and Enterprise",
      "size": 27054,
      "forks": 1,
      "watchers": 1,
      "name": "openshift-origin-cartridge-cassandra",
      "language": "C++",
      "created_at": "2014-05-28T10:19:18Z",
      "stargazers": 1,
      "owner_type": "User",
      "default_app_name": "openshiftorigincartridgecassandra",
      "cartridges": [
         ""
      ],
      "git_repo_url": "https://github.com/juhoffma/openshift-origin-cartridge-cassandra.git",
      "type": "cartridge",
      "submitted_at": "2014-06-09T18:18:59.578162",
      "owner_name": "Juergen Hoffmann",
      "owner_avatar_url": "https://avatars.githubusercontent.com/u/998315?"
   },
]
```

Make sure the cartridges defined in the quickstart are available to developers

--

# Disabling Downloadable Cartridges

You can configure the broker to disable downloadable cartridges for security reasons

In _/etc/openshift/broker.conf_ set the following variable to __false__ and restart the Broker Service

```
DOWNLOAD_CARTRIDGES_ENABLED="false"
broker# service openshift-broker restart
```

--

# Obsolete Cartridges

Obsolete cartridges represent technologies, or versions of technologies, for which you do not want
developers to be able to deploy new applications or add-on cartridges, but that are still required
by the applications already using them.

Edit _/etc/openshift/broker.conf_ and follow the below steps:

```
ALLOW_OBSOLETE_CARTRIDGES="false"
```

