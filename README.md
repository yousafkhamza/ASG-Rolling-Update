# ASG Rolling Update (Ansible + Jenkins)
[![Builds](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

---
## Description

I just try to a new explanation method for easy to convey my work and details. Basically, this is an ansible-playbook with ASG Rolling update and Create an AWS Infrastructure with (ELB + ASG + Security Group). Its automated with Jenkins.

_Client Query_: I have an ELB (Elastic LoadBalancer) in amazon and that ELB under instances is registered from an ASG. Also, the developers are uploaded the site contents to the git, and the developers make updates on git (ELB git changes through user-data with git). So, that's very complicated each update time has to change the count of ASG but it's very annoying and expensive creates and removes instances unwanted is there any solution?

_Answer_: We have created a ASG oriented ansible playbook with dynamic inventory and its help to update git contents which if the current available instances and you can use this manually or automate via jenkins like (continues deployment) and who use the playbook it never needs to create instances unwanted.

---
## Features
- ASG Rolling updates through ansible-playbook (_Primary_)
- Includs ELB + ASG + Security group infrastructure on this ansible-playbook
- No need for hosts (Inventory file) for ASG under client servers. Because its work with Dynamic Inventory
- Furthermore, I have included a test website with user-data for more clarification
- Easy to handle and everyone can change the ASG (Count, Project_Name.. etc values)
- No need to install any dependencies like boto and boto3 (Please note that if you have using Ansible 2.2+ and python 2.7)
---
## Pre-Requests
- Install Ansible on your Master Machine
- Create an IAM user role under your AWS account and please enter the values once the playbook running time
##### Installation
[Ansible2](https://docs.ansible.com/ansible/2.3/index.html) (For your reference visit [How to install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html))
##### IAM Role Creation
[IAM Role Creation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)
##### Ansible Modules used
- [yum](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html) 
- [pip](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/pip_module.html)
- [ec2-key](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_key_module.html)
- [copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
- [ec2-group](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_group_module.html)
- [debug](https://www.google.com/search?q=debug+%2B+ansible&rlz=1C1ONGR_enIN928IN928&oq=debug+%2B+ansible&aqs=chrome..69i57.5092j0j4&sourceid=chrome&ie=UTF-8)
- [ec2_lc](https://docs.ansible.com/ansible/latest/collections/community/aws/ec2_lc_module.html)
- [ec2_elb_lb](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_elb_lb_module.html)
- [ec2_asg](https://docs.ansible.com/ansible/latest/collections/community/aws/ec2_asg_module.html)
- [ec2_instance_info](https://docs.ansible.com/ansible/latest/collections/community/aws/ec2_instance_info_module.html)
- [add_host](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/add_host_module.html)
- [git](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/git_module.html)
- [file](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)
- [pause](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/pause_module.html)
---
### How To Use
Ansible Installation article is in pre-request section so please check out the pre-request section.
```sh
amzon-linux-extras install -y ansible2
yum install git -y
git clone https://github.com/yousafkhamza/ASG-rolling-update.git
cd ASG-rolling-update

---Please-Change Your-Credentials---

ansible-playbook main.yml
```
---
## Architacture with Jenkins Automated

- Architacture

![alt text](https://i.ibb.co/TqsqVML/rolling-update.jpg)
---
## Behind the playbook
_I just explained the primary thing ASG Rolling update and Which variables I used so if you have any further doubts please look at the YAML file complete._
### ASG Rolling update (_Primary Code_)
```sh
# Dynmic Inventory Creation
    - name: "Task 07-Fetch ASG Created EC2 Details"
      ec2_instance_info:
        aws_access_key: "{{ access_key }}"
        aws_secret_key: "{{ secret_key }}"
        region: "{{ region }}"
        filters:
          "tag:aws:autoscaling:groupName": "{{ asg_status.auto_scaling_group_name }}"      <------- this is your ASG Name
          instance-state-name: [ "running"]
      register: asg_instances


    - name: "Task 08-Autoscale - Creating Dynamic Inventory Of Autoscaling EC2"
      add_host:                    <---------- Creating Dynmic Inventory
        name: "{{ item.public_ip_address }}"
        groups: "asg"
        ansible_host: "{{ item.public_ip_address }}"
        ansible_port: 22
        ansible_user: "ec2-user"
        ansible_ssh_private_key_file: ansible.pem
        ansible_ssh_common_args: "-o StrictHostKeyChecking=no"
      with_items:
        - "{{ asg_instances.instances }}"

# Play 02-Dyanamic Inventory Rolling Update. (Dynamic Inventory Play starts)
- name: "Task Dynamic Inventory Play 09-ASG Rolling Update Start"
  hosts: asg          <------------ host works with dynmic inventory (hosts)
  become: true
  serial: 1
  gather_facts: False
  vars_files:
    - instance.vars
  tasks:
```

### Used Variables:
- asg.vars
```sh
env: "ansible_project"  <---------- ProjectName
ami: "ami-0d5eff06f840b45e9" <---------- I used us-east-1 region so the AMI under us-east-1
key: "ansible" <-------------- Creating A keypair for entering ASG under instances
region: "us-east-1" <-------------- Region
sg_name: sgroup  <--------------- Security group were I used. 
type: "t2.micro" <---------------  Instance_type
count: "2" <----------------- ASG Count
```
- cred.vars
```sh
access_key: "<your-access-key>"     <------------------ Enter your IAM Access Key
secret_key: "<your-secret-key>"     <------------------ Enter your IAM Secret Key
```
---
## Conclusion 
_This playbook is used for ASG rolling update with a website contents from git without recreate instances. Furthermore, I tried to create a additional AWS infrastructure with a (ELB + ASG + Lauch Configuration + Security Group) through Ansible. So, you guys please reffer the playbook for creating infrastructure and dynamic inventory works with localhost inventory_


### ⚙️ Connect with Me

<p align="center">
<a href="mailto:yousaf.k.hamza@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/yousafkhamza"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a> 
<a href="https://www.instagram.com/yousafkhamza"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://wa.me/%2B917736720639?text=This%20message%20from%20GitHub."><img src="https://img.shields.io/badge/WhatsApp-25D366?style=for-the-badge&logo=whatsapp&logoColor=white"/></a>
