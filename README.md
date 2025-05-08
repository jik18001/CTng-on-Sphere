# Model 
The model file creates a star topology as shown in the figure below: 

![image](https://github.com/user-attachments/assets/ebc51907-91f9-44cf-89da-249f72210f03)

Every entity (including a control node) is connected to the backbone router via a 100Mbps access link. 


# Operations from the XDC
## 1. Generate Inventory

> **Note:** Replace `<USERNAME>`, `<PATH_TO_CTNGEXP>`, and `<SOURCE_HOST>` with your actual details.

> You will also need to modify the automation scripts for replication. 

1. Open a root shell in a new terminal and navigate to your CTngexp directory:

   ```bash
   cd <PATH_TO_CTNGEXP>
   sudo su          # become root
   ```
2. Generate the Ansible inventory file:

   ```bash
   python3 genini.py    # creates inv.ini
   ```
3. Switch back to your non-root user:

   ```bash
   exit               # leave root shell
   su <USERNAME>      # replace with your username
   ```

## 2. Distribute Playbooks
Distribute the CTngexp folder:
```bash
ansible-playbook -i inv.ini distribute.yml   # copy files from <SOURCE_HOST> to control node
```
From the CTngexp directory, run:

```bash

ansible-playbook -i inv.ini ssh.yml          # configure SSH keys/config
```
# Operations from the control Node

## 3. Configure Ansible

Create or update an ansible.cfg file in the project root with:

```ini
[defaults]
forks = 60
timeout = 60
retry_files_enabled = False
async_poll_interval = 5

[ssh_connection]
pipelining = True
```

> **Reminder:** If you modify control-node settings or delete this file, recreate it accordingly.

## 4. Install Prerequisites

```bash
sudo apt update
sudo apt install ansible
```

## 5. Bootstrap All Hosts

Run these playbooks to install prerequisites and Go on every host:

```bash
ansible-playbook -i inv.ini gpp.yml      # install general prerequisites
ansible-playbook -i inv.ini newgo.yml    # install Go runtime
```

Verify on any host:

```bash
ssh <HOSTNAME>
g++ --version
go version
```

## 6. Repository Setup & Distribution

1. **Initial checkout/setup**

   ```bash
   ansible-playbook -i inv.ini repo.yml
   ```
2. **Propagate code changes** (e.g., edits in gen\_test.go)

   ```bash
   ansible-playbook -i inv.ini redis.yml
   ```

## 7. Run the Experiment

From the control node in your CTngexp directory:

```bash
ansible-playbook -i inv.ini ctngv3.yml
```

## 8. Iterating on Experiments

1. Edit your test code (e.g., CTngV3/deter/gen\_test.go).
2. Redistribute and rerun:

   ```bash
   ansible-playbook -i inv.ini redis.yml
   go test ./CTngV3/deter
   ansible-playbook -i inv.ini ctngv3.yml
   ```

## 9. Collect Results

```bash
ansible-playbook -i inv.ini collect.yml
```

## 10. Retrieve Results & Cleanup

1. **Copy back to local machine:**

   ```bash
   ansible-playbook -i inv.ini copy.yml
   ```
2. **Clean up temporary files on control node:**

   ```bash
   rm -rv output output.tar.gz monitor_files
   ```
