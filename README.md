# 🚀 Deploying Galera High Availability (HA) Cluster

## 📌 Project Overview
This project sets up a **Galera High Availability (HA) Cluster** using MariaDB and the Galera plugin. The cluster ensures **synchronous multi-master replication**, providing high availability, data consistency, and fault tolerance for your database infrastructure.

## 🎯 Purpose
- **High Availability**: Eliminate single points of failure with a multi-node cluster.
- **Data Consistency**: Ensure all nodes are in sync with synchronous replication.
- **Automation**: Use Ansible to automate the setup and configuration of the cluster.
- **Scalability**: Easily add or remove nodes from the cluster.

## 🔧 Technologies Used
- **MariaDB**: The database server.
- **Galera Plugin**: Provides synchronous multi-master replication.
- **Ubuntu**: The operating system for the cluster nodes.
- **Ansible**: Automates the setup and configuration of the cluster.

---

## 🏗️ Architecture
The Galera Cluster consists of the following components:
1. **3 Nodes**: Each node runs MariaDB with the Galera plugin.
2. **Synchronous Replication**: All nodes are in sync, ensuring data consistency.
3. **High Availability**: If one node fails, the others continue to operate.

---

## 📌 Setup Instructions (Manual Method on Ubuntu)

### 1️⃣ Prerequisites
Ensure you have:
- **3 Ubuntu Servers**: Each with static IPs and SSH access.
- **Network Configuration**: All nodes must communicate over ports:
  - `3306` (MySQL)
  - `4567` (Galera communication)
  - `4568` (IST)
  - `4444` (SST)

#### Configure Firewall Rules
Open the necessary ports on all nodes:
```
sudo ufw enable
sudo ufw allow 3306/tcp && \
sudo ufw allow 4567/tcp && \
sudo ufw allow 4568/tcp && \
sudo ufw allow 4444/tcp
sudo ufw reload
sudo ufw status
```
### Updated System:
Ensure you have:
``` sudo apt update && sudo apt upgrade -y ```

### 2️⃣ Install MariaDB and Galera Plugin
Install MariaDB and the Galera plugin on all nodes:
``` sudo apt install mariadb-server galera-4 -y ```

### 3️⃣ Configure MariaDB for Galera
Edit the MariaDB configuration file on all nodes:
- Open the configuration file:
``` sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf ```
- Add the following configuration:
```
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="my_galera_cluster"
wsrep_cluster_address="gcomm://192.168.254.101,192.168.254.102,192.168.254.103"
# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="192.168.254.101"
wsrep_node_name="galera-node1"

```
- Replace node1_ip, node2_ip, and node3_ip with the actual IPs of your nodes.

- Set wsrep_node_name and wsrep_node_address to the respective node's hostname and IP.

### 4️⃣ Bootstrap the Cluster
On the First Node:
- Stop the MariaDB service:
  ```sudo systemctl stop mariadb ```
- Bootstrap the cluster:
``` sudo galera_new_cluster ```
On the Remaining Nodes:
 - Start the MariaDB service:
``` sudo systemctl start mariadb ```

### 5️⃣ Verify the Cluster
- Check the cluster size:
``` mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';" ```
Expected output: wsrep_cluster_size: 3.

- Check the cluster status:
``` mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_%';" ```
- Create table in node 1 and see if it created in node 3
  ``` sudo mysql -u root -p -e "CREATE DATABASE final_test;" ```
  ``` sudo mysql -u root -p -e "SHOW DATABASES;" ```
--- 

## 📌 Setup Instructions (Automatic Method on Ubuntu)
- **structure of the Ansible project:**
```
📂 galera-ansible/
 ├── inventory**
 ├── ansible.cfg
 ├── galera_setup.yml
 ├── roles/
 │   ├── common/
 │   │   ├── tasks/main.yml
 │   │ 
 │   ├── firewall/
 │   │   ├── tasks/main.yml
 │   ├── mariadb/
 │   │   ├── tasks/main.yml
 │   │   ├── templates/galera.cnf.j2
 ├── README.md
```
- **After you make this file and apply 3 role, you can run the Playbook:**
```
ansible-playbook galera_setup.yml
```
- **Verify the Setup**
```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_%';"
 ```
---
## Publish to Ansible Galaxy (Optional)
### If you want to publish your roles to Ansible Galaxy, follow these steps:
- Create a meta/main.yml file in each role to define metadata (e.g., author, license, description).
- Push your roles to GitHub.
- Import to Ansible Galaxy:
- Go to Ansible Galaxy.
- Log in and click Add Content > Import Role from GitHub: Follow the prompts to import your roles.
