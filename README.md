# aws_ec2_restart
## Youtube Channel: https://www.youtube.com/c/TamilTutEra
## Video Link: https://youtu.be/u3Wpz8jWBc4
## Tamiltutera - How to Restart AWS EC2 Instance using AWS CLI in Ansible Playbook

hosts
```
localhost
```

ansible.cfg
```
[defaults]
inventory = hosts
```

ansible-playbook main.yml -e "instance_id=<aws_instance_id> region=us-east-1"
```
---
- hosts: all
  gather_facts: false
  connection: local
  tasks:
    # checking the current state of the EC2 Instance
    - name: check current state of the EC2 Instance
      command: aws ec2 describe-instances --instance-ids {{ instance_id }} --region {{ region }} --output text --query 'Reservations[*].Instances[*].State.Name'
      register: pre_check_instance_state
      delegate_to: localhost
    - debug:
        msg: "{{ pre_check_instance_state.stdout }}"
    - debug:
        msg: "Instance is already running"
      when: pre_check_instance_state.stdout == "running"

    # AWS Instance Stopping
    - name: Stopping the EC2 Instance
      command: aws ec2 stop-instances --instance-ids {{ instance_id }} --region {{ region }}
      register: instance_stopped_state
      when: pre_check_instance_state.stdout != "stopped"
    - name: waiting to stopping the instance
      wait_for:
        timeout: 30
    # - debug:
    #     msg: "{{ instance_stopped_state}}"
    # - debug:
    #     msg: "{{ instance_stopped_state.stdout }}"
    - debug:
        msg: "Instance Stopped Successfully"
      when: instance_stopped_state.changed == true

    # checking the current state of the EC2 Instance after stopping
    - name: check current state of the EC2 Instance
      command: aws ec2 describe-instances --instance-ids {{ instance_id }} --region {{ region }} --output text --query 'Reservations[*].Instances[*].State.Name'
      register: check_instance_state
      delegate_to: localhost
    - debug:
        msg: "{{ check_instance_state.stdout }}"

    # AWS Instance Starting
    - name: starting the EC2 Instances
      command: aws ec2 start-instances --instance-ids {{ instance_id }} --region {{ region }}
      register: instance_started_state
      when: check_instance_state.stdout != "running"
      delegate_to: localhost
      
    - name: waiting to stopping the instance
      wait_for:
        timeout: 30
    

    # checking the current state of the EC2 Instance after starting
    - name: check current state of the EC2 Instance
      command: aws ec2 describe-instances --instance-ids {{ instance_id }} --region {{ region }} --output text --query 'Reservations[*].Instances[*].State.Name'
      register: post_check_instance_state
      delegate_to: localhost
    - debug:
        msg: "{{ post_check_instance_state.stdout }}"
    - fail:
        msg: "Failed to start the EC2 Instance"
      when: post_check_instance_state.stdout != "running"
    - debug:
        msg: "Instance is restart successfully"
      when: post_check_instance_state.stdout == "running"
      ```

