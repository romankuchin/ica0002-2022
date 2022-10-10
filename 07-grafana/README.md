# Lab 7

In this lab we will continue with monitoring. The goal of this lab is to create a place where everyone can get overview of infrastructure state.

## Task 1: Install MySQL exporter

Install MySQL exporter from Ubuntu package repository.

Create MySQL user for it.

Docs: https://github.com/prometheus/mysqld_exporter

Use ~/.my.cnf for passing auth data to MySQL exporter. Find under what user it runs and what forder is home folder for that user. Nobody except user itself can read this file. Nobody can change the file.

Content of ~/.my.cnf:

    [client]
    user=your_user
    password=your_password

Make sure that exporter will be restarted in case username/password change.

**No cleartext passwords in repo**

## Task 2: Install Bind9 exporter

Install Bind9 exporter from Ubuntu package repository.

Expose Bind9 statistics for exporter.

Docs: https://github.com/prometheus-community/bind_exporter

## Task 3: Install Nginx exporter

Install Nginx exporter from Ubuntu package repository.

Make sure that Nginx exposes statistics to exporter.

Docs: https://github.com/nginxinc/nginx-prometheus-exporter/

## Task 4: Install Grafana

Install Grafana.

Docs: https://grafana.com/docs/grafana/latest/installation/debian/#install-on-debian-or-ubuntu

## Task 5: Configure reverse proxy

Add necessary locations to Nginx config:

    - location /grafana -> localhost:(grafana_port)
    
Helpful docs for Grafana: https://grafana.com/tutorials/run-grafana-behind-a-proxy/

Don't add locations that point to unexisting services. You can use this code for checking if we should expect exporter on this VM or not:

    {% if inventory_hostname in groups['prometheus'] %}
    location /prometheus {
        proxy_pass http://localhost:xyz/;
    }
    {% endif %}

## Task 6: Create dashboard in Grafana

Create Grafana `Main` dashboard that will show:
 - CPU load on VMs
 - Memory consumption on VMs
 - Bind9 status + amount of A DNS queries per minute (bind_resolver_queries_total)
 - MySQL status + amount of selects per minute (mysql_global_status_commands_total)
 - Nginx status + amount of requests per minute (nginx_http_requests_total)

Use Prometheus as datasource.

## Task 7: Configure Grafana provisioning

To avoid manual operations every time do the following during Grafana installation:

 - Configure Prometheus as default datasource (https://grafana.com/docs/grafana/latest/administration/provisioning/#data-sources)
 - Precreate `Main` dashboard (https://grafana.com/docs/grafana/latest/administration/provisioning/#dashboards)
 - Precreate user/password

## Expected result

Your repository contains these files and directories:

    ansible.cfg
    group_vars/all.yaml
    hosts
    infra.yaml
    roles/grafana/tasks/main.yaml
    roles/grafana/files/main.json
    roles/mysql_exporter/tasks/main.yaml
    roles/bind_exporter/tasks/main.yaml
    roles/nginx_exporter/tasks/main.yaml

Your repository also contains all the required files from the previous labs.

Your repository **does not contain** Ansible Vault master password.

Grafana, exporters and reverse proxy are installed and configured with this command:

	ansible-playbook infra.yaml

Running the same command again does not make any changes to any of the managed
hosts.

After playbook execution you should be able to:

1. See grafana dashboard by using \<your_VM_http_link\>/grafana.

2. Check your Prometheus web-interface by using \<your_VM_http_link\>/prometheus (lab06).

3. Access Agama by using \<your_VM_http_link\> (lab04).
