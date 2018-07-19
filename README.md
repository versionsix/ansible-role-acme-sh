Acme.sh Role
=========

This ansible role is for installing [acme.sh](https://github.com/Neilpang/acme.sh) using ansible. This role only supports dns txt name verification.

Requirements
------------

This role has no software requirements, however it is required to own a domain and have access to a dns api.

Role Variables
--------------

Currently following variables are supported:

 - acme_sh_test: (bool) run in staging mode if set to true, default false
 - acme_sh_env: a dict of all environment Variables
 - command: a list of all commands to run
 - acme_sh_version: git version of acme.sh Set to master for current master

Dependencies
------------

None

Example Playbook
----------------

Make sure to add ` || true` to a command to prevent it from failing if the service isn't installed yet
```
  tasks:
    - name: Create let's encrypt certificates using acme.sh
      include_role:
        name: ansible-role-acme-sh
      vars:
        acme_sh_test: true
        acme_sh_env:
          DO_API_KEY: "myverysecretapikey"
        command:
          - "--dns dns_dgon --dnssleep 15 --issue --log -d 'machine-a.l0c4l.host'"
          - "--install-cert
              -d 'machine-a.l0c4l.host'
              --key-file /etc/ssl/private/machine-a.l0c4l.host.key
              --fullchain-file /etc/ssl/certs/machine-a.l0c4l.host.pem
              --reloadcmd \"service nginx force-reload  || true\""

    - name: Ensures /var/www/machine-a.l0c4l.host dir exists
      file: path=/var/www/machine-a.l0c4l.host state=

    - name: Set index page content
      copy:
        dest: /var/www/machine-a.l0c4l.host/index.html
        content: |
          <html>
            Hi there! </br>
            Welcome to machine-a.l0c4l.host
          </html>

    - name: Setup nginx
      include_role:
        name: ansible-role-nginx
      vars:
        nginx_vhosts:
          - listen: "443 ssl http2"
            server_name: "machine-a.l0c4l.host"
            root: "/var/www/machine-a.l0c4l.host"
            index: "index.php index.html index.htm"
            state: "present"
            template: "{{ nginx_vhost_template }}"
            filename: "machine-a.l0c4l.host.conf"
            extra_parameters: |
              ssl_certificate     /etc/ssl/certs/machine-a.l0c4l.host.pem;
              ssl_certificate_key /etc/ssl/private/machine-a.l0c4l.host.key;
              ssl_protocols       TLSv1.1 TLSv1.2;
              ssl_ciphers         HIGH:!aNULL:!MD5;

```
License
-------

BSD

Author Information
------------------

XenefiX - xenefix monkeytail protonmail com - https://xlw.be
