# Ansible role for Promtail in Linux/Unix

[![Ansible](https://img.shields.io/badge/ansible-6.4-EE0000?logo=ansible)](https://docs.ansible.com/) [![Molecule](https://img.shields.io/badge/molecule-v3.2-3CAFCE)](https://molecule.readthedocs.io/) [![License: Apache](https://img.shields.io/badge/License-Apache-yellow.svg)](https://github.com/universalvishwa/k3s-ansible/blob/master/LICENSE) [![Python](https://img.shields.io/badge/python-3.10-blue?logo=python)](https://www.python.org/downloads/release/python-379/)

Ansible playbooks to configure Promtail monitoring agent with Grafana Loki logging service in Linux OS platform.

## Overview
- This role is based off [ansible-role-promtail](https://github.com/patrickjahns/ansible-role-promtail).
    - Has a lot of similarities to that repo. But, wanted to offer some variations.
    - Can use Vagrant development environment to check the deployment _**OR**_ tune the Promtail config.
    - The Loki service can have `No Auth` or `Basic Auth` enabled.
    - Support configuring the Promtail agent to send metrics to _Prometheus_ monitoring service.

## Local Testing on a VM Server
- A Vagrant development environment can be used to easily setup the Server/VM needed to test the deployment.
    - Ref: [vagrant/ubuntu_22.04](https://github.com/universalvishwa/misc/tree/master/vagrant/ubuntu_22.04)

## Configuration options
### Basic configuration
- The basic configuration is fixed to collect all logs on `/var/log*.log` path.
- Use the advanced option if extra customizations needed.
- The user only need to provide the Loki URL in the following format,
    - With No Auth: `{{ protocol }}//{{ loki_host }}:{{ loki_port }}`
        - e.g. http://192.168.0.133:3100
    - Basic Auth: `{{ protocol }}//{{ loki_username }}:{{ loki_password }}@{{ loki_host }}:{{ loki_port }}`
        - e.g. http://lokiuser:lokip@ssword!@192.168.0.133:80
- The pass the Loki URL to the playbook role as follows. (e.g. Refer `playbook.yml`)
    ```yaml
    - name: Install and Configure promtail
      hosts: all
      become: yes
      roles:
        - role: promtail
          vars:
            loki_url: http://lokiuser:lokip@ssword!@192.168.0.133:80
    ```

### Advanced configuration
- Recommended to use this method to provide the best configurable options.
- Offer the ability to customization for `server, clients, positions, scrape_configs, target_config` configuration blocks.
- Set the respective configurations blocks as Ansible variables within the role as follows. (e.g. Refer `playbook_advanced.yml`)
    ```yaml
    - name: Install and Configure promtail
      hosts: all
      become: yes
      roles:
        - role: promtail
          vars:
            server:
              http_listen_port: 9080
              grpc_listen_port: 9095
            clients:
              url: "http://lokiuser:lokip@ssword!@192.168.0.133:80/loki/api/v1/push"
              external_labels:
                host: "{{ ansible_hostname }}"
            positions:
              filename: "/var/log/positions.yaml"
            scrape_configs:
              - job_name: system
                static_configs:
                  - targets:
                      - localhost
                    labels:
                      job: varlogs
                      __path__: /var/log/*.log
    ```

### Amazon EC2 instances
- Example config for AWS EC2 instances can be found on `playbook_aws_ec2.yml`.

## Running the playbook
#### Pre-requisites:
1. URL for Existing Loki logging service.
2. Credentials for Basic Auth if applicable.

### Running Ansible playbook with 
For this example, we are running against a Vagrant host.

1. Update the hosts.ini with the target IP address, port etc.
2. Verify Ansible connectivity
    ```bash
    ansible-playbook -i hosts.ini ping_vms.yml -l vagrant
    ansible-playbook -i hosts.ini ping_vms.yml
    ```
3. Run playbook against Vagrant host
    ```bash
    ansible-playbook -i hosts.ini playbook.yml
    ```

## Tech Debt
- Amazon Linux example playbook.
- Add Molecule testing + documentation.

## References
- [Gathering logs from a Linux host using Promtail](https://grafana.com/docs/grafana-cloud/quickstart/logs_promtail_linuxnode/)
- [Loki tutorial: How to set up Promtail on AWS EC2 to find and analyze your logs](https://grafana.com/blog/2020/07/13/loki-tutorial-how-to-set-up-promtail-on-aws-ec2-to-find-and-analyze-your-logs/)
- [Promtail Config : Getting Started with Promtail](https://www.chubbydeveloper.com/promtail-config/)
- [Install Promtail Binary and Start as a Service](https://sbcode.net/grafana/install-promtail-service/)
- [Configuring Promtail](https://grafana.com/docs/loki/latest/clients/promtail/configuration/)
- [ansible-role-promtail](https://github.com/patrickjahns/ansible-role-promtail)