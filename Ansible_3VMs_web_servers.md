# Ansible - 3 VM's web servers
For this exercise, we will need:

- Oracle VM VirtualBox installed
- Any version of a Linux server (for example, AlmaLinux 9.5)
- A terminal capable of establishing an SSH connection (for example, PowerShell)

1. VM Preparation:
 - Open the VM’s Settings.
 - Go to Network.
 - Under Attached to, select Bridged Adapter.

![Image](https://github.com/user-attachments/assets/59365b7b-fb16-48c7-bcad-865a05a1b0eb)
 
 - Clone the original VM in which you installed the Linux OS and name it "Ansible Host."
 - Be sure to select the option to generate new MAC addresses.
 - Repeat this process for Host01, Host02, and Host03.

![Image](https://github.com/user-attachments/assets/f4c54ee4-1927-47ae-9558-334c47e512f6)

- You can group all four VMs and take snapshots. If something goes wrong, it will be easy to roll back your actions.
- The installation process will be skipped.
- 
SSH connection and editing some configuration files:
- Run the VM.
- To be able to connect to the VMs, we need to check their IP addresses. Open each VM window one by one, log in, and type `ip a`. Write down their IP addresses, and note which VM each one belongs to. 

![Image](https://github.com/user-attachments/assets/87d11c4f-a82e-4aeb-b91e-ff5a6f3abc45)

- Open four PowerShell windows and type the following command

```bash
ssh username@ip_that_you_write
```

![Image](https://github.com/user-attachments/assets/6b728742-bbda-413f-a385-3ef3834629fa)

- To change the hostname, type:
```bash
sudo hostnamectl set-hostname ansible
```
- To apply the settings, type:
 ```bash
exit
```
Then log back in.
- To add hostnames, type:
 ```bash
sudo vi /etc/hosts
```
It should look like this:

![Image](https://github.com/user-attachments/assets/9b9b653b-0350-43ed-96cb-e1201d1e8e3b)

You can copy this section, as it will be the same for every host. We can check if everything is working by executing:
```bash
ping host01
```
For example:

![Image](https://github.com/user-attachments/assets/25f0dba6-c47f-47b4-94f8-4e11b1ac5769)

- Now we need to repeat the steps for changing the hostname and adding IP addresses to the `/etc/hosts` file.
 
Preparation for work with ansible:

- We need to create a user on Host01-03. This user will allow Ansible to connect to these hosts and perform the required playbook actions.
Cretate user:
 ```bash
sudo adduser ansible
```
Set the user password: 
```bash
sudo passwd ansible
```
Add the user to the wheel group (this will allow it to perform required system actions):
 ```bash
 sudo usermod -aG wheel ansible
``` 
- Repeat these steps on all hosts.
- On the ansible system, generate ssh keys so that you can have passwordless communications with each host:
```bash
ssh-keygen
```
- Press Enter to use the default settings.
- Copy the public key to all hosts:
```bash
for i in 1 2 3; do ssh-copy-id hostname@host0$i; done
```

Ansible instalation and configuration:

- Install epel-release repository:
```bash
sudo dnf install epel-release y
```
- Install ansible:
```bash
sudo dnf install -y ansible
```
- Make a dir for storing playbooks:
```bash
mkdir Ansible_Playbooks
```
- Go to the created directory:
```bash
cd Ansible_Playbooks/
```
- Create nginx yaml file (playbook itself):
 ```bash
touch Nginx_Playbook.yaml
```
- Create a playbook, paste the folowing text:
```yaml
---
- name: Install and Start Nginx on AlmaLinux 9.5
  hosts: webservers
  become: yes
  tasks:
    - name: Ensure DNF is up to date
      dnf:
        name: "*"
        state: latest

    - name: Install Nginx
      dnf:
        name: nginx
        state: present

    - name: Enable and Start Nginx
      systemd:
        name: nginx
        enabled: yes
        state: started

    - name: Open HTTP and HTTPS Ports in Firewalld
      firewalld:
        service: "{{ item }}"
        permanent: yes
        state: enabled
      with_items:
        - http
        - https

    - name: Reload Firewalld
      command: firewall-cmd --reload

    - name: Verify Nginx is Running
      command: systemctl is-active nginx
      register: nginx_status

    - name: Print Nginx Status
      debug:
        msg: "Nginx is {{ nginx_status.stdout }}"
```
- Open the file /etc/ansible/hosts and append the folowing:
```
[webservers]
host01 ansible_host=IP_of_host01
host02 ansible_host=IP_of_host02
host03 ansible_host=IP_of_host03

[webservers:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/id_rsa

```
- To run you task type the folowing:
```bash
ansible-playbook path_to_your_playbook_yaml_file -K
```

Voilà! We have automated the web server installation. We can download preconfigured Ansible collections from the official Ansible website.
