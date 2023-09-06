# Ansible - Facts

Ref : [Ansible Facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html)

`setup` 모듈을 통해서 ansible_facts를 수집할 수 있다.

## 특징

* Ansible facts는 원격 시스템, 운영체제, ip 주소, 파일시스템 등의 정보를 포함한다.
* `ansible_facts`로 이러한 정보에 접근할 수 있다.
* Playbook에서 `gather_facts : false` 는  Facts를 수집하지 않는다.
  * 대규모 시스템 관리 시 성능 향상을 위해서 사용한다. 대규모 시스템에서는 facts를 수집하는 것이 상당한 시간이 걸리기 때문이다.
  * 실험적인 환경에서 앤서블 사용 준비를 할 때
    * 도커 컨테이너에서 앤서블을 실행할 때 파이썬이 설치되어 있지 않아서 앤서블이 작동하지 않는다. 이 때 `gather_facts : false`를 하고 파이썬을 설치하면 prepare 용도로 사용할 수 있다. 그래서 파이썬을 사용하지 않는 모듈로 파이썬을 설치해야 한다.`yum` 이나 `apt` 대신  `command`, `shell`, `raw` 같은 모듈은 파이썬을 사용하지 않기 때문에 prepare 단계에서 파이썬을 설치할 수 있다.

## 예제

```yaml
---

- name: Prepare
  hosts: all
  become: true
  gather_facts: false
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Install python on Amazon Linux"
      yum:
        name: "python3"
        state: "present"
      when:
      - ansible_facts['distribution'] == 'Amazon'

		- name: "Install python on Ubuntu"
      apt:
        name: "python3"
        state: "present"
        update_cache: true
      when:
      - ansible_facts['distribution'] == 'Ubuntu'
```

`gather_facts : false`이기 때문에 `ansible_facts`를 사용할 수 없어서 `Error`가 날 것이다.

```yaml
---

- name: Prepare Amazon Linux
  hosts: amazon
  become: true
  gather_facts: false
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Install python on Amazon Linux"
      yum:
        name: "python3"
        state: "present"

- name: Prepare Ubuntu
  hosts: ubuntu
  become: true
  gather_facts: false
  tasks:
    - name: "Install python on Ubuntu"
      apt:
        name: "python3"
        state: "present"
        update_cache: true

- name: Debug
  hosts: all
  become: true
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Debug Ansible facts"
      debug:
        var: ansible_facts
```

그래서 host 별로 따로 yum과 apt를 사용해야 한다. 하지만 `gather_facts : false`를 사용할 때는 도커 컨테이너를 이용할 때이다. 이 때는 파이썬이 설치되어 있지 않기 때문에 yum과 apt 모듈을 사용할 수 없다.

```yaml
---

- name: Prepare Amazon Linux
  hosts: amazon
  become: true
  gather_facts: false
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Install python on Amazon Linux"
      command:
        name: "python3"
        state: "present"

- name: Prepare Ubuntu
  hosts: ubuntu
  become: true
  gather_facts: false
  tasks:
    - name: "Install python on Ubuntu"
      command:
        name: "python3"
        state: "present"
        update_cache: true

- name: Debug
  hosts: all
  become: true
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Debug Ansible facts"
      debug:
        var: ansible_facts
```

이 때는 command 모듈을 사용해서 파이썬을 설치해야 한다.

## AddHoc으로 facts 수집하기

```shell
ansible localhost -m setup
```

setup 모듈을 이용해서 facts를 수집할 수 있다.

```shell
ansible localhost -m setup -a "filter=ansible_distiribution*"
```

filter 파라미터를 이용해서 원하는 facts만 수집할 수 있다.