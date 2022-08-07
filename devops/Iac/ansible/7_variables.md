# Ansible - Variables

## Invalid name

* Ansible은 파이썬으로 동작하다보니 파이썬의 지시어(python keywords)를 사용할 수 없다. such as `async` and `lambda`
* Playbook keywords such as `environment`
* `foo-port`, `foo port`, `foo.port`
* `5foo`, `12`

## Referencing simple variables

**Jinja2 syntax** : {{}}로 변수 사용

* Python에서 많이 사용하는 템플릿 언어

```yaml
ansible.builtin.template:
  src: foo.cfg.j2
  dest: '{{ remote_install_path }}/foo.cfg'
```



## Vars Usage

### Hard coding

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  tasks:
     Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
   	 - name: "Create a user"
       user:
         name: "lewis"
         comment: "hard coding"
         shell: "/bin/bash"
         uid: "7777"
```

### Vars

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  vars:
    user_name: "lewis"
    user_comment: "from playbook vars"
    user_shell: /bin/bash
    user_uid: "7777"
  tasks:
    - name: "Create a user"
      user:
        name: "{{ user_name }}"
        comment: "{{ user_comment }}"
        shell: "{{ user_shell }}"
        uid: "{{ user_uid }}"
```

### Vars file

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  vars_files:
  - vars.yaml
  tasks:
    - name: "Create a user"
      user:
        name: "{{ user_name }}"
        comment: "{{ user_comment }}"
        shell: "{{ user_shell }}"
        uid: "{{ user_uid }}"
```

#### vars.yaml

```yaml
user_name: "hongjun"
user_comment: "from vars.yaml file"
user_shell: "/bin/bash"
user_uid: "7777"
```



### Inventory vars

#### Inventory

```
[amazon]
amazon1 ansible_host=3.35.53.163 ansible_user=ec2-user
amazon2 ansible_host=3.34.253.165 ansible_user=ec2-user

[ubuntu]
ubuntu1 ansible_host=ec2-3-34-49-77.ap-northeast-2.compute.amazonaws.com ansible_user=ubuntu user_name=hongjun user_comment="from inventory" user_shell=/bin/bash user_uid=7777
ubuntu2 ansible_host=ec2-13-209-20-108.ap-northeast-2.compute.amazonaws.com ansible_user=ubuntu user_name=hongjun user_comment="from inventory" user_shell=/bin/bash user_uid=7777

[linux:children]
amazon
ubuntu
```

#### example.yaml

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  tasks:
    - name: "Create a user"
      user:
        name: "{{ user_name }}"
        comment: "{{ user_comment }}"
        shell: "{{ user_shell }}"
        uid: "{{ user_uid }}"
```



## Command

Command Line의 우선순위가 가장 높다.

#### example.yaml

```yaml
---

- name: Example
  hosts: ubuntu
  become: true
  vars:
    user_name: "hongjun"
    user_comment: "from playbook vars"
    user_shell: /bin/bash
    user_uid: "7777"
  vars_files:
  - vars.yaml
  tasks:
    - name: "Create a user"
      user:
        name: "{{ user_name }}"
        comment: "{{ user_comment }}"
        shell: "{{ user_shell }}"
        uid: "{{ user_uid }}"

```

```shell
ansible-playbook -i default.inv playbook.yaml -e "user_name=hongjun user_comment=hello"
```

```shell
ansible-playbook -i default.inv playbook.yaml -e "@vars.yaml" # @<파일경로>
```
