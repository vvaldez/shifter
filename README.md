# Shifter - Automated application lift and shift

This project provides a framework for migrating applications from an OS (VM or bare-metal) into a container running on a likely newer platform. This is being referred to here as **lift and shift**. Note that this term is widely used to describe different types of application migrations, but in general refers to migrating applications to a cloud platform rather than refactoring the application to take advantage of the native cloud platform. This is a quicker approach but may require more work to maintain longer term. However, in the case of lifting an application from a legacy OS that may not be taking full advantage of newer hardware, for example, this could be very advantageous.

**Lift and shift** could also apply to other migration types. For example, **lift and shift** of the entire OS in a VM and creating a container from this which would more closely resemble a **pet container**. It could also be possible to **lift and shift** application-specific code using this approach only, and create a container with the required framework in the case of something using JBoss EAP for example.

## Prerequisites
Before running the playbook, you must first define a profile for each of the applications to look for in **inventory/profiles.yml**. This is a list of applications with attributes such as services, packages, files and directories that comprise that application. The example used here is wordpress which uses mysql, httpd services, php-mysql and other packages (see **profiles.yml** for details). 

Next, create matching tasks for this profile in **roles/lift/tasks/{profile}.yml** and **roles/shift/tasks/{profile}.yml** roles using the same name defined in **profiles.yml**. Refer to the **wordpress.yml** in each of the **roles/{lift,shift}/tasks/** directories for examples and see below for a description of each role. Note that the **shift** role also requires files and templates to define the Dockerfile and any supporting scripts to start the container and restore the application.

Lastly, define a hosts file with source hosts to scan and a destination host to lift and shift the matching application to. This initial prototype assumes an Atomic Host but as it simply uses **docker** module it could be any system running docker. Future plans include OpenShift support.

## Roles
### Introspect
This role is called first, unless the **perform_introspection** is overridden to False. In this case you must explicitly assign a profile type to each source host in the inventory using the variable **profile**. Now the playbook is ready to be run. First read about the roles before doing so.

### Lift
This role is called next and is where the matching **roles/lift/tasks/{profile}.yml** is called. This is where each defined profile attribute should be backed up depending on the type of attribute. For example, a mysqldump for a database. Some attributes may not require an action at this stage yet such as any pacakges, but may be used in the **shift** role.

### Shift
This role moves data from the **lift** role to the destination host, then pulls the appropriate docker image, does a **docker build** and starts the container. The Dockerfile and any supporting files should be provided as templates or files in the role with the matching profile name. See **roles/shift/files/wordpress** and **roles/shift/templates/wordpress** for examples. Also see https://github.com/CentOS/CentOS-Dockerfiles for additional ideas as the wordpress example used here was modified from that repo.

## Running Shifter
With the profile and supporting tasks defined, simply run the playbook as such:

```
$ ansible-playbook shifter.yml
```

## Quickstart Example
To use the included wordpress lift and shift profile, create a simple wordpress instance on a CentOS 6 or RHEL 6 system using this site as a guide: https://www.theurbanpenguin.com/creating-wordpress-site-on-rhel-7-1/

Be sure to substitute **mysql** for **mariadb**. The database and wordpress password are set in the **roles/shift/templates/wordpress-start.sh** and hard coded as **shifter** but can be easily made dynamic via a variable.

## Future Plans
I hope to add more example application profiles, include some middleware examples and target OpenShift as a destination.
