Ansible role: Etcd
=========

This role is used to install etcd cluster.

For now, it does the following:
- if separate disk path if defined, creates lvm volumes from it
- does all necessary OS preparations
- creates all necessary certificates
- configures and starts etcd cluster


Requirements
------------

This is not strict requirements and it may not work with other versions than tested ones.
Anyway. feel yourself free to test by yourself, suggest addition of new functionality and contribute.

Role is tested with:
- Ansible version >= 2.8.6
- CentOS version >= 7.6 (1803)

Currently supports only 'with_kubeadm' deployment type, it means that etcd cluster will be deployed via kubeadm, so kubeadm, kubelet and docker must be present and configured on the hosts, which can be done using 'kubernetes/prepare_cluster' and 'container_runtime/docker'


Role Variables
--------------

Variables and their descriptions copied from defaults/main.yml

```bash

# Name of separate device in /dev/disk/by-path/ for etcd storage:
etcd_disk_path: ""          # pci-0000:13:00.0-scsi-0:0:0:0

# Defines the way to deploy etcd, currently the only supported option is
# with kubeadm, which creates certificates and runs cluster inside containers:
etcd_deployment_type: with_kubeadm

# Variable which is common for most projects, used in 
# configuration files or file/directory names.
# By default used as reference for etcd_project_dir variable.
# Currently used as prefix for etcd initial cluster token:
etcd_project_name: test

# Variable which is common for most projects, used as
# project working directory on the localhost for the role.
# Currently is used for fetching etcd certificates
# to localhost and copying them among other cluster members:
etcd_project_dir: files/{{ etcd_project_name }}

# SANs which will be added to etcd server certificate:
etcd_server_cert_sans: []

# Initial cluster state for etcd cluster:
etcd_initial_cluster_state: new

# Initial cluster token to make sure etcd will contact
# members of it's cluster only in the network:
etcd_initial_cluster_token: "{{ etcd_project_name }}-etcd"

# Minimum version of TLS used:
etcd_tls_min_version: VersionTLS12

# Cipher suites used with TLS:
etcd_tls_cipher_suites:
- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
- TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
- TLS_RSA_WITH_AES_128_GCM_SHA256
- TLS_RSA_WITH_AES_256_GCM_SHA384

# Time in minutes to wait for cluster to start up before health check:
etcd_wait_before_healthcheck: 3

```

Variables and their descriptions copied from vars/with_kubeadm.yml

```bash

# Versions of kubeadm and kubelet used in role, usueally this must satisfy
# the version of kubernetes cluster, which will connect to etcd cluster:
etcd_k8s_version: 1.14.10

```

Dependencies
------------

If deployment type is 'with_kubeadm', then kubeadm, kubelet and docker must be installed and configuredon hosts.


Example Playbook
----------------

```bash
---
- hosts: localhost
  gather_facts: false
  become: no
  tasks:
  - name: Check ansible version >=2.8.6
    assert:
      msg: Ansible must be v2.8.6 or higher
      that:
      - ansible_version.string is version("2.8.6", ">=")
    tags:
    - check
  vars:
    ansible_connection: local

- hosts: all
  become: yes
  tasks:
  - import_role:
      name: kubernetes/prepare_cluster # if 'deployment_type: with_kubeadm' is set
  - import_role:
      name: container_runtime/docker # or container_runtime if 'container_runtime_name: docker' is set in variables
  - import_role:
      name: etcd

```

More detailed examples ( inventories, playbooks etc. ) of this and other roles can be found [here](https://github.com/caermeglaeddyv/examples/tree/dev/ansible).

It's highly recommended to start you test deploys from there, especially if you use Google Cloud Platform or VMware vCenter as your infrastructure, for now that [repository](https://github.com/caermeglaeddyv/examples) contains [Packer](https://github.com/caermeglaeddyv/examples/tree/dev/packer) and [Terraform](https://github.com/caermeglaeddyv/examples/tree/dev/terraform) examples to build templates and deploy machines on this platforms.


License
-------

[Apache 2.0](https://github.com/caermeglaeddyv/ansible-role-rear/blob/dev/LICENSE)


Author Information
------------------

Copyright 2020 [caermeglaeddyv](https://github.com/caermeglaeddyv)
