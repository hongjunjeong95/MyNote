# Ansible - Loop

Ref : [Ansible loop](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)

## Loop Usage

### with_items

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group_module.html
    - name: "Create groups"
      group:
        name: "{{ item }}"
        state: "present"
      with_items:
      - backend
      - frontend
      - devops
```

### loop

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Create a user"
      user:
        name: "{{ item }}"
        comment: "FastCampus DevOps"
        state: "present"
      loop:
      - john
      - alice
      - claud
      - henry
      - jeremy
      - may
```

### loop with variables

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  vars:
    users:
      - john
      - alice
      - claud
      - henry
      - jeremy
      - may
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Create a user"
      user:
        name: "{{ item }}"
        comment: "FastCampus DevOps"
        state: "present"
      loop: "{{ users }}"
```

### loop using {{ tags | dict2items }}

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  vars:
    tags:
      Name: "Debug"
      Environment: "Test"
      Owner: "posquit0"
  tasks:
    - name: "Debug data"
      debug:
        msg: "{{ item.key }}: {{ item.value }}"
      loop: "{{ tags | dict2items }}"
```

### loop with dictionary

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  vars:
    users:
      - name: john
        shell: /bin/bash
      - name: alice
        shell: /bin/sh
      - name: claud
        shell: /bin/bash
      - name: henry
        shell: /bin/sh
      - name: jeremy
        shell: /bin/bash
      - name: may
        shell: /bin/sh
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Create a user"
      user:
        name: "{{ item.name }}"
        shell: "{{ item.shell }}"
        comment: "FastCampus DevOps"
        state: "present"
      loop: "{{ users }}"
```
