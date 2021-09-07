# Puppet SSH

A tool for running puppet command noop through SSH, gather the reports, and display
the results aggregates per servers.
This makes it very useful to deploy changes applying to every servers of an infrastructure,
and have a reports of the exceptions.

## Installation

The script requires ruby, and was tested with ruby 2.6 (it requires ruby at least 1.9 due to
some features used).

The dependencies are specified in `Gemfile` and can be installed with `bundler`.

## Main features

Tool to aggregate the result of puppet run for several servers.
Will aggregate each message by the list of hosts that got it in their puppet run.
Each puppet run is done in a different thread. The output is not streamed, which means
you have to wait until the end of the puppet run to see the output.
The tool batch by group of 10 by default (configurable).

This makes it practical to spot unexpected discrepancies among a fleet of servers when
applying a recipe to a large number of servers.

The run is done with `--noop` argument.

Usage:
```bash
./puppet-ssh puppet0{1..4} --environment production --tags myclass
```

Example output:
```console
[Batch 1/1] puppet01 to puppet04
[Case 1/6] puppet01 puppet02 puppet03 puppet04
[notice] Class[Telegraf::Service]: Unscheduling all events on Class[Telegraf::Service]
[notice] /Stage[main]/Jnx_puppet::Server/File[/etc/sysconfig/puppetserver]/content:
--- /etc/sysconfig/puppetserver YYYY-MM-DD HH:mm:ss +0900
+++ /tmp/puppet-fileXXXX        YYYY-MM-DD HH:mm:ss +0900
@@ -6,7 +6,7 @@
 JAVA_BIN="/usr/bin/java"

 # Modify this if you'd like to change the memory allocation, enable JMX, etc
-JAVA_ARGS="-Xms14848m -Xmx14848m -XX:MaxPermSize=256m"
+JAVA_ARGS="-Xms7168m -Xmx7168m -XX:MaxPermSize=256m"

 # These normally shouldn't need to be edited if using OS packages
 USER="puppet"
[notice] /Stage[main]/Jnx_puppet::Server/File[/etc/sysconfig/puppetserver]/content: current_value '{md5}b3fe7f5f69bb28934c06393010d0a315', should be '{md5}868d6f7bcadbbb3ab4d18f1fbfdc37ca' (noop)

[Case 2/6] puppet02 puppet04
[...]

[Case 3/6] puppet01
[...]

[Case 4/6] puppet02
[...]

[Case 5/6] puppet03
[...]

[Case 6/6] puppet04
[...]
```

A report of the run is also stored in `./puppet_log/`

The idea is that the intended change will be present in the first case (the list that contains all
servers), while the remaining cases are discrepancies that need to be addressed by the user.
For cases > 1, the following actions might be performed by the user:
* Leave it as it is. Puppet is doing its job properly
* Fix the recipe to take into account an unaccounted-for case
* Perform manual action

## Options

```
$./puppet-ssh --help
Usage: ./puppet-ssh [options] HOST...
    -h, --hosts file                 A file containing a list of hosts
    -l, --login user                 User to login with
    -e, --environment env            Puppet environment
    -t, --tags tags                  A comma delimited list of tags
    -b, --batch batch                Batch size
    -s, --source source              A regex to select only certain resources
        --help
```

Note that the `--source` option is a regex for the source part of each message.
Example: `/Stage[main]/MyModule::Myclass/File[file_name]/content` is a source.
