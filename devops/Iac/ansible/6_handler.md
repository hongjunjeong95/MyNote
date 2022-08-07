# Ansible - Handler

Task에서 이벤트를 발생시키고 해당 이벤트에 대해서 핸들러가 작동

### Handler Example

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
    - name: "Add DNS server to resolv.conf"
      lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver 8.8.8.8'

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
        mode: '0644'
      notify:
      - Restart Nginx

    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
    - name: "Ensure nginx service started"
      service:
        name: nginx
        state: started

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

위 코드에서 일부를 가져옴

```yaml
- name: "Copy nginx configuration file"
      copy:
        src: files/default
        dest: /etc/nginx/sites-enabled/default
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: '0644'
      notify:
      - Restart Nginx
- name: "Ensure nginx service started"
      service:
        name: nginx
        state: started
handlers:
			- name: Restart Nginx
				service:
					name: nginx
					state: retarted
```

`notify : Restart Nginx`로 이벤트를 날리고 `Restart Nginx` 이벤트를 수신하는 핸들러가 `service`를 작동함

## 핸들러 특징

* Play 내에서 같은 이벤트를 여러 번 호출하더라도 동일한 핸들러는 한 번만 실행된다.
* 모든 핸들러는 플레이 내에 모든 작업이 완료된 후에 실행된다.
* 핸들러는 이벤트 호출 순서에 따라 실행되는 것이 아니라 핸들러 정의 순서에 따라 실행된다.
