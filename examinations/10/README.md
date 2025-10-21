# Examination 10 - Templating

With the installation of the web server earlier in Examination 6, we set up
the `nginx` web server with a static configuration file that listened to all
interfaces on the (virtual) machine.

What we really want is for the webserver to _only_ listen to the external
interface, i.e. the interface with the IP address that we connect to the machine to.

Of course, we can statically enter the IP address into the file and upload it,
but if the IP address of the machine changes, we have to do it again, and if the
playbook is meant to be run against many different web servers, we have to be able
to do this manually.

Make a directory called `templates/` and put the `nginx` configuration file from Examination 6
into that directory, and call it `example.internal.conf.j2`.

If you look at the `nginx` documentation, note that you don't have to enable any IPv6 interfaces
on the web server. Stick to IPv4 for now.

# QUESTION A

Copy the finished playbook from Examination 6 (`06-web.yml`) and call it `10-web-template.yml`.

Make the static configuration file we used earlier into a Jinja template file,
and set the values for the `listen` parameters to include the external IP
address of the virtual machine itself.

Use the `ansible.builtin.template` module to accomplish this task.

- My finished playbook:

---
- name: Copy the local files/https.conf file to /etc/nginx/conf.d/https.conf
  hosts: web
  become: true
  vars:

    listen_address: "192.168.121.135"

  tasks:
    - name: Ensure nginx is installed
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Deploy templated nginx config for example.internal
      ansible.builtin.template:
        src: templates/example.internal.conf.j2
        dest: /etc/nginx/conf.d/example.internal.conf
        mode: '0644'
      notify: Reload nginx to apply new template conf

    - name: Create directory structure
      ansible.builtin.file:
        path: /var/www/example.internal/html/
        state: directory
        setype: httpd_sys_content_t
        mode: '0644'

    - name: Copy files/index.html to /var/www/example.internal/html/index.html
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/example.internal/html/index.html
        setype: httpd_sys_content_t
        mode: '0644'

    - name: Ensure nginx is started at boot
      ansible.builtin.service:
        name: nginx
        enabled: true
        state: started

  handlers:
    - name: Reload nginx to apply new template conf
      ansible.builtin.service:
        name: nginx
        state: reloaded

Main things I have added in this playbook: "Notify", "handlers", "vars" and "Deploy templated nginx ...." task. I have used notify and handlers because of unneccesary downtime. Handlers run last after the playbook has run, and the "notify" lets the "handlers" know if a change has happened in the templated config file. If a change is made then the handler will run, and in this case reload nginx to apply new template configuration. "Vars" works like a variable where you store something, in this case the external IP address for the webserver. "Listen address:" is refered to the "example.internal.conf.j2" file.

# Resources and Documentation

* https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html
* https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html
* https://nginx.org/en/docs/http/ngx_http_core_module.html#listen
