# Examination 9 - Use Ansible Vault for sensitive information

In the previous examination we set a password for the `webappuser`. To keep this password
in plain text in a playbook, or otherwise, is a huge security hole, especially
if we publish it to a public place like GitHub.

There is a way to keep sensitive information encrypted and unlocked at runtime with the
`ansible-vault` tool that comes with Ansible.

https://docs.ansible.com/ansible/latest/vault_guide/index.html

*IMPORTANT*: Keep a copy of the password for _unlocking_ the vault in plain text, so that
I can run the playbook without having to ask you for the password.

- Password for Vault is: q

# QUESTION A

Make a copy of the playbook from the previous examination, call it `09-mariadb-password.yml`
and modify it so that the task that sets the password is injected via an Ansible variable,
instead of as a plain text string in the playbook.

- This is how my playbook is looking:
---
- name: Install and start mariadb-server
  hosts: db
  become: true
  tasks:
    - name: Ensure MariaDB-server is installed.
      ansible.builtin.package:
        name: mariadb-server
        state: present

    - name: Ensure mariadb-server is enabled and started
      ansible.builtin.service:
        name: mariadb
        enabled: true
        state: started

    - name: Install PyMySQL so Ansible can talk to MariaDB
      ansible.builtin.package:
        name: python3-PyMySQL
        state: present

    - name: Create database instance
      community.mysql.mysql_db:
        name: webappdb
        state: present
        login_unix_socket: /var/lib/mysql/mysql.sock

    - name: Create database user
      community.mysql.mysql_user:
        name: webappuser
        password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          61356561333934623661333139656232616563613731323863376461623062363739636438306561
          6263373733393166343239303137373662636361356564320a366539613363653937376265306239
          31373330643866343032613131363938613034303965643465353133363936353866336263393265
          6432333632363836300a626433303663336263323135623465633938326137383136373838616561
          3731
        priv: "*.*:ALL"
        login_unix_socket: /var/lib/mysql/mysql.sock
        state: present

First I created a passwordfile.yml where I stored my password. Then I encrypted it with 'ansible-vault encrypt passwordfile.yml'. I chose a Vault password 'q'. I proceeded to cat passwordfile.yml which gave me my encrypted file. Lastly I replaced the string password with the Ansible Vault encrypted file password.


# QUESTION B

When the [QUESTION A](#question-a) is solved, use `ansible-vault` to store the password in encrypted
form, and make it possible to run the playbook as before, but with the password as an
Ansible Vault secret instead.

- I can refer to my previous answer: I first created a passwordfile.yml where I stored my password. Then I encrypted it with 'ansible-vault encrypt passwordfile.yml'. I chose a Vault password 'q'. Then you just run it with: 'ansible-playbook 09-mariadb-password.yml --ask-vault-pass', it will ask you for the password to your encrypted file you created before, type 'q'. If you want to run the playbook without it asking you for the Vault password then you need to store the Vault password in a separate file that you refer to when running the playbook. Looks like this: ansible-playbook 09-mariadb-password.yml --vault-password-file vault_pass.txt.
