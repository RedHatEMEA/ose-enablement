
# OpenShift Infrastructure

--

# General Information

* [Online Documentation](https://access.redhat.com/documentation/en-US/OpenShift_Enterprise/2/html-single/Administration_Guide/index.html#Upgrading_OpenShift_Enterprise)
* Multiple Steps
* Can take from 30 minutes up to several hours
* Environment will be in maintenance while upgrading core components

Notes: OpenShift Enterprise relies on a complex set of dependencies; to avoid problems, caution is required when applying software upgrades to broker and node hosts.
For bug fixes and other targeted changes, updated RPMs are released in existing channels. Read errata advisories carefully for instructions on how to safely apply upgrades and details of required service restarts and configuration changes. For example, when upgrading rubygem packages required by the broker application, it is necessary to restart the openshift-broker service. This step regenerates the bundler utility's Gemfile.lock file and allows the broker application and related administrative commands to use the updated gems. See the latest OpenShift Enterprise Deployment Guide at https://access.redhat.com/site/documentation for instructions on how to apply asynchronous errata updates.
For systemic upgrades from previous versions of OpenShift Enterprise requiring formal migration scripts and lockstep package updates, see the latest OpenShift Enterprise Deployment Guide at https://access.redhat.com/site/documentation for instructions on how to use the ose-upgrade tool.

--

# Looking at it in detail <!-- .element: style="color: white" -->

<!-- .slide: data-background="/2015-emea-enablement-training/images/woman-looking-through-magnifying-glass.jpg" -->

--

# One at a time

Upgrades across major or minor versions must be taken one upgrade at a time. For example, to upgrade from 2.0 to 2.2,

you must first use the `ose-upgrade` tool to upgrade from 2.0 to 2.1, then use the tool again to upgrade from 2.1 to 2.2.

--

# Services hit by Outage

* Broker Services are disabled during upgrade
* Applications are unavailable during certain steps of the upgrade. During the outage, users can still access their gears using `SSH`, but should be advised against performing any `Git` pushes
* Red Hat recommends to reboot all Hosts after the update (kernel updates, etc.)
* Updates are usually distributed in new channel repositories (To not confuse `yum update ...`)

--

# `ose-upgrade` Tool Overview

* Series of steps
* Must be executed in order
* Keeps track of steps already executed
* You can skip certain steps (i.e. `--skip`) but make sure you know what you are doing
* The upgrade process is logged under `/var/log/openshift/upgrade.log`
* Use `ose-upgrade status` at any time to view all lists and to see what step needs to be executed next
* Use `ose-upgrade all` on node hosts to accelerate the upgrade (**not advised for brokers**)
* Different flow for different upgrades (1.2->2.0, 2.0->2.1, 2.1->2.2)

--

# Preparation

* Ensure a backup is present
* Disable any Change / Configuration Management Software that controls OSE (puppet, chef, etc.)
* Review any dangling `rpmsave` or `.rpmnew` file from previous updates

```
# updatedb
# locate --regex '\.rpm(save|new)$'
```

* Review [Asynchronous Errata Updates](https://access.redhat.com/documentation/en-US/OpenShift_Enterprise/2/html-single/Deployment_Guide/index.html#chap-Asynchronous_Errata_Updates)
* Run `yum update` to make sure your system is up to date
* Run `oo-admin-chk` on the broker and `oo-diagnostics -v` on all hosts to ensure environment stability

--

# `ose-upgrade` steps 1.2-2.0, 2.0-2.1

Broker           | Node(s)          | BSN(s)
-----------------|------------------|-------------------
`ose-upgrade begin` | `ose-upgrade begin` |
`yum install openshift-enterprise-upgrade-broker` | `yum install openshift-enterprise-upgrade-node` |
`ose-upgrade pre` | `ose-upgrade pre` |
`ose-upgrade outage` | `ose-upgrade outage` |
 | | `yum update`
`ose-upgrade rpms` | `ose-upgrade rpms` |
`ose-upgrade conf` | `ose-upgrade conf` |
`ose-upgrade maintenance_mode` | `ose-upgrade maintenance_mode` |
`ose-upgrade pending_ops` | |

--

# `ose-upgrade` steps 1.2-2.0, 2.0-2.1

Broker           | Node(s)          | BSN(s)
-----------------|------------------|-------------------
 `ose-upgrade confirm_nodes` | |
 `ose-upgrade data` | |
 `ose-upgrade gears` | |
 | `ose-upgrade test_gears_complete` |
 `ose-upgrade end_maintenance_mode` | `ose-upgrade end_maintenance_mode` |
 | `oo-accept-node -v` |
 `ose-upgrade post` | |
 `oo-diagnostics -v` | |
 `oo-admin-chk` | |


--

# Known Issues

1. Because Jenkins applications cannot be migrated, follow these steps to regain functionality:
  1. Save any modifications made to existing Jenkins jobs.
  2. Remove the existing Jenkins application.
  3. Add the Jenkins application again.
  4. Add the Jenkins client cartridge as required.
  5. Reapply the required modifications from the first step.
2. There are no notifications when a gear is successfully migrated but fails to start. This may not be a
migration failure because there may be multiple reasons why a gear fails to start. However, Red Hat
recommends that you verify the operation of your applications after upgrading. The service
`openshift-gears status` command may be helpful in certain situations.

--

# `ose-upgrade` steps 2.1-2.2

Broker           | Node(s)          | BSN(s)
-----------------|------------------|-------------------
 `ose-upgrade begin` | `ose-upgrade begin` |
 `yum install openshift-enterprise-upgrade-broker` | `yum install openshift-enterprise-upgrade-node` |
 `ose-upgrade pre` | `ose-upgrade pre` |
 | | `yum update`
 `ose-upgrade rpms` | `ose-upgrade rpms` |
 `ose-upgrade conf` | `ose-upgrade conf` |

--

# Heads up

Starting with OpenShift Enterprise 2.2, the apache-mod-rewrite front-end server proxy plug-in is
deprecated. New deployments of OpenShift Enterprise 2.2 now use the apache-vhost plug-in as the default.

```
# export OSE_UPGRADE_MIGRATE_VHOST=true
```

## If you do not want to migrate set **nothing** at all

--

# `ose-upgrade` steps 2.1-2.2 (cont.)

Broker           | Node(s)          | BSN(s)
-----------------|------------------|-------------------
 `ose-upgrade maintenance_mode` | `ose-upgrade maintenance_mode` |
 `ose-upgrade pending_ops` | |
 `ose-upgrade confirm_nodes` | |
 `ose-upgrade data` | |
 `ose-upgrade gears` | |
 | `ose-upgrade test_gears_complete` |
 `ose-upgrade end_maintenance_mode` | `ose-upgrade end_maintenance_mode` |
 | `oo-accept-node -v` |
 `ose-upgrade post` | |

--

# `ose-upgrade` steps 2.1-2.2 (cont.)

Broker           | Node(s)          | BSN(s)
-----------------|------------------|-------------------
 `oo-diagnostics -v` | |
 `oo-admin-chk` | |

