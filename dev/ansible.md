---
layout: page
title: Ansible
---

# Ansible

## installation

* on os x, use the `--ignore-install` option to pip install if you get `Operation no permitted`

## prereqs on target

### unix

* need to install `sudo` on target machine
    * user needs to be allowed to su (`visudo` on freebsd and allow wheel)

### windows

* run this powershell command on target machine

    set-executionpolicy -executionpolicy unrestricted

* run a script on the target machine
    * see <https://docs.ansible.com/ansible/intro_windows.html>
* be sure you can ping the target from the control
* if you get an error about not being able to delete a file, run this again:

    ConfigureRemotingForAnsible.ps1

## playbooks

* `become: true` in playbook for su instead of `-s` when running playbook
* `-K` to prompt for su password
* `-i` to specify inventory file

    ansible-playbook -i hosts/host -K book.yml

## misc

* see facts: `ansible all -m setup`
