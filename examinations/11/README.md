# Examination 11 - Loops

Imagine that on the web server(s), the IT department wants a number of users accounts set up:

    alovelace
    aturing
    edijkstra
    ghopper

These requirements are also requests:

* `alovelace` and `ghopper` should be added to the `wheel` group.
* `aturing` should be added to the `tape` group
* `edijkstra` should be added to the `tcpdump` group.
* `alovelace` should be added to the `audio` and `video` groups.
* `ghopper` should be in the `audio` group, but not in the `video` group.

Also, the IT department, for some unknown reason, wants to copy a number of '\*.md' files
to the 'deploy' user's home directory on the `db` machine(s).

I recommend you use two different playbooks for these two tasks. Prefix them both with `11-` to
make it easy to see which examination it belongs to.

# QUESTION A

Write a playbook that uses loops to add these users, and adds them to their respective groups.

When your playbook is run, one should be able to do this on the webserver:

    [deploy@webserver ~]$ groups alovelace
    alovelace : alovelace wheel video audio
    [deploy@webserver ~]$ groups aturing
    aturing : aturing tape
    [deploy@webserver ~]$ groups edijkstra
    edijkstra : edijkstra tcpdump
    [deploy@webserver ~]$ groups ghopper
    ghopper : ghopper wheel audio

There are multiple ways to accomplish this, but keep in mind _idempotency_ and _maintainability_.

---
- name: Create users and add them to their groups
  hosts: web
  become: true

  tasks:
    - name: Add users to their groups
      ansible.builtin.user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        append: true
        state: present
      loop:
        - {name: "alovelace", groups: "wheel,video,audio"}
        - {name: "aturing", groups: "tape"}
        - {name: "edijkstra", groups: "tcpdump"}
        - {name: "ghopper", groups: "wheel,audio"}

# QUESTION B

Write a playbook that uses

    with_fileglob: 'files/*.md5'

to copy all `\*.md` files in the `files/` directory to the `deploy` user's directory on the `db` server(s).

For now you can create empty files in the `files/` directory called anything as long as the suffix is `.md`:

    $ touch files/foo.md files/bar.md files/baz.md

--
- name: Copy all the '\*.md' files to 'deploy' user's directory
  hosts: db
  become: true
  tasks:
    - name: Copy each file over that matches the given pattern
      ansible.builtin.copy:
        src: "{{ item }}"
        dest: /home/deploy
        owner: deploy
        group: deploy
        mode: '0644'
      with_fileglob:
        - 'files/*.md'


# BONUS QUESTION

Add a password to each user added to the playbook that creates the users. Do not write passwords in plain
text in the playbook, but use the password hash, or encrypt the passwords using `ansible-vault`.

There are various utilities that can output hashed passwords, check the FAQ for some pointers.

---
- name: Create users and add them to their groups
  hosts: web
  become: true

  tasks:
    - name: Add users with their groups
      ansible.builtin.user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        append: true
        state: present
        password: "{{ item.password }}"
      loop:
        - {name: "alovelace", groups: "wheel,video,audio", password: "$6$rounds=656000$salt1$kT5FYBCPAO4ii3Yx8zFCM8gB3cLufVWPRBnXVDtPweZdkjyWr/px4J7FnVV7SAHbKOWeXZtTENrq7bdIG5tUr0"}
        - {name: "aturing", groups: "tape", password: "$6$rounds=656000$salt2$alTmmNCl9o1B7DtkO7IIFlSNjuiDNB17cm.xYk2A71uqVQ.2UJIJoLJrOPFCIliHX/xE2x2tJOEhAWvU8Mi4h/"}
        - {name: "edijkstra", groups: "tcpdump", password: "$6$rounds=656000$salt3$NFBVhdSb3bNdoIm8jVnwbMjEINOHWfeo5fJLjStLD0.xK.kfOKpCevjG17d2FpokXdZ83EvFfKfiSwkPH08F8."}
        - {name: "ghopper", groups: "wheel,audio", password: "$6$rounds=656000$salt4$yLL4idgjdVc4jyFrIuRrYJ8E4itY7tAsaUrwEO09bEMdAT0xhXvkl4Z6ghXKRxSNQMhS7W2VUXlwztrmJ3Lpx0"}

# BONUS BONUS QUESTION

Add the real names of the users we added earlier to the GECOS field of each account. Google is your friend.

# Resources and Documentation

* [loops](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html)
* [ansible.builtin.user](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)
* [ansible.builtin.fileglob](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/fileglob_lookup.html)
* https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-encrypted-passwords-for-the-user-module

