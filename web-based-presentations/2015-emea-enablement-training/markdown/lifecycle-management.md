# Lifecycle Management

## Nodes, Gears, Community Cartridges, Applications, OpenShift

---

<!-- .slide: data-background="images/change-management.png" -->

---

# What Changes can occur?

* OpenShift Infrastructure
* Cartridges
    * Custom Cartridges
    * Community Cartridges
    * OpenShift Marketplace
* Applications
* OpenShift Major Versions
* OpenShift Minor Versions (dot releases)


Notes: OpenShift can be changed in various ways. Multiple components can be changed. There are different requirements for each change
and how those changes are reflected within the system. We are going to review each one on its own.

---

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

<!-- .slide: data-background="images/woman-looking-through-magnifying-glass.jpg" -->

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

--

# Preparation

* Ensure a backup is present
* Disable any Change / Configuration Management Software that controls OSE (puppet, chef, etc.)
* Review any dangling `rpmsave` or `.rpmnew` file from previous updates

```
# updatedb
# locate --regex '\.rpm(save|new)$'
```

* Reviw [Asynchronous Errata Updates](https://access.redhat.com/documentation/en-US/OpenShift_Enterprise/2/html-single/Deployment_Guide/index.html#chap-Asynchronous_Errata_Updates)
* Run `yum update` to make sure your system is up to date
* Run `oo-admin-chk` on the broker and `oo-diagnostics -v` on all hosts to ensure environment stability

--

# `ose-upgrade` steps

Broker            | Node(s)
:-----------------|------------------:
`ose-upgrade begin` | `ose-upgrade begin`
`yum install openshift-enterprise-upgrade-broker` | `yum install openshift-enterprise-upgrade-node`
`ose-upgrade pre` | `ose-upgrade pre`
`ose-upgrade outage` | `ose-upgrade outage`





