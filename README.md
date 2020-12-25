# sbog/ansible_ca

[![Build Status](https://travis-ci.com/sorrowless/ansible_ansible_ca.svg?branch=master)](https://travis-ci.com/sorrowless/ansible_ansible_ca)
[![Ansible Role](https://img.shields.io/ansible/role/42562)](https://galaxy.ansible.com/sorrowless/ansible_ca)
[![Ansible Quality Score](https://img.shields.io/ansible/quality/42562)](https://galaxy.ansible.com/sorrowless/ansible_ca)
[![Ansible Role](https://img.shields.io/ansible/role/d/42562)](https://galaxy.ansible.com/sorrowless/ansible_ca)
[![GitHub](https://img.shields.io/github/license/sorrowless/ansible_ansible_ca)](https://github.com/sorrowless/ansible_ansible_ca/blob/master/LICENSE)

An Ansible role which installs and configures [CA](https://jamielinux.com/docs/openssl-certificate-authority/index.html) on Linux

## Requirements

Ansible 2.7+

## Role Variables

You can see all vars in `defaults/main.yml` vars file.

## Dependencies

None

## Example Playbook

```yaml
- name: Ensure CA
  hosts: ansible_cas
  remote_user: root

  roles:
    - ansible_ca
```

## License

Apache 2.0

## Author Information

This role was created by [Stan Bogatkin](https://sbog.ru).
