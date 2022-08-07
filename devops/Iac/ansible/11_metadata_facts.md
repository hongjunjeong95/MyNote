# Ansible - Metadata Facts

Ref : [Ansible Metadata Facts](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_metadata_facts_module.html)

## 예제

```yaml
---

- name: Gather EC2 Metadata Facts
  hosts: ubuntu
  become: true
  tasks:
    # Docs: https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_metadata_facts_module.html
    - name: "Gather EC2 Metadata Facts"
      amazon.aws.ec2_metadata_facts: {}

    # Docs: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: "Debug Ansible facts"
      debug:
        var: ansible_facts

    - name: "Set VPC CIDR"
      # set_fact를 이용하면 중간 단계에서 변수 정의 가능
      set_fact:
        vpc_cidr: "{{ (ansible_facts | dict2items | selectattr('key', 'match', '^ec2_network_interfaces_macs_.*_vpc_ipv4_cidr_block$') | map(attribute='value'))[0] }}"

    - name: "Debug VPC CIDR"
      debug:
        var: vpc_cidr

    - name: "Set VPC facts"
      set_fact:
        vpc_dns_server: '{{ vpc_cidr | ipaddr(2) | ipaddr("address") }}'
        vpc_network: '{{ vpc_cidr | ipaddr("network") }}'
        vpc_netmask: '{{ vpc_cidr | ipaddr("netmask") }}'

    - name: "Debug variables"
      debug:
        var: vars
```

## Modules Example

* [set_fact](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/set_fact_module.html#ansible-collections-ansible-builtin-set-fact-module) : 중간 단계에서 변수 정의 가능
* [ipaddr](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters_ipaddr.html)
  * ipaddr 같은 경우 `netaddr` 파이썬 패키지가 의존성으로서 설치되어 있어야 한다. 