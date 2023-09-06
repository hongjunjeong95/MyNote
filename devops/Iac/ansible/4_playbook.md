# Ansible - Playbook

## 앤서블을 사용하는 방법

* Ansible playbook : yaml 파일을 이용해서 서버 관리
* Ansible-pull
* ad-hoc command : 명령어라서 간단하지만 재사용하기 어렵다. 그래서 테스트 목적으로 사용된다.

## Commands

### Install nginx

```shell
ansible-playbook -i inventory install-nginx.yaml
```

* `ansible-playbook install-nginx.yaml` : `install-nginx.yaml` playbook을 실행
* Ansible의 장점 중 하나가 멱등성이다. 위 명령으로 nginx를 설치했다면 다시 실행해도 아무런 변화가 없을 것이다.

### Install nginx

```yaml
---

- name: Install Nginx on Ubuntu
  hosts: ubuntu
  become: true
  tasks:
    - name: "Install Nginx"
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: "Ensure nginx service started"
      service:
        name: nginx
        state: started

- name: Install Nginx on Amazon Linux
  hosts: amazon
  become: true
  tasks:
    - name: "Enable Nginx repository provided by Amazon"
      command: "amazon-linux-extras enable nginx1"

    - name: "Install Nginx"
      yum:
        name: nginx
        state: present

    - name: "Ensure nginx service started"
      service:
        name: nginx
        state: started
```

```shell
ansible -i inventory amazon -m command -a "curl localhost"
ansible -i inventory ubunutu -m command -a "curl localhost"
```

 위 명령어로 nginx가 잘 동작하는지 확인 가능

### Uninstall nginx

```yaml
---

- name: Uninstall Nginx on Ubuntu
  hosts: ubuntu
  become: true
  tasks:
    - name: "Ensure nginx service stopped"
      service:
        name: nginx
        state: stopped

    - name: "Uninstall Nginx"
      apt:
        name: nginx
        state: absent

- name: Uninstall Nginx on Amazon Linux
  hosts: amazon
  become: true
  tasks:
    - name: "Ensure nginx service stopped"
      service:
        name: nginx
        state: stopped

    - name: "Uninstall Nginx"
      yum:
        name: nginx
        state: absent

    - name: "Disable Nginx repository provided by Amazon"
      command: "amazon-linux-extras disable nginx1"
```

```shell
ansible -i inventory amazon -m command -a "curl localhost"
ansible -i inventory ubunutu -m command -a "curl localhost"
```

위 명령어로 Nginx가 삭제되었는지 확인
