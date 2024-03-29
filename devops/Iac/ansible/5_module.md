# Ansible - Module

## Collection

모듈의 집합이다.

* [ansible.builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html#plugins-in-ansible-builtin) 모듈이 앤서블을 잘 다루기 위한 모듈이다.

### Module Example

```yaml
---
- name: Example
  hosts: ubuntu
  become: true
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Create a user"
      user: "name=fastcampus shell=/bin/bash"

    - name: "Hello World"
      command: "echo 'Hello World!'"

    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html
    # linefile은 설정파일을 관리할 때 사용하는 모듈. 파일에 특정 라인이 있는지 보장해주는 모듈.
    # /etc/resolv.conf에 'nameserver 8.8.8.8' line이 있는지 확인 후 없으면 추가.
    - name: "Add DNS server to resolv.conf"
      lineinfile:
        path: /etc/resolv.conf
        line: "nameserver 8.8.8.8"

    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html
    - name: "Install Nginx"
      apt:
        name: nginx
        state: present
        update_cache: true

    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/posix/synchronize_module.html
    - name: "Upload web directory"
      synchronize:
        src: files/html/
        dest: /var/www/html
        archive: true
        checksum: true
        recursive: true
        delete: true

    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
    - name: "Copy nginx configuration file"
      copy:
        src: files/default
        dest: /etc/nginx/sites-enabled/default
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: "0644"

    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
    - name: "Ensure nginx service started"
      service:
        name: nginx
        state: started
```
