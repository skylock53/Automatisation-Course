# Examination 7 - MariaDB installation

To make a dynamic web site, many use an SQL server to store the data for the web site.

[MariaDB](https://mariadb.org/) is an open-source relational SQL database that is good
to use for our purposes.

We can use a similar strategy as with the _nginx_ web server to install this
software onto the correct host(s). Create the playbook `07-mariadb.yml` with this content:

    ---
    - hosts: db
      become: true
      tasks:
        - name: Ensure MariaDB-server is installed.
          ansible.builtin.package:
            name: mariadb-server
            state: present

# QUESTION A

Make similar changes to this playbook that we did for the _nginx_ server, so that
the `mariadb` service starts automatically at boot, and is started when the playbook
is run.

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

Worked successfully.

# QUESTION B

When you have run the playbook above successfully, how can you verify that the `mariadb`
service is started and is running?

- You firstly either SSH in to your dbserver or log in to it directly through virt-manager. You can check the status of a service via systemd. You this by typing "sudo systemctl status <service>". In this case you type "mariadb" instead of "service". Our mariadb service is successfully running.

# BONUS QUESTION

How many different ways can use come up with to verify that the `mariadb` service is running?

- You can use the already mentioned command above, "sudo systemctl status <service>" for checking status via systemd. You can also "grep" for a process that is running. You do this through "ps aux | grep mariadb".
