# cf-deploy

cf-deploy is a tool designed to deploy configuration files for configuration management tools (CFEngine policies, Puppet manifests...). The script itself is part of a wider strategy where the code repository is designed to support multiple projects at once, sharing common libraries and keeping project-specific code separated.

You can read more about this approach in my blog post ["Git repository and deployment procedures for CFEngine policies"](http://syslog.me/2014/04/07/git-repository-and-deployment-procedures-for-cfengine-policies/). You can also take a look at the [slides of my presentation at Config Management Camp 2015](https://speakerdeck.com/brontolinux/many-projects-one-code).


# WARNING!!!!

The code is perfectly functional but doesn't support an external configuration. All the configuration information is inside the cf-deploy script and the Makefile and you'll need to edit both to suit them to your environment. Or, better, you could **fork this project** and modify cf-deploy so that it reads its defaults from a configuration file and command line options -- I'll gladly accept pull requests to implement that. I am personally fond of [AppConfig](https://metacpan.org/release/AppConfig), but feel free to use any tool that fits.


# Components

The components of the tool are currently the following:

* a Makefile, the real workhorse of the system;
* cf-deploy, a Perl frontend to the Makefile;
* a configuration file called projects.db that establishes a correspondance between a project's name, the directory in which the files for those projects reside, and the type of project: remote (deploy the files to a remote server) or local (deploy the files in a local directory)
* a configuration file called hub.db that associates each hub with its location, the project it is assigned to and the environment it serves: production (prod), preproduction (preprod), staging, test...

A sample for hub.db and projects.db is provided.


# Current defaults

- you are using git; you can use a different tool (e.g.: svn, hg...) by changing the Makefile;
- your local copy of the git repository is checked out in `/var/cfengine/git`, as set in the Makefile variable `LOCALDIR`;
- cf-deploy and its database files are in `/var/cfengine/git/common/tools/deploy`, the name of cf-deploy's "databases" is also hardcoded in the script; **you are supposed to place/link cf-deploy in a directory in your PATH**
- the master branch is deployed by default; it is set as a variable `BRANCH` in the Makefile but overridden in cf-deploy;
- the files are deployed using rsync; you can use a different tool/system or different options for rsync by changing the Makefile;
- when deploying to a remote server, the destination directory is `/var/cfengine/masterfiles` as set in the Makefile variable `MASTERDIR`; it is then overridden in cf-deploy;
- the options using when diff-ing files are set in the Makefile in the variable `DIFF_OPTS`.


# How it works

Just run the command without operands and it will tell you. This is what it returns on my laptop where it is linked in `/var/cfengine/bin`:

```
$ cf-deploy 

Usage:
  /var/cfengine/bin/cf-deploy PROJECT_NAME
    deploys project PROJECT_NAME on all hubs. It's a shortcut for
    /var/cfengine/bin/cf-deploy deploy PROJECT_NAME

  /var/cfengine/bin/cf-deploy deploy PROJECT_NAME [ branch BRANCH ] [ hub SERVER ]
    Deploys PROJECT_NAME with the specified options

  /var/cfengine/bin/cf-deploy preview PROJECT_NAME [ branch BRANCH ] [ hub SERVER ]
    preview the changes that would be applied

  /var/cfengine/bin/cf-deploy diff PROJECT_NAME hub SERVER [ branch BRANCH ]
    runs a diff for a project (hub is mandatory)

  /var/cfengine/bin/cf-deploy show PROJECT_NAME
    Describes the project PROJECT_NAME, other options are ignored

  /var/cfengine/bin/cf-deploy list projects
    Lists all defined projects, other options are ignored

  /var/cfengine/bin/cf-deploy list hubs
    Lists all but the test hubs (normally they are the one you are
    mostly interested in, but in case you are more curious...)

  /var/cfengine/bin/cf-deploy list all_hubs
    List absolutely all hubs, skipping none of them.

  You can override the default branch of a project by using the keyword
  branch. You can also override the project's hub list by specifying an
  hub with the hub keyword -- notice that specifying a hub for a local
  project is pointless

```