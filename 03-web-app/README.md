# Lab 3

Goal of this lab is to deploy a simple web application with Ansible.

The application itself is rather trivial but all the stack needed to get it running may be
challenging to set up.

We recommend approaching the problems one by one, and moving in smaller steps.
This is the fastest way to complete these tasks.


## The application

We couldn't find any good web application for you to practice on -- which is not too easy nor
too difficult to set up, so we've created out own :)

Meet the [AGAMA: A (very) Generic App to Manage Anything](https://github.com/hudolejev/agama).

Goal of this task is to get this application running on server, in the simplest possible but still
automated production-grade way.


## Requirements

There is one additional requirements for this lab:

**If you are using `pip` to install Python packages make sure to install them to
Virtualenv (not system-wide).**

In any case package instllation via APT -- if possible -- is a recommended approach.


## Before you start

Update your Ansible inventory file from the [lab 2](../02-web-server) and change your virtual
machine connection parameters. You can find these on your page
[in this list](http://193.40.156.67/students.html).

Note that this step is needed every next day to run your Ansible successfully;
virtual machines are rebuilt every night, and connection details change.


## Task 1: the application

Create an Ansible role named `agama` that deploys the AGAMA application on the managed host.
If you don't remember the exact file structure in the role subdirectory check out the Ansible
related slides from the [lab 2](../02-web-server).

This role should:
 1. Create a system user named `agama`
 2. Create the directory `/opt/agama` owned by user `agama` where the application will be installed
 3. Install application dependencies
 4. Install the application itself

For step 1 use Asnible module `user` as you did in the previous lab.

For step 2 use Ansible module
[file](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)
with attribute `state: directory`.

For steps 3 and 4 check out
[AGAMA installation instructions](https://github.com/hudolejev/agama#installation).

For step 3 use Ansible modules `apt` as you did in the previous lab to install APT packages.

For step 4 use Ansible module
[get_url](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html)
to download publicly available files from the Internet to the managed host.

Then, update a play `Web server` created in the previous lab and add a role named `agama` to the
role list. Note that this role should be added **before** `nginx` -- we'll need the application to
be fully set up before configuring the web server.

This play should apply a role named `agama` on every web server, i. e. should contain something
like this:

    hosts: web_servers
    roles:
      - agama
      - nginx

You can delete the `users` from them the play, but please don't delete the `roles/users` directory
in your Ansible repository, or any files from it.

Once done, run Ansible to set up Agama, and make sure it executes sucessfully:

    ansible-playbook infra.yaml

After that you can verify that the application is working by running this command on the
**managed host** as user `ubuntu`:

    sudo -u agama AGAMA_DATABASE_URI=sqlite:////opt/agama/test.sqlite3 python3 /opt/agama/agama.py

This is a manual step just to verify the result. Don't do it with Ansible please.

You should see this line in the output, without any errors:

     * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)

Note that it's **not** the proper way to run web applications, but is good enough for this lab to
test your solution at this stage.

If you feel that something is not working as expected -- fix it first before approaching the next
task.


## Task 2: uWSGI

Create an Ansible role named `uwsgi` the deploys [uWSGI](https://uwsgi-docs.readthedocs.io) on the
managed host. uWSGI is an application container server that will run our application.

This role should:
 1. Install uWSGI packages; Ubuntu 20.04 packages that we'll need are named `uwsgi` and
    `uwsgi-plugin-python3`, you can check the package details here: https://packages.ubuntu.com/.
 2. Add the uWSGI configuration for AGAMA application to `/etc/uwsgi/apps-enabled/agama.ini` file;
    requirements:
     - AGAMA application should be run by user `agama`.
     - uWSGI should listen on local interface (`localhost` or `127.0.0.1`); alternatively, you can
       use UNIX socket file if you want.
     - AGAMA should be configured to use SQLite3 database located at `/opt/agama/db.sqlite3`.
       No need to _create_ that file explicitly though, AGAMA will create it automatically on the
       first request.
 3. Ensure that uWSGI service is started (unconditionally) and enabled to start automatically on
    system boot.
 4. Ensure that uWSGI service is restarted automatically if uWSGI configuration is changed;
    note that uWSGI service shoud **not** be restarted if uWSGI configuration was not changed.

> Note:
> uWSGI on Debian/Ubuntu is pre-configured to read existing configuration files automatically from
> `/etc/uwsgi/apps-enabled` directory. This is Debian/Ubuntu specific behavior brought to you by
> `uwsgi` package from the APT repository; default (upstream) uWSGI is configured differently.

For step 2 use the Ansible module `copy` and check out
[AGAMA deployment instructions](https://github.com/hudolejev/agama#running) --  it has uWSGI
configuration file example for MySQL; you will need to adjust it for SQLite.

If you feel that AGAMA configuration example is not enough -- uWSGI configuration reference can be
found [here](https://uwsgi-docs.readthedocs.io/en/latest/Options.html).

For step 3 use Ansible module
[service](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html)
to manage system services (start, enable etc.).

For step 4 use
[Ansible handlers](https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html)
to handle service restarts correctly; also check out the lecture slides about Ansible handlers.

You can delete the task that updates the index page added in the lab 2, and the index file itself
from your repository. We won't need that task anymore.

Then, update a play `Web server` and add a role named `uwsgi` to the role list. Note that this role
should be added **after** `agama` but **before** `nginx` -- we'll need the application to be fully
set up before configuring uWSGI, and that should be completed before configuring the web server:

    roles:
      - agama
      - uwsgi
      - nginx

Once done, run Ansible to set up uWSGI, and make sure it executes sucessfully:

    ansible-playbook infra.yaml

After that you can verify that uWSGI is started by running this command manually on a managed host
(port number will be different if you changed it):

    ss -l | grep 5000

You should see the output very similar to this (last column may differ):

    tcp  LISTEN  0  100  127.0.0.1:5000  0.0.0.0:*

This is an indication that uWSGI is set up correctly.

If you feel that something is not working as expected -- fix it first before approaching the next
task.

Hints:
 - uWSGI logs can be found in `/var/log/uwsgi/app` directory. If something is not working as
   expected you will probably find an answer there. You can read logs using `cat`, `tail`, `less`,
   `grep` or any other tools available.
 - You can additionaly run `service uwsgi status` on the managed host to check the uWSGI system
   service status, and view a few latest logged messages -- it is surely helpful for debugging.
 - To test the automatic service restart you can simulate a configuration file change by adding or
   deleting an empty line to it; this change means nothing to uWSGI but triggers a change for
   Ansible.


## Task 3: Nginx as uWSGI frontend

In the previous lab we've installed Nginx web server that served static documents (default mode).
We'll now need to reconfigure it to "talk" to uWSGI to generate dynamic documents instead.

Update the `nginx` role from the previous lab so that Nginx is configured as a
frontend for uWSGI. For that,

 1. Add new file to your Ansible repository: `roles/nginx/files/default`; this file should contain
    a Nginx configuration as uWSGI frontend -- you can find related examples in the lecture slides
    and in the AGAMA deployment instructions (section 'Running')
 2. Use Ansible module `copy` to replace the `/etc/nginx/sites-enabled/default` file on a managed
    host with your copy; this file comes from the APT package `nginx` and you can safely overwrite
    it for our labs.
 3. Ensure that Nginx service is restarted automatically if Nginx configuration is changed; note
    that Nginx service shoud **not** be restarted if Nginx configuration was not changed.
 4. Ensure that Nginx service is started (unconditionally) and enabled to start automatically on
    system boot.

For steps 3 use Ansible handlers, and for step 4 -- Ansible module `service`. Solution here should
be pretty similar to what you did with uWSGI in the previous task.

Make sure that Nginx listens on a public interface port 80 -- otherwise your public URL just won't
work. This line in the `server` section of Nginx configuration should solve it:

    listen 80 default_server;

If you feel that examples mentioned above are not enough -- Nginx uWSGI module configuration
reference can be found [here](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass).

`Web server` play should already contain the `nginx` role from the previous lab; if it doesn't --
add the role after the `uwsgi`.

Once done, run Ansible reconfigure Nginx, and make sure it executes sucessfully:

    ansible-playbook infra.yaml

Once done, AGAMA should be available at [your public URL](http://193.40.156.67/students.html).

Feel free to click around and break all the things. If you feel that AGAMA app has some issues
please consider [reporting them](https://github.com/hudolejev/agama#contributing).

If you need to 'reset' the app just delete the `/opt/agama/db.sqlite3` file on the managed host;
AGAMA will re-create it (if missing) on the next request.

Hints:
 - Nginx logs can be found in `/var/log/nginx` directory.


## Expected result

Your repository contains these files and directories; other files may (and should) be there but
these are the ones that we'll check:

    ansible.cfg
    hosts
    infra.yaml
    roles/
        agama/
            tasks/main.yaml
        nginx/
            files/default
            tasks/main.yaml
        uwsgi/
            files/agama.ini
            tasks/main.yaml

Your repository also contains all the required files from the previous labs.

AGAMA with uWSGI and Nginx can be installed and configured on empty machine by running exactly this
command exactly once:

    ansible-playbook infra.yaml

Running the same command again does not make any changes on the managed host.

AGAMA web application is available on [your public URL](http://193.40.156.67/students.html).
