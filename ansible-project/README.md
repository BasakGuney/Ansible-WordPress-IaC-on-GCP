# Ansible WordPress IaC on GCP   

#
![Ansible](https://img.shields.io/badge/automation-ansible-red?logo=ansible)
![Jenkins](https://img.shields.io/badge/CI--CD-Jenkins-blue?logo=jenkins)
![Prometheus](https://img.shields.io/badge/monitoring-prometheus-orange?logo=prometheus)
![Grafana](https://img.shields.io/badge/dashboards-grafana-F46800?logo=grafana)
![Grafana Loki](https://img.shields.io/badge/logging-Grafana%20Loki-0A0A0A?logo=grafana)
![MySQL](https://img.shields.io/badge/database-MySQL-blue?logo=mysql)
![Redis](https://img.shields.io/badge/cache-Redis-red?logo=redis)
![WordPress](https://img.shields.io/badge/CMS-WordPress-21759b?logo=wordpress)
![Gitea](https://img.shields.io/badge/git-Gitea-609926?logo=gitea)
![Semgrep](https://img.shields.io/badge/test-Semgrep-yellow?logo=semgrep)
![Supervisor](https://img.shields.io/badge/process-Supervisord-lightgrey)

</br>
</br>

# Table of Contents

### 1. [**Overview**](#overview)  
### 2. [**Key Features**](#key-features)  
### 3. [**Filesystem**](#filesystem)  
### 4. [**Server Roles, Software, and Descriptions**](#servers)  
### 5. [**WordPress Autoscaling and Load Balance**](#load-balance)  
### 6. [**Playbook Descriptions**](#playbook-descriptions) 
### 7. [**Usage**](#usage) 

</br>
</br>

<a name="overview"></a>
# 1. Overview
The project provides modular WordPress infrastructure that is fully automated using Ansible and deployed on Google Cloud Platform (GCP). It demonstrates a DevOps approach by combining infrastructure as code, CI/CD pipelines, monitoring, logging, and service orchestration across six dedicated servers.

</br>
</br>

<a name="key-features"></a>
# 2. Key Features

### 1. **MySQL with Dedicated Disk**
- MySQL is installed and configured under `supervisord`.
- A dedicated GCP persistent disk is attached for storing MySQL data.
- The default data directory is migrated to this disk for improved manageability and persistence.

### 2. **WordPress Setup with Redis Caching**
- Two separate WordPress sites are deployed.
- Redis is installed and configured to provide object caching for WordPress.
- Nginx is set up as the reverse proxy to serve both sites with isolated configuration.

### 3. **Gitea – Self-Hosted Git Service**
- Gitea is installed as a self-hosted Git platform.
- WordPress source files are tracked in Gitea repositories.
- Gitea serves as the central source control for the WordPress application.

### 4. **Jenkins CI/CD Pipeline**
- Jenkins is installed and managed by `supervisord`.
- A pipeline is created that triggers automatically when changes are pushed to Gitea.
- The pipeline performs the following steps:
  - Pulls the latest code from the Gitea repository.
  - Updates local WordPress files.
  - Runs security checks using **Semgrep** static analysis.

### 5. **Monitoring with Prometheus and Grafana**
- Prometheus collects metrics from all servers and services.
- Grafana dashboards visualize these metrics in real-time.
- Alerting rules are configured in Grafana for critical thresholds.

### 6. **Centralized Logging with Loki**
- Loki is deployed for log aggregation and visualization.
- An additional persistent disk is mounted on the log server at `/mnt/loki-data`.
- Logs are stored on this disk to prevent data loss and enable scalability.
- Logs are queried and visualized through Grafana.

### 7. **Log Management with Logrotate**
- Logs from all services (MySQL, Redis, Gitea, Jenkins, Prometheus, Loki, Nginx, etc.) are managed by `logrotate`.
- Retention periods and rotation settings are controlled via the `logrotate_services` variable.
- This ensures disk space is used efficiently and old logs are pruned based on policy.

### 8. **WordPress Autoscaling and Load Balance**
- The master WordPress instance holds the canonical application files and serves as the synchronization point for Git-driven updates and Jenkins deployments.
- Autoscaled instances are provisioned from a template, connect to the same central database, and mount shared content from the master node over NFS.
- A global load balancer monitors instance health and evenly distributes incoming traffic, automatically adjusting the number of active instances based on demand.

## Technologies Used

- **Google Cloud Platform (GCP)** – Compute Engine, Disks, Firewall, Load Balance
- **Ansible** – Configuration management
- **Supervisord** – Process manager for all services
- **MySQL** – Relational database for WordPress
- **Redis** – In-memory cache for WordPress
- **WordPress** – Two independent instances
- **Nginx** – Web server and reverse proxy
- **Gitea** – Self-hosted Git service
- **Jenkins** – Continuous Integration and Deployment
- **Semgrep** – Static security analysis
- **Prometheus** – Monitoring and metrics
- **Grafana** – Dashboards and alerts
- **Loki** – Centralized log aggregation
- **Logrotate** – Log retention and cleanup


</br>
</br>

<a name="filesystem"></a>
# 3. Filesystem
```text
.
├── README.md
├── main.yml
└── roles
    ├── delete-all
    │   └── tasks
    │       └── main.yml
    ├── firewall-rules
    │   └── tasks
    │       ├── allow-host.yml
    │       ├── allow-internal.yml
    │       ├── deny-all-ingress.yml
    │       └── main.yml
    ├── installations
    │   ├── tasks
    │   │   ├── add-loki-data-disk.yml
    │   │   ├── add-mysql-data-disk.yml
    │   │   ├── add-nginx-dashboard.yml
    │   │   ├── add-prometheus-dashboard.yml
    │   │   ├── add-promtail-dashboard.yml
    │   │   ├── configure-gitea.yml
    │   │   ├── configure-grafana.yml
    │   │   ├── configure-jenkins.yml
    │   │   ├── configure-loki.yml
    │   │   ├── configure-prometheus.yml
    │   │   ├── configure-promtails.yml
    │   │   ├── configure-retentions.yml
    │   │   ├── configure-word-press.yml
    │   │   ├── create-app-alert.yml
    │   │   ├── create-cpu-alert.yml
    │   │   ├── create-db.yml
    │   │   ├── create-pipeline.yml
    │   │   ├── create-repos.yml
    │   │   ├── create-word-press.yml
    │   │   ├── get-internal-ips.yml
    │   │   ├── install-gitea.yml
    │   │   ├── install-grafana.yml
    │   │   ├── install-jenkins.yml
    │   │   ├── install-loki.yml
    │   │   ├── install-node-exporter.yml
    │   │   ├── install-prometheus.yml
    │   │   ├── install-promtail.yml
    │   │   ├── install-redis.yml
    │   │   ├── install-semgrep.yml
    │   │   ├── main.yml
    │   │   ├── nginx-prometheus-exporter.yml
    │   │   └── set-supervisord.yml
    │   ├── templates
    │   │   ├── app-alert.json.j2
    │   │   ├── cpu-alert.json.j2
    │   │   ├── create-alerts-folder.json.j2
    │   │   ├── gitea.conf.j2
    │   │   ├── gitea_app.ini.j2
    │   │   ├── grafana.conf.j2
    │   │   ├── jenkins-job.xml.j2
    │   │   ├── jenkins-pipeline.groovy.j2
    │   │   ├── jenkins.conf.j2
    │   │   ├── logrotate.conf.j2
    │   │   ├── loki-config.yml.j2
    │   │   ├── loki.conf.j2
    │   │   ├── mysql.conf.j2
    │   │   ├── nginx-exporter.conf.j2
    │   │   ├── nginx-site.conf.j2
    │   │   ├── nginx.conf.j2
    │   │   ├── node-exporter.conf.j2
    │   │   ├── prometheus.conf.j2
    │   │   ├── prometheus.yml.j2
    │   │   ├── promtail-config.yml.j2
    │   │   ├── promtail.conf.j2
    │   │   ├── redis.conf.j2
    │   │   └── supervisord_ui.conf.j2
    │   └── vars
    │       └── main.yml
    ├── instances
    │   └── tasks
    │       ├── app-server.yml
    │       ├── cache-server.yml
    │       ├── ci-cd-server.yml
    │       ├── db-server.yml
    │       ├── log-server.yml
    │       ├── main.yml
    │       └── monitoring-server.yml
    ├── load-balance
    │   ├── tasks
    │   │   ├── create-instance-template.yml
    │   │   ├── create-mig.yml
    │   │   ├── enable-autoscaling.yml
    │   │   ├── get-internal-ip.yml
    │   │   ├── main.yml
    │   │   ├── set-nfs-in-master.yml
    │   │   └── setup-load-balancer.yml
    │   ├── templates
    │   │   └── startup-script.sh.j2
    │   └── vars
    │       └── main.yml
    ├── network
    │   └── tasks
    │       ├── create-subnet.yml
    │       ├── create-vpc.yml
    │       └── main.yml
    └── setup-env
        ├── tasks
        │   └── main.yml
        └── vars
            └── main.yml
```

</br>
</br>

<a name="servers"></a>
# 4. Server Roles, Software, and Descriptions
</br>

| Server Name         | Role                   | Software/Services Installed                                   | Description                                                       |
|---------------------|------------------------|----------------------------------------------------------------|-------------------------------------------------------------------|
| `app-server`        | Web server             | Nginx, PHP, WordPress (2 sites),Supervisord, Node Exporter, Promtail      | Hosts two WordPress websites served via Nginx and PHP             |
| `db-server`         | Database server        | MySQL, Supervisord, Node Exporter, Promtail                                             | Stores WordPress data and user content                            |
| `cache-server`      | Caching server         | Redis, Supervisord, Node Exporter, Promtail                                             | Provides in-memory object caching for WordPress                   |
| `ci-cd-server`      | CI/CD & Git hosting    | Jenkins, Gitea, Semgrep CLI, Supervisord, Node Exporter, Promtail                 | Runs pipelines and hosts Git repositories                         |
| `monitoring-server` | Monitoring             | Prometheus, Supervisord,, Node Exporter, Promtail                         | Collects system metrics and exposes them to Grafana               |
| `log-server`        | Centralized logging    | Grafana, Supervisord, Node Exporter, Promtail                           | Collects, stores, and visualizes logs using Loki and Grafana      |

</br>
</br>

<a name="playbook-descriptions"></a>
# 5. Playbook Descriptions

## 5.1  main.yml

### 5.1.1 Variables

Startup variables that will be used in many places are defined here, you have to set them according to your preferences before you start:

```yaml
app_server_user: "your-app-server-username"
e_mail: "your-email"
prefered_venv_path: "{{ current_path }}"
gcp_project: "your-project-name"
gcp_credentials_file: "your-gcp-credential-json-path"
prefix: "your-prefered-prefix"
gcp_region: "europe-west3"
gcp_zone: "europe-west3-a"
network_name: "{{ prefix }}-vpc"
network_auto_create_subnetworks: false
subnet_name: "{{ prefix }}-subnet"
```


### 5.1.2 Roles
```yaml
  roles:
  - setup-env
  - network
  - firewall-rules
  - instances
  - installations
  - load-balance
#  - delete-all
```

####  `setup-env:`

> **Purpose**: Prepares the Ansible control environment on the local machine by installing dependencies, creating a virtual environment, authenticating with GCP, and ensuring all required tools are ready.

####  `network:`

> **Purpose**: Creates a custom Virtual Private Cloud (VPC) network and subnet in the Google Cloud Platform (GCP) environment.

####  `firewall-rules`

> **Purpose**: Manages firewall rules in the GCP environment to secure network traffic.

- **Deny all ingress traffic by default**  
  Applies a restrictive baseline by blocking all incoming connections to the network unless explicitly allowed.

- **Allow internal access within the VPC**  
  Enables free communication between all instances inside the same VPC network to allow services to interact seamlessly.

- **Allow access from the local machine**  
  Opens firewall rules to permit SSH and other necessary access only from the control machine’s public IP address, enhancing security by limiting external access.

####  `instances`

> **Purpose**: Creates and configures the virtual machine instances in the GCP environment.

####  `installations`

> **Purpose**: Installs and configures all required software, tools, and applications on the appropriate servers.

####  `load-balance`

> **Purpose**:  Ensures a scalable, synchronized, WordPress deployment by combining a master-driven file system, centralized database access, and automated load-balanced instance scaling.

####  `delete-all`

> **Purpose**: Cleans up and deletes all resources created by the playbook.

</br>
</br>

## 5.2 Additional vars Folders

There are var folders, **./roles/setup-env/vars**, **./roles/installations/vars** and **./roles/load-balance/vars** .

####  `./roles/setup-env/vars`
> These variables are used to configure the Python virtual environment and required packages for interacting with Google Cloud:

```yaml
venv_path: "{{ prefered_venv_path }}/.venvs/ansible-gcp"
pip_bin: "{{ venv_path }}/bin/pip"
python_bin: "{{ venv_path }}/bin/python"
pip_packages:
- google-auth
- google-auth-httplib2
- google-api-python-client
- requests
- packaging
```


####  `./roles/installations/vars`

> This variable file defines all the configuration details and parameters required for installing and setting up the software components across the different servers in the infrastructure. It includes:
> 
> - The names of all server instances where software will be deployed, ensuring clear mapping between services and their hosts.
> - Configuration details for WordPress sites, such as site names, admin credentials, and database connection information, enabling automated setup and secure access.
> - User and directory paths, versioning, and administrative credentials for Gitea, facilitating the installation and configuration of the Git hosting service.
> - Jenkins administrative credentials and a list of essential plugins to automate the setup of the continuous integration/continuous deployment (CI/CD) environment.
> - Logrotate configurations for multiple services across the infrastructure, specifying log retention periods, user and group ownership, and targeted servers to ensure effective log management and system hygiene.
> 
> Overall, these variables provide a centralized and detailed configuration blueprint that drives the automated installation and configuration of all necessary software, services, and tools across the multi-server environment, supporting a robust, secure, and maintainable infrastructure.

```yaml
instance_names:
  - "{{prefix}}-app-server"
  - "{{prefix}}-ci-cd-server"
  - "{{prefix}}-db-server"
  - "{{prefix}}-log-server"
  - "{{prefix}}-monitoring-server"
  - "{{prefix}}-cache-server"


# wp name
wp1_name: "wp1"
wp2_name: "wp2"


# wp admin configs
wp1_admin_username: "admin1"
wp2_admin_username: "admin2"
wp1_admin_password: "StrongPass1!"
wp2_admin_password: "StrongPass2!"
wp1_admin_email: "{{ e_mail }}"
wp2_admin_email: "{{ e_mail }}"


# wp db configs
wp1_db_name: "wp1db"
wp2_db_name: "wp2db"
wp1_username: "wp1user"
wp2_username: "wp2user"
wp1_db_password: "password1"
wp2_db_password: "password2"
redis_password: "password"


# gitea configs
gitea_version: "1.23.6"  
gitea_user: gitea
gitea_home: /home/gitea
gitea_install_dir: /usr/local/bin
gitea_app_dir: /var/lib/gitea
gitea_custom_dir: /etc/gitea
gitea_log_dir: /var/log/gitea
gitea_admin_name: admin
gitea_admin_email: "{{ e_mail }}"
gitea_admin_password: "admin-pass"


# jenkins
jenkins_admin_user: admin
jenkins_admin_password: admin123
jenkins_admin_email: "{{ e_mail }}"
jenkins_plugin_list:
  - git
  - workflow-aggregator
  - pipeline
  - ssh-agent
  - credentials
  - scm-api
  - generic-webhook-trigger
  - ansicolor 
  - warnings-ng 
  - workspace-cleanup

# if you would like to use Semgrep App create token in app and paste it here.
semgrep_token: "27c27646afcf16651c80cf7320ceb418bbae4abc20ebfe974bf8b4661461c6ae"


# logrotate
logrotate_services:
  nginx:
    retention: 14
    user: root
    group: adm
    servers:
      - "{{ prefix }}-app-server"
  redis:
    retention: 7
    user: redis
    group: redis
    servers:
      - "{{ prefix }}-cache-server"
  mysql:
    retention: 7
    user: mysql
    group: mysql
    servers:
      - "{{ prefix }}-db-server"
  supervisor:
    retention: 7
    user: root
    group: root
    servers:
      - "{{ prefix }}-app-server"
      - "{{ prefix }}-cache-server"
      - "{{ prefix }}-db-server"
      - "{{ prefix }}-ci-cd-server"
      - "{{ prefix }}-monitoring-server"
      - "{{ prefix }}-log-server"
  node_exporter:
    retention: 7
    user: node_exporter
    group: node_exporter
    servers:
      - "{{ prefix }}-app-server"
      - "{{ prefix }}-cache-server"
      - "{{ prefix }}-db-server"
      - "{{ prefix }}-ci-cd-server"
      - "{{ prefix }}-monitoring-server"
      - "{{ prefix }}-log-server"
  promtail:
    retention: 7
    user: root
    group: root
    servers:
      - "{{ prefix }}-app-server"
      - "{{ prefix }}-cache-server"
      - "{{ prefix }}-db-server"
      - "{{ prefix }}-ci-cd-server"
      - "{{ prefix }}-monitoring-server"
      - "{{ prefix }}-log-server"
  gitea:
    retention: 30
    user: gitea
    group: gitea
    servers:
      - "{{ prefix }}-ci-cd-server"
  jenkins:
    retention: 30
    user: jenkins
    group: jenkins
    servers:
      - "{{ prefix }}-ci-cd-server"
  prometheus:
    retention: 15
    user: prometheus
    group: prometheus
    servers:
      - "{{ prefix }}-monitoring-server"
  grafana:
    retention: 15
    user: grafana
    group: grafana
    servers:
      - "{{ prefix }}-log-server"
  loki:
    retention: 15
    user: root
    group: root
    servers:
      - "{{ prefix }}-log-server"

```

####  `./roles/load-balance/vars`

> This variable file defines parameters for autoscaling and WordPress:

```yaml
# wp name
wp1_name: "wp1"


# autoscale
max_replicas: 3
min_replicas: 1
target_load_balancing_utilization: 0.6

```

</br>
</br>

## 5.3 Templates
There is a templates folder under **./roles/installations/** and **./roles/load-balance/** to be used for configurations.

####  `./roles/installations/templates` 

> The `templates` directory contains Jinja2 (`.j2`) template files used to generate configuration files, scripts, and pipeline definitions dynamically during the Ansible playbook execution. These templates serve as blueprints that are rendered with variable values specific to your environment and deployment requirements.
>
> Typical templates include configuration files for services like Nginx, MySQL, Jenkins, Grafana, and Gitea, as well as pipeline scripts (e.g., Jenkins Groovy pipelines) that automate CI/CD workflows. By using templates, the playbook can customize each deployed component according to the desired settings, such as IP addresses, credentials, alert thresholds, and resource paths, without manual editing.
>
> This approach promotes reusability, consistency, and flexibility across multiple environments, allowing you to maintain a single source of truth for configuration that adapts automatically based on input variables. Additionally, templating helps to reduce errors, streamline deployments, and simplify updates, since changes made in the template files propagate automatically to all affected configurations during subsequent runs.
>
> In summary, the `templates` folder is a critical part of the automation framework, enabling dynamic and scalable management of software, tools, and pipeline setups across your infrastructure.


####  `./roles/load-balance/templates` 
> startup-script.sh.j2 is the only template in this folder.
> Installs all required packages for running WordPress, including NGINX, PHP, MySQL client, and NFS utilities.
>
> Mounts the shared WordPress application directory from the master instance via NFS, ensuring all instances serve the same content.
>
>Removes default NGINX configurations and mounts the custom virtual host configuration directory from the master, keeping web server settings consistent across instances.
>
>Creates a symbolic link to activate the mounted NGINX site configuration and restarts NGINX to apply the changes.
>

</br>
</br>

<a name="usage"></a>
# 6. Usage
### 6.1 Add Your GCP Credentials

Change directory to ansible-project:
```bash
cd ansible-project
```

</br>

### 6.2 Add Your GCP Credentials

Before running the playbook, make sure you have your GCP service account key file (`.json`).

</br>

### 6.3 Edit Variables in `./main.yml`

In the `./main.yml` file, update the following variables to fit your environment:

| Variable                          | Description                                                |
|-----------------------------------|------------------------------------------------------------|
| `app_server_user`                 | The default SSH user on your app server                    |
| `e_mail`                          | Your email address for system notifications (optional)     |
| `prefered_venv_path`              | Python virtual environment path                            |
| `gcp_project`                     | Google Cloud project ID                                    |
| `gcp_credentials_file`           | Path to your GCP credentials JSON file                     |
| `prefix`                          | Prefix used to name all resources (instances, disks, etc.) |
| `gcp_region`                      | GCP region where resources will be created                 |
| `gcp_zone`                        | GCP zone for resource allocation                           |

>  These variables ensure all resources are consistently named and deployed to the correct region/zone.  


> Make sure your credentials file exists and your account has enough permission to create resources in GCP.

</br>

### 6.4 Run the playbook
Launch the full infrastructure setup with:
```bash
sudo ansible-playbook main.yml -v
```

</br>

### 6.5 Access the Web Interfaces

After provisioning, you can access the services using your browser or a GUI-enabled session:

| **Service**               | **URL**                                         | **Login Info**                                      |
|---------------------------|--------------------------------------------------|-----------------------------------------------------|
| WordPress Site 1          | http://wp1.example.com                           | No login required          |
| WordPress Site 2          | http://wp2.example.com                           | No login required          |
| WordPress Site 1 Admin    | http://wp1.example.com/wp-admin                  | Can be changed in  ./roles/installations/vars/main.yml wp1_admin_username and wp1_admin_password.</br>      `Username:` admin1 `Password:` StrongPass1!                |
| WordPress Site 2 Admin    | http://wp2.example.com/wp-admin                  | Can be changed in  ./roles/installations/vars/main.yml wp2_admin_username and wp2_admin_password.</br>      `Username:` admin2 `Password:` StrongPass2!                       |
| Gitea                     | http://`<ci-cd-server-external-ip>`:3000          |  Can be changed in  ./roles/installations/vars/main.yml gitea_admin_name and gitea_admin_password.</br>      `Username:` admin `Password:` admin-pass      |
| Jenkins                   | http://`<ci-cd-server-external-ip>`:8080          | Can be changed in  ./roles/installations/vars/main.yml jenkins_admin_user and jenkins_admin_password.</br>      `Username:` admin `Password:` admin123  |
| Prometheus                | http://`<monitoring-server-external-ip>`:9090     | No login required                       |
| Grafana                   | http://`<log-server-external-ip>`:3000            | `Username:` admin `Password:` admin     |
| Supervisord UI            | http://`<desired-server-external-ip>`:9001        | `Username:` admin `Password:` admin           |

> ℹ️ Replace `<...-external-ip>` with the actual external IP of the relevant server.

### To check external IPs:

```bash
gcloud compute instances list
```
### To open browser:

```bash
firefox
```
</br>

### 6.5 To Delete All
If you want to delte all resources created by the playbook, in `./main.yml` comment out the role delete-all and comment in the other roles:
```yml
  roles:
#  - setup-env
#  - network
#  - firewall-rules
#  - instances
#  - installations
#  - load-balance
  - delete-all
```

and run:
```bash
sudo ansible-playbook main.yml -v
```
