# ASG Rolling Update (Ansible)
[![Builds](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

---
## Description

_Client Query_: I have an ELB (Elastic LoadBalancer) in amazon and that ELB under instances is registered from an ASG. Also, the developers are uploaded the site contents to the git, and the developers make updates on git (ELB git changes through user-data with git). So, that's very complicated each update time has to change the count of ASG but it's very annoying and expensive creates and removes instances unwanted is there any solution?

_Answer_: We have created a ASG oriented ansible playbook with dynamic inventory and its help to update git contents which if the current available instances and you can use this manually or automate via jenkins like (continues deployment) and who use the playbook it never needs to create instances unwanted.

---
## Feutures
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
Please insatall ansible first please check out the pre-request section.
```sh
ansible-playbook main.yml
```
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
```sh
env: "ansible_project"  <---------- ProjectName
ami: "ami-0d5eff06f840b45e9" <---------- I used us-east-1 region so the AMI under us-east-1
key: "ansible" <-------------- Creating A keypair for entering ASG under instances
region: "us-east-1" <-------------- Region
sg_name: sgroup  <--------------- Security group were I used. 
type: "t2.micro" <---------------  Instance_type
count: "2" <----------------- ASG Count
```
---
## Conclusion 
_This playbook is used for ASG rolling update with a website contents from git without recreate instances. Furthermore, I tried to create a additional AWS infrastructure with a (ELB + ASG + Lauch Configuration + Security Group) through Ansible. So, you guys please reffer the playbook for creating infrastructure and dynamic inventory works with localhost inventory_

_By

Yousaf K Hamza_

[LinkedIn](linkedin.com/in/yousaf-k-hamza-9274ba145)
