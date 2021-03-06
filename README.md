# NAME

ec2-consistent-snapshot - Create EBS snapshots on EC2 w/consistent filesystem/db

# SUPPORT STATUS

This project is no longer maintained. It is unclear if such a tool is
really necessary on EC2/EBS.

1\. Modern filesystems and databases have built-in crash recovery,
   minimizing the importance of "consistent" snapshots.

2\. If creating consistent snapshots were a real issue, then surely
   Amazon would be recommending it in official documentation and would
   be providing tools to enable it.

3\. This project was written in Perl, which has no SDK officially
   supported by Amazon.

If you are still interested in creating consistent snapshots of file
systems using a supported project, you can check out
[ec2-consistent-snapshot.sh](https://github.com/RideAmigosCorp/ec2-consistent-snapshot.sh)
which is inspired by this project and written by one of the former
maintainers. It has fewer features and options, but uses the
officially supported AWS CLI.

You are also welcome to fork this project and continue working on it
under the terms of the ["LICENSE"](#license).

# SYNOPSIS

    ec2-consistent-snapshot [opts] VOLUMEID...

# OPTIONS

- -h --help

    Print help and exit.

- -d --debug

    Debug mode.

- -q --quiet

    Quiet mode.

- -n --noaction

    Dry run. Just say what you would have done, don't do it.

- --aws-access-key-id KEY
- --aws-secret-access-key SECRET

    Amazon AWS access key and secret access key.  Defaults to
    environment variables or .awssecret file contents described below.

- --aws-access-key-id-file KEYFILE
- --aws-secret-access-key-file SECRETFILE

    Files containing Amazon AWS access key and secret access key.
    Defaults to environment variables or .awssecret file contents
    described below.

- --aws-credentials-file CREDENTIALSFILE

    File containing both the Amazon AWS access key and secret access
    key on seprate lines and in that order.  Defaults to contents of
    $AWS\_CREDENTIALS environment variable or the value $HOME/.awssecret

- --use-iam-role

    The instance is part of an IAM role that that has permission to create
    snapshots so there is no need to specify access key or secret. See ["IAM ROLES"](#iam-roles)
    section for an example Managed Policy with the required permissions.

- --region REGION

    Specify a different EC2 region like "eu-west-1".  Defaults to
    "us-east-1".

- --description DESCRIPTION

    Specify a description string for the EBS snapshot.  Defaults to the
    name of the program.

    You may specify this option multiple times if you need to customize
    descriptions of multiple volumes snapshots. If specified multiple
    times descriptions count has to match volumes count and they will be
    applied on the same order.

- --tag "KEY=VALUE;KEY2=VALUE2"

    If the --tag option is given once, the provided tags will be applied to all
    created snapshots.  Tag keys and values must be separated by '='. Multiple tags
    must be separated by ';'.

    If the --tag option is provided more than once, the tags for each use of --tag
    will apply to each volume in the order that the volumes are provided.

    To check how your tags will be applied, you can use the --noaction flag before
    actually running a snapshot.

- --freeze-filesystem MOUNTPOINT
- --xfs-filesystem MOUNTPOINT \[OBSOLESCENT form of the same option\]

    Indicates that the filesystem at the specified mount point should be
    flushed and frozen during the snapshot. Requires the xfs\_freeze or
    fsfreeze program. Note that xfs\_freeze is equivalent to fsfreeze and
    works on any filesystems that support freezing, provided the kernel
    you are using supports it. (Linux Ext3/4, ReiserFS, JFS, XFS.)
    fsfreeze comes with newer versions of util-linux.

    You may specify this option multiple times if you need to freeze multiple
    filesystems on the the EBS volume(s).

    If no EBS volume ids are specified as command arguments, the specified
    mountpoints will be used along with mount points passed to
    \--no-freeze-filesystem to determine the volume ids.

- --no-freeze-filesystem MOUNTPOINT

    Indicates that the filesystem at the specified mount point should be used for
    volume id discovery if no volume ids are specified as arguments, but that it
    should not be frozen.

    You may specify this options multiple times if you need to discover EBS volumes
    for multiple filesystems.

- --mongo

    Indicates that the volume contains data files for a running Mongo
    database, which will be flushed and locked during the snapshot.

- --mongo-host HOST

    Define host used with the \`--mongo\` option.

    Defaults to 'localhost'.

- --mongo-port PORT

    Define MongoDB port used with the \`--mongo\` option.

    Defaults to 27017

- --mongo-username USER

    Define MongoDB username used with the \`--mongo\` option.

    Defaults to \`undef\`. Required only if authentication is required.

- --mongo-password PASS

    Define MongoDB password used with the \`--mongo\` option.

    Defaults to \`undef\`.  Required only if authentication is required.

- --mongo-stop

    Indicates that the volume contains data files for a running Mongo
    instance.  The instance is shutdown before the snapshot is initiated
    and restarted afterwards. \[EXPERIMENTAL\]

    Uses the \`--mongo-init\` option to stop and start the database ignores
    all other Mongo-related options

- --mongo-init

    The command used to stop and start mongo.

    Defaults to \`/etc/init.d/mongod\`.

    Used only if \`--mongo-stop\` is used. The \`--mongo-init\` command is passed the 'stop' and 'start'
    options during to the stop/start process.

- --mysql

    Indicates that the volume contains data files for a running MySQL
    database, which will be flushed and locked during the snapshot.

    To connect, we will first look in \`--mysql-defaults-file\`, if provided.
    Otherwise, the values of \`--mysql-host\`, \`--mysql-socket\`, \`--mysql-username\`
    and \`--mysql-password\` will be used to build the connection string.

- --mysql-defaults-file FILE

    MySQL defaults file, containing host, username and password, this
    option will ignore the --mysql-host, --mysql-username,
    \--mysql-password parameters

- --mysql-host HOST
- --mysql-socket PATH
- --mysql-username USER
- --mysql-password PASS

    MySQL host, socket path, username, and password used to flush logs and
    lock tables.  User must have appropriate permissions.  Defaults to
    $HOME/.my.cnf file contents.

- --mysql-master-status-file FILE

    Store the MASTER STATUS output in a file on the snapshot. It will be
    removed after the EBS snapshot is taken.  This option will be ignored
    with --mysql-stop

- --mysql-stop

    Indicates that the volume contains data files for a running MySQL database.
    The database is shutdown using \`/usr/bin/mysqladmin stop\` before the snapshot
    is initiated and restarted afterwards using \`/etc/init.d/mysql start\`. Suitable
    for running as root when a few seconds of downtime are acceptable.
    \[EXPERIMENTAL\]

- --percona

    Indicates that the volume contains data files for a running Percona/MySQL
    database, which will be locked using Percona's unique backup locking commands.
    Note: this sets '--mysql' automatically.

- --snapshot-timeout SECONDS

    How many seconds to wait for the snapshot-create to return.  Defaults
    to 10.0

- --lock-timeout SECONDS

    How many seconds to wait for a database lock. Defaults to 0.5.
    Making this too large can force other processes to wait while this
    process waits for a lock.  Better to make it small and try lots of
    times.

- --lock-tries COUNT

    How many times to try to get a database lock before failing.  Defaults
    to 60.

- --lock-sleep SECONDS

    How many seconds to sleep between database lock tries.  Defaults
    to 5.0.

- --pre-freeze-command COMMAND

    Command to run after MySQL stop/lock and before filesystem freeze.

- --post-thaw-command COMMAND

    Command to run immediately after filesystem unfreeze and before MySQL
    start/unlock.

# ARGUMENTS

- VOLUMEID

    EBS volume id(s) for which a snapshot is to be created.

# DESCRIPTION

This program creates an EBS snapshot for an Amazon EC2 EBS volume.  To
help ensure consistent data in the snapshot, it tries to flush and
freeze the filesystem(s) first as well as flushing and locking the
database, if applicable.

Filesystems can be frozen during the snapshot. Prior to Linux kernel
2.6.29, XFS must be used for freezing support. While frozen, a filesystem
will be consistent on disk and all writes will block.

There are a number of timeouts to reduce the risk of interfering with
the normal database operation while improving the chances of getting a
consistent snapshot.

If you have multiple EBS volumes in a RAID configuration, you can
specify all of the volume ids on the command line and it will create
snapshots for each while the filesystem and database are locked.  Note
that it is your responsibility to keep track of the resulting snapshot
ids and to figure out how to put these back together when you need to
restore the RAID setup.

If you have multiple EBS volumes which are hosting different file
systems, it might be better to simply run the command once for each
volume id.

# EXAMPLES

Snapshot a volume with a frozen filesystem under /vol containing a
MySQL database:

    ec2-consistent-snapshot --mysql --freeze-filesystem /vol vol-VOLUMEID

Snapshot a volume with a frozen filesystem under /data containing a
Mongo database:

    ec2-consistent-snapshot --mongo --freeze-filesystem /data vol-VOLUMEID

Snapshot a volume mounted with a frozen filesystem on /var/local but
with no MySQL database:

    ec2-consistent-snapshot --freeze-filesystem /var/local vol-VOLUMEID

Snapshot four European volumes in a RAID configuration with MySQL,
saving the snapshots with a description marking the current time:

    ec2-consistent-snapshot                                      \
      --mysql                                                    \
      --freeze-filesystem /vol                                   \
      --region eu-west-1                                         \
      --description "RAID snapshot $(date +'%Y-%m-%d %H:%M:%S')" \
      vol-VOL1 vol-VOL2 vol-VOL3 vol-VOL4

Snapshot four us-east-1 volumes in a RAID configuration with Mongo,
saving the snapshots with a description marking the current time:

    ec2-consistent-snapshot                                      \
      --mongo                                                    \
      --freeze-filesystem /data                                  \
      --region us-east-1                                         \
      --description "RAID snapshot $(date +'%Y-%m-%d %H:%M:%S')" \
      vol-VOL1 vol-VOL2 vol-VOL3 vol-VOL4

Snapshot two volumes with customized descriptions:

    ec2-consistent-snapshot                                      \
      --description "Description 1st Volume"                     \
      --description "Description 2nd Volume"                     \
      vol-VOL1 vol-VOL2

Snapshot a volume without specifying a volume id (requires ec2:DescribeInstances
permission):

    ec2-consistent-snapshot --freeze-filesystem /vol

# ENVIRONMENT

- $AWS\_ACCESS\_KEY\_ID

    Default value for access key.
    Can be overridden by command line options.

- $AWS\_SECRET\_ACCESS\_KEY

    Default value for secret access key.  Can be overridden by command
    line options.

- $AWS\_CREDENTIALS

    Default value for filename containing both access key and secret
    access key on separate lines and in that order. Can be overriden by
    the --aws-credentials command line option.

# FILES

- $HOME/.my.cnf

    Default values for MySQL user and password are sought here in the
    standard format.

- $HOME/.awssecret

    Default values for access key and secret access keys are sought here.
    Can be overridden by environment variables and command line options.

# INSTALLATION

On most Ubuntu releases, the **ec2-consistent-snapshot** package can be
installed directly from the Alestic.com PPA using the following
commands:

    sudo add-apt-repository ppa:alestic
    sudo apt-get update
    sudo apt-get install ec2-consistent-snapshot

This program may also require the installation of the Net::Amazon::EC2
Perl package from CPAN.  On Ubuntu 10.04 Lucid and higher, this should
happen automatically by the dependency on the libnet-amazon-ec2-perl
package.

On some earlier releases of Ubuntu you can install the required
package with the following command:

    sudo PERL_MM_USE_DEFAULT=1 cpan Net::Amazon::EC2

On Ubuntu 8.04 Hardy, use the following commands instead:

    code=$(lsb_release -cs)
    echo "deb http://ppa.launchpad.net/alestic/ppa/ubuntu $code main"|
      sudo tee /etc/apt/sources.list.d/alestic-ppa.list
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BE09C571
    sudo apt-get update
    sudo apt-get install ec2-consistent-snapshot build-essential
    sudo cpan Net::Amazon::EC2

The default values can be accepted for most of the prompts, though it
is necessary to select a CPAN mirror on Hardy.

<a name="iam-roles"></a>

# IAM ROLES

When authenticating using a IAM Role your role must have the appropriate
permissions.  You can create a single IAM Managed Policy that allows the
necessary permissions and attach the managed policy to each IAM role you would
like to allow to create EC2 snapshots.

The following policy allows both creating the snapshots as the read-only
`DescribeInstances` permission which is required if you want to snapshot a volume
without specifying the volume ID.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "1",
                "Effect": "Allow",
                "Action": [
                    "ec2:CreateSnapshot",
                    "ec2:CreateTags",
                    "ec2:DescribeInstances"
                ],
                "Resource": [
                    "*"
                ]
            }
        ]
    }

If you get an "unauthorized" error when using the `--use-iam-role` option, you
can use the `--debug` option to confirm whether it was the
`DescribeInstances` or `CreateSnapshot` API call that failed.

You might also use IAM policies to allow automating deleting old snapshots
through another tool. Using a separate policy is recommended for that. By
putting the "delete" permission in the same policy, you would be allowing
someone with access to one of your EC2 instances to delete the backups of the
instance volumes. Instead, carefully restrict the delete permission.

# VOLUME DISCOVERY

If no EBS volume ids are passed as arguments to ec2-consistent-snapshot, it
will attempt to discover the volume ids based on the mount points passed to
\--no-freeze-filesystem and --freeze-filesystem.

In order to determine the volume ids, ec2-consistent-snapshot first makes a
call to the EC2 instance metadata service to determine the instance's id. Next,
it makes a call to the DescribeInstances EC2 api to get the list of volumes
attached to the instance. It then compares the device names for each attachment
with a list of device names determined using mountpoint(1) and sysfs.

# SEE ALSO

- Amazon EC2
- Amazon EC2 EBS (Elastic Block Store)
- [aws ec2 create-snapshot](http://docs.aws.amazon.com/cli/latest/reference/ec2/create-snapshot.html)

# CAVEATS

Freezing the root filesystem is not recommended. Be sure to test
each filesystem you use it on before putting this into production, in
exactly the way you would run it in production (e.g., inside cron if
that's how you invoke it).

ec2-consistent-snapshot can hang if its output is directed at a
filesystem that is being frozen, leading to a dead machine.

EBS snapshots are a critical part of protecting your valuable data.
This program or the environment in which it is run may contain defects
that cause snapshots to not be created.  Please test and check to make
sure that snapshots are getting created for the volumes as you intend.

EBS snapshots cost money to create and to store in your AWS account.
Be aware of and monitor your expenses.

You are responsible for what happens in your EC2 account.  This
software is intended, but not guaranteed, to help in that effort.

This program tries hard to figure out some values are for the AWS key
and AWS secret access key.  In fact, it tries too hard.  This results
in possibly using some credentials it finds that are not the correct
ones you wish to use, especially if you are operating in an
environment where multiple sets of credentials are in use.

# BUGS

Please report bugs at https://bugs.launchpad.net/ec2-consistent-snapshot

# CREDITS

Thanks to the following for performing tests on early versions,
providing feature development, feedback, bug reports, and patches:

    David Erickson
    Steve Caldwell
    Gryp
    Ken Huang
    Jefferson Noxon
    Bobb Crosbie
    Craig Tracey
    Diego Salvi
    Christian Marquardt
    Todd Roman
    Ben Tucker
    David Rogeres
    Kevin Lewis
    Eric Lubow
    Seth de l'Isle
    Peter Waller
    yalamber
    Daniel Beardsley
    dileep-p
    theonlypippo
    Tobias Lindgren
    nbfowler
    Mark Stosberg
    Tim McEwan

# AUTHOR/MAINTAINER

Eric Hammond &lt;https://github.com/ehammond>
Mark Stosberg &lt;https://github.com/markstos>

<a name="license"></a>

# LICENSE

Copyright 2009-2018 Eric Hammond

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
