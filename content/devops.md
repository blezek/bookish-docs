# QIA Jira Configuration

This document describes the Jira configuration for the QIA / RSSU group.

QIA runs the ["starter" version of the Atlassian Jira](https://www.atlassian.com/licensing/starter/#licensingandordering-1) suite including Jira, Confluence, FishEye, Crucible, Stash and Bamboo.  The suite is installed on a VM called `qia-jira.mayo.edu`.

## Server

There is a local account on `qia-jira.mayo.edu` called `jira` (password is `jira` and documented in [qia-device42.mayo.edu](https://qia-device42.mayo.edu/admin/rowmgmt/password/9/)).  All the Atlassian tools are run by the `jira` user.  `init.d` scripts (yes, so old-skool) start up each tool.  They are:

- [Jira](init.d/jira) is the ticket tracking system, it also is the central user authentication and authorization server that all the other tools depend on
- [Bamboo](init.d/bamboo) provides continuous integration services
- [Confluence](init.d/confluence) is an integrated wiki
- [fisheye](init.d/fisheye) is a source code browser and code review system
- [stash](init.d/stash) is a Github clone

All the tools are installed in `/opt/jira/atlassian` and are considered in turn.  Data for each application is stored in `/opt/jira/atlassian/application-data`.

## Database

Each of the Jira tools (except Stash) use the PostgreSQL server running on `qia.mayo.edu`.  Each tool has a separate database and all database migrations are handled by the tool independantly.

## Backup

A [backup script](script/DoBackup) is installed @ `/qia/projects/JiraBackup/`.  It dumps the PostgreSQL database and copies `qia-jira:/ept/jira/atlassian/application-data` locally to `qia.mayo.edu` for eventual backup to DCIS.  The script is relatively simple and straight forward.  It is run by a `crontab` entry:

```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=blezek.daniel@mayo.edu
HOME=/home/mra9161

# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
 10 23  *  *  * /qia/projects/JiraBackup/DoBackup
```

## Jira

[Jira](https://www.atlassian.com/software/jira/) is an issue tracking software.  The QIA Jira server is running @ [http://jira.qia.mayo.edu/](http://jira.qia.mayo.edu/).  The configuration of Jira happens in several places.

- [`/etc/sysconfig/jira`](script/jira) holds environment variables for all the tools (including Jira).  Notably, `JIRA_HOME=/opt/jira/atlassian/jira`
- [`/opt/jira/atlassian/application-data/jira/dbconfig.xml`](jira/dbconfig.xml) database configuration for Jira
- [`/opt/jira/atlassian/jira/atlassian-jira/WEB-INF/classes/jira-application.properties`](jira/jira-application.properties) sets `jira.home = /opt/jira/atlassian/application-data/jira`

On startup, [`/etc/init.d/jira`](init.d/jira) sources `/etc/sysconfig/jira` and launches `$JIRA_HOME/bin/startup.sh`.  This runs Tomcat which looks in the expanded WAR directory `/opt/jira/atlassian/jira/atlassian-jira`.  Finally, Jira looks for the `jira.home` variable set in `jira-application.properties`.

### Jira data directory

All Jira data is stored in `/opt/jira/atlassian/application-data/jira`, mostly consisting of plugins, cache files and attachments to issues.

### Jira admin

The Jira administrative page is [http://jira.qia.mayo.edu/secure/admin/ViewApplicationProperties.jspa](http://jira.qia.mayo.edu/secure/admin/ViewApplicationProperties.jspa), and is restricted to admins.

## Bamboo

[Bamboo](https://www.atlassian.com/software/bamboo/) is a continuous integration and build server running @ [http://bamboo.qia.mayo.edu/](http://bamboo.qia.mayo.edu/).  Bamboo is responsible for checking for `git` changes and building our source code.

Bamboo is configured in by:

- [`/etc/init.d/bamboo`](init.d/bamboo) starts Bamboo by running `/opt/jira/atlassian/bamboo/bin/startup.sh` after settting `BAMBOO_HOME` from `/etc/sysconfig/jira`.

### Bamboo admin

The Bamboo administrative page is [http://bamboo.qia.mayo.edu/admin/administer.action](http://bamboo.qia.mayo.edu/admin/administer.action).  This is the place to configure agents and build settings including global timeout of hung builds.

## Confluence

[Confluence](https://www.atlassian.com/software/confluence/) is a team collaboration software running @ [http://confluence.qia.mayo.edu/](http://confluence.qia.mayo.edu/).  Confluence has largely fallen out of favor for SharePoint.

Confluence is started by:

- [`/etc/init.d/confluence`](init.d/confluence) runs `${CONFLUENCE_HOME}/bin/start-confluence.sh`
- [`/opt/jira/atlassian/confluence/confluence/WEB-INF/classes/confluence-init.properties`](confluence/confluence-init.properties) points to the Confluence home directory `confluence.home = /opt/jira/atlassian/application-data/confluence`
- [`/opt/jira/atlassian/application-data/confluence/confluence.cfg.xml`](confluence/confluence.cfg.xml) configures the database connection for Confluence

### Confluence admin

The Confluence administrative page is [http://confluence.qia.mayo.edu/confluence/admin/console.action](http://confluence.qia.mayo.edu/confluence/admin/console.action).

## Fisheye

[Fisheye](https://www.atlassian.com/software/fisheye/overview) is a source code management software and is installed @ [http://fisheye.qia.mayo.edu/](http://fisheye.qia.mayo.edu/).  Fisheye indexes all the QIA `git` repositories and provides source code reviews via [Crucible](https://www.atlassian.com/software/crucible/overview/).

Fisheye is configured by:

- [`/etc/init.d/fisheye`](init.d/fisheye) runs `${FISHEYE_HOME}/bin/fisheyectl.sh start`
- [`/opt/jira/atlassian/fecru/config.xml`](fisheye/config.xml) sets up the proxy from `qia.mayo.edu` and database settings

### Fisheye admin

Fisheye admin page is [http://fisheye.qia.mayo.edu/admin/admin.do](http://fisheye.qia.mayo.edu/admin/admin.do).  It is protected by a simple password (`jira`).  This admin page is very difficult to find because it is a small link on the footer of each page.  Unfortunately, the footer is pushed down when Fisheye expands the repo history and scrolls away, just when it is visible to be clicked!

## Stash

[Stash](https://www.atlassian.com/software/bitbucket/server) is now Bitbucket server and is available only in the "cloud" in the future.  We have a 10 user license for Stash and it runs @ [http://stash.qia.mayo.edu/](http://stash.qia.mayo.edu/).

Stash is configured by:

- [`/etc/init.d/stash`](init.d/stash) starts up `${STASH_BINDIR}/start-stash.sh`
- [`STASH_HOME`] set to `STASH_HOME="/opt/jira/atlassian/application-data/stash"` in `/etc/init.d/stash`
- [`STASH_INSTALLDIR`] set to `STASH_INSTALLDIR="/opt/jira/atlassian/stash"`
- [`STASH_BINDIR`] set to `STASH_BINDIR="$STASH_INSTALLDIR/bin"`
- [`/opt/jira/atlassian/application-data/stash/shared/stash-config.properties`](stash/stash-config.properties) configures the database, etc for Stash

### Stash admin

The Stash administrative page is @ [http://stash.qia.mayo.edu/admin](http://stash.qia.mayo.edu/admin).  Here Stash users, permissions, etc can be configured.



