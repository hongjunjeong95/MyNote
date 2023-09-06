# Ansible - Condition

```yaml
---
- name: Example
  hosts: all
  become: true
  vars:
    users:
      - name: john
        shell: /bin/bash
        enabled: true
      - name: alice
        shell: /bin/sh
        enabled: false
      - name: claud
        shell: /bin/bash
        enabled: true
      - name: henry
        shell: /bin/sh
        enabled: false
      - name: jeremy
        shell: /bin/bash
        enabled: true
      - name: may
        shell: /bin/sh
        enabled: false
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Create a user if enabled in Amazon Linux"
      user:
        name: "{{ item.name }}"
        shell: "{{ item.shell }}"
        comment: "FastCampus DevOps"
        state: "present"
      loop: "{{ users }}"
      # and : item.enabled 면서 동시에 ansible_facts["distribution"]이 Amazon일 때
      when: item.enabled and (ansible_facts["distribution"] == "Amazon")

    - name: "Show items between 10 and 100"
      debug:
        var: item
      loop: [0, 192, 154, 456, 7, 2, -1, 55, 234]
      # list는 and다. 여기서는 10 <= item <= 100일 때
      when:
        - item >= 10
        - item <= 100

    - name: "Show items not between 10 and 100"
      debug:
        var: item
      loop: [0, 192, 154, 456, 7, 2, -1, 55, 234]
      # item < 10 이거나 item > 100일 때
      when:
        - (item < 10) or (item > 100)

    - name: "Install Packages on Ubuntu"
      apt:
        name: "{{ item }}"
        update_cache: true
        state: "present"
      loop:
        - git
        - curl
        - htop
      # 자주 사용된다. os가 Ubuntu일 때 위 loop에 있는 패키지를 설치
      when:
        - ansible_facts['distribution'] == 'Ubuntu'

    - name: "Install Packages on Amazon Linux"
      yum:
        name: "{{ item }}"
        state: "present"
      loop:
        - git
        - curl
        - htop
      # 자주 사용된다. os가 Amazon일 때 위 loop에 있는 패키지를 설치
      when:
        - ansible_facts['distribution'] == 'Amazon'

			# cut -d 뒤에 오는 구분자(:)로 나눈다.
			# cut -f1 => 첫번째 필드만 가져온다.
			# cut -d: f1 /etc/passwd => /etc/passwd의 구분자는 (:)이므로 이것으로 자르고
			# 맨 앞에 username만 users register에 저장한다.
    - name: "Print users"
      command: "cut -d: -f1 /etc/passwd"
      register: users

    - name: "Is there claud"
      debug:
        msg: "There is no claud"
      # 위 register에 저장된 users에서 'claud'가 없으면 'There is no claud' 출력
      when: users.stdout.find('claud') == -1

```
