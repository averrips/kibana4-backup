kibana4-backup
==============

**Stability: 1 - Stable**

[![Build Status](https://api.travis-ci.org/godaddy/kibana4-backup.png)](https://travis-ci.org/godaddy/kibana4-backup)

[![NPM](https://nodei.co/npm/kibana4-backup.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/kibana4-backup/)

Backup, restore, and deploy changes to Kibana configs, index-patterns, dashboards, searches, and visualizations.  **It should work for all versions of Kibana, not just 4**.  We created the project before realizing it though, hence the name :).

The intention of kibana4-backup is to make sure any changes you make to your Kibana instance will be backed up in source control, with the ability to easily restore them.  Furthermore, it provides a way to deploy changes from source control, to specific environments.  Deploying a dashboard from test to prod is as easy as copying a file into a different folder and commiting the change.

# 2.0 Breaking Change

All dependent packages have been updated to latest, which may drop support for older versions of node.

# 1.0 Breaking Change

File names will now be written as such (pre sanitation): `<type>_<id>_<md5_checksum of id>`.  This is to help combat the issue of duplicate names for different types, as well as case sensitivity issues on OSX.  It is recommended that you start synching to a new repository when upgrading to 1.0.0 (or clean out the old one before running the first sync with 1.0.0).  If you don't, all your existing Kibana objects will be duplicated with the new filename syntax.

# Prerequistes

* The box you install kibana4-backup on must have git installed
* The user you run `kibana4-backup` as must have SSH access to your backup repo
* The ssh private key MUST NOT have a passphrase
* The box you install kibana4-backup on must have firewall access to the elasticsearch HTTP url
* You have an empty git repository where you want to store your kibana4 configs, index-patterns, dashboards, searches, and visualizations.

# Usage

```
npm install -g kibana4-backup
kibana4-backup --clone-directory /tmp/kibana4-backup --elasticsearch-url http://myelasticsearch.com:9200 --repo git@github.com:myorg/myrepo.git
```

The commands above will install kibana4-backup and run it once, targetting the specified elasticsearch instance and git repository.  It will restore (if applicable), deploy (if applicable), and backup items under the .kibana index.  It will do all its work in the /tmp/kibana4-backup directory (creating it if it doesn't exist).  Specifying the directory isn't required, but it's recommended, as the default will be inside the node_module's installation directory, which you may not have access to in a global install.  More on the restore/deploy/backup process in the sections below.

```
kibana4-backup \
  -s http://mytestelasticsearch.com:9200 \
  -r git@github.com:myorg/myrepo.git \
  -e test \
  -d /tmp/kibana4-backup
```

The command above will run kibana4-backup, targetting a specific environment.  If you have multiple elasticsearch instances in different environments, you can move kibana4 dashboards, searches, and visualizations between environments easily using the deploy feature.  Each environment will exist as a different folder in the git repo specified.  Specifying the environment will cause kibana4-backup to target the related environment folder for restore/deploy/backup operations.

We leave process management up to you.  Running kibana4-backup from the command line will only run it a single time.  You could create a cron to run it at an interval.  In the future we'd like to daemonize this an provide a way to run it at an interval.

## Restore

The restore logic is the first step in the process.  If the .kibana index does not exist, the latest backup files from the repo/environment you specify will be PUT'd to the index.  If the .kibana index does exist, this step is skipped.

You can also use the `-o` argument to explicitly restore to a specific git commit sha1.  Using `-o` will deploy everything in the backup directory to the Kibana index.

## Deploy

The deploy logic is the next step in the process.  Any files in the deploy folder under the specified environment are read and PUT'd to the .kibana index, and deleted.  If there are no files, this step is skipped.

## Backup

The last step in the process is to perform the backup. The .kibana index will be pulled from elasticsearch and any new configs/index-patterns/dashboards/searches/visualizations will be saved to the correct environment/backup folder.  The changes will then be committed and pushed to the specified git repo.

## Include kibana4-backup in your project

You can also add kibana4-backup as a dependency in your project and use it programatically

```javascript
var kibanaBackup = require('kibana4-backup');

kibanaBackup({
  repo: 'git@github.com:myorg/myrepo.git',
  elasticsearchUrl: 'http://myelasticsearch.com:9200',
  cloneDirectory: '/tmp/kibana4-backup',
  environment: 'test'
}, function(err, results){
  if(err) throw err;
  console.log(results); //not very useful data at the moment
});
```

# Options

```
  Usage: kibana4-backup [options]

  Options:

    -h, --help                      output usage information
    -V, --version                   output the version number
    -r, --repo <url>                REQUIRED - Git repo to store kibana4 data
    -s, --elasticsearch-url <url>   REQUIRED - Elasticsearch HTTP url you want to target
    -d, --clone-directory <path>    RECOMMENDED - The directory to clone the git repo to. Should be an absolute path, must have write access.
    -e, --environment <env>         The environment you want to target.  Alphanumeric only, including dashes and underscores; no whitespace.  Default is "default"
    -c, --commit-message <message>  Commit message to use when changes are made.  Default is "Backing up %i in %e", where %i is the index and %e is the environment.
    -i, --index <name>              The name of the elasticsearch index you are using for kibana.  Default ".kibana"
    -o, --restore-sha1 <sha1>       Deploy all backup files at the provided git sha1.  Useful for restoring kibana to a previous state.
    -x, --clean                     This will remove the clone directory and force a re-clone.
```

# Other Stuff

We are using [debug](https://github.com/visionmedia/debug), name is 'kibana4-backup'.

```
export DEBUG=kibana4-backup && kibana4-backup ......
```
