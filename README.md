
---

# Ansible and Docker Setup for Multiple Ubuntu Servers

## Overview

This setup demonstrates how to use Ansible to configure multiple Ubuntu servers running inside Docker containers. We will:

1. Create and run 4 Docker containers with different ports.
2. Use Ansible to configure these servers by upgrading packages, installing utilities, and creating a test file.

---

## Prerequisites

Before proceeding, ensure you have the following installed:

- **Docker**: To run containers
- **Ansible**: To automate server configurations
- **SSH Key**: To enable Ansible to connect to Docker containers

---

## File Structure
![Structure](images/1.png)

```bash
.
├── Dockerfile           # Dockerfile to create Ubuntu-based containers
├── inventory.ini        # Ansible inventory file listing all servers
├── playbook1.yml        # Ansible playbook to configure servers
└── README.md            # This file
```

---

## Step-by-Step Guide

### 1. Create Docker Containers

Run 4 Docker containers with SSH enabled.

```bash
# Run server 1 on port 2201
docker run -d --rm -p 2201:22 --name server1 ubuntu-server

# Run server 2 on port 2202
docker run -d --rm -p 2202:22 --name server2 ubuntu-server

# Run server 3 on port 2203
docker run -d --rm -p 2203:22 --name server3 ubuntu-server

# Run server 4 on port 2204
docker run -d --rm -p 2204:22 --name server4 ubuntu-server
```

### 2. Generate SSH Key Pair

To allow Ansible to connect to the servers, generate an SSH key pair:
![keygen](images/2.png)
```bash
ssh-keygen -t rsa -b 4096
```

Press Enter to accept the default file locations, and choose an empty passphrase for easy automation.

### 3. Copy SSH Keys

Copy the public and private SSH keys to the necessary locations for Ansible:
![Copy](images/3.png)
```bash
cp ~/.ssh/id_rsa ~/.ssh/id_rsa.pub .
```

### 4. Create Ansible Inventory File
![inventory](images/4.png)
Create the `inventory.ini` file to list all the servers and specify how to connect to them.

```bash
echo "[servers]" > inventory.ini

for i in {1..4}; do
  docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' server${i} >> inventory.ini
done

cat << EOF >> inventory.ini

[servers:vars]
ansible_user=root
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
EOF
```

This file will list the IP addresses of all the running Docker containers.

### 5. Create Ansible Playbook

Create the `playbook1.yml` file to configure the servers. This playbook will:

- Update the apt cache and upgrade packages
- Install `vim`, `htop`, and `wget`
- Create a test file to verify the configuration
![playbook1](images/5.png)
```yaml
---
- name: Configure multiple servers
  hosts: servers
  become: yes

  tasks:
    - name: Update apt
      apt: update_cache=yes upgrade=dist

    - name: Install packages
      apt: name=["vim", "htop", "wget"] state=present

    - name: Create test file
      copy:
        dest: /root/test_file.txt
        content: "Configured by Ansible on {{ inventory_hostname }}"
```

### 6. Run the Ansible Playbook

Run the playbook to configure all 4 servers:
![runplaybook1](images/6.png)
```bash
ansible-playbook -i inventory.ini playbook1.yml
```

The output should indicate that the tasks were successfully executed on all servers:

```
PLAY RECAP *********************************************************************
server1                    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
server2                    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
server3                    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
server4                    : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### 7. Verify the Configuration

To check if the test file was created on all servers, run:
![testfile](images/7.png)
```bash
ansible all -i inventory.ini -m command -a "cat /root/test_file.txt"
```

This should display the message:

```bash
server1 | SUCCESS | rc=0 >>
Configured by Ansible on server1
```

Repeat for all servers (`server2`, `server3`, `server4`).

---

## Cleanup

If you want to stop all the containers after completing the configuration:
![cleanup](images/8.png)
```bash
for i in {1..4}; do docker stop server${i}; done
```

---

## Conclusion

This setup demonstrates how to use Ansible to configure multiple Docker containers running Ubuntu. You can extend this playbook to perform additional tasks, such as installing more software, setting up services like `nginx` or `mysql`, or automating deployments.

Feel free to modify the playbook and inventory file as per your requirements.

---

### Troubleshooting

1. **SSH Connection Issues**: Make sure that the SSH service is running inside the containers, and the correct ports are exposed.
2. **Ansible Command Not Found**: Ensure Ansible is properly installed on your system.

---
