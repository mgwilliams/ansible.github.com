Examples
========

.. seealso::

   :doc:`modules`
       A list of available modules
   :doc:`playbooks`
       Alternative ways to use ansible


Parallelism and Shell Commands
``````````````````````````````

Reboot all web servers in Atlanta, 10 at a time::

    ssh-agent bash
    ssh-add ~/.ssh/id_rsa.pub

    ansible atlanta -a "/sbin/reboot" -f 10

The -f 10 specifies the usage of 10 simultaneous processes.

Note that other than the command module, ansible modules do not work like simple scripts. They make the remote system look like you state, and run the commands neccessary to get it there.


Example 2: Time Limited Background Operations
`````````````````````````````````````````````


Long running operations can be backgrounded, and their status can be checked on later. The same job ID is given to the same task on all hosts, so you won't lose track. Polling support is pending in the command line.::

    ansible all -B 3600 -a "/usr/bin/long_running_operation --do-stuff"
    ansible all -n job_status -a jid=123456789

Any module other than 'copy' or 'template' can be backgrounded.


Examples 3: File Transfer & Templating
``````````````````````````````````````

Ansible can SCP lots of files to multiple machines in parallel, and optionally use them as template sources.

To just transfer a file directly to many different servers::

    ansible atlanta copy -a "/etc/hosts /tmp/hosts"

To use templating, first run the setup module to put the template variables you would like to use on the remote host. Then use the template module to write the files using the templates. Templates are written in Jinja2 format. Playbooks (covered below) will run the setup module for you, making this even simpler.::

    ansible webservers -m setup    -a "favcolor=red ntp_server=192.168.1.1"
    ansible webservers -m template -a "src=/srv/motd.j2 dest=/etc/motd"
    ansible webservers -m template -a "src=/srv/ntp.j2 dest=/etc/ntp.conf"

Need something like the fqdn in a template? If facter or ohai are installed, data from these projects will also be made available to the template engine, using 'facter' and 'ohai' prefixes for each.

Examples 3: Deploying From Source Control
`````````````````````````````````````````

Deploy your webapp straight from git::

    ansible webservers -m git -a "repo=git://foo dest=/srv/myapp version=HEAD"

Since ansible modules can notify change handlers (see 'Playbooks') it is possible to tell ansible to run specific tasks when the code is updated, such as deploying Perl/Python/PHP/Ruby directly from git and then restarting apache.

