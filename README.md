![TravisCI build status](https://travis-ci.org/axmac/axmac.apache.svg?branch=master)

# axmac.apache

This is an Ansible role to install Apache 2.4 on Centos 6 and 7.

Originally forked from [jpnewman.ansible-role-apache](https://github.com/jpnewman/ansible-role-apache).

Uses [software collections](https://www.softwarecollections.org/en/scls/rhscl/httpd24/) to install Apache 2.4 on Centos 6.

Note: httpd service on Centos 6 is `httpd24-httpd`.

## Requirements

Ansible 2.x

## Role Variables

| Variable | Description | Default |
| -------- | ----------- | ------- |
| `apache_path`                         | | /etc/httpd             |
| `ssl_cert_directory`                  | | /etc/pki/tls/certs     |
| `apache_certificates.ca_cert_src:`    | | files/ca_cert.pem      |
| `apache_certificates.server_cert_src` | | files/server_cert.pem  |
| `apache_certificates.server_key_file` | | files/server_key.pem   |
| `apache_log_dir`                      | | /var/log/httpd24       |
| `server_name`                         | | {{ ansible_nodename }} |
| `apache_virtualhosts`                 | List of `vhost` objects | |
| `apache_mods`                         | List of `mod` objects | |

| `mod` Object   | Description |
| -------------- | ----------- |
| `mod_name`     | Name of the module, e.g. mime_module. |
| `mod_path`     | Optional: include if the module needs to be loaded. (I.e. is not in the base loaded set.) |
| `conf_file`    | Optional. Name of the destsination configuration file. |
| `conf_src`     | Optional. Path to a configuration source file. |
| `conf_content` | Optional. Raw content for the conf file. Only one of conf_src and conf_content should be provided. |

| `vhost` Object         | Description |
| ---------------------- | ----------- |
| `file_name`            | File name for the vhost |
| `listening_ip`         | Listening IP |
| `listening_port`       | Listening port |
| `server_name`          | Server Name |
| `document_root`        | Document Root |
| `SSLEngine`            | SSL Engine enabled (On / Off) |
| `SSLCACertificateFile` | SSL CA Certificate File |
| `SSLCertificateFile`   | SSL Certificate File |
| `SSLCertificateKeyFile`| SSL Certificate Key File |
| `extra_directives`     | Contains raw apache directives, one per line |
| `_directories`         | List of `_directory` Objects |

| `_directory` Object|Description|
| -|:--|
| `_root`|Directory location|

***NOTES***

- Any key under `apache_virtualhosts` or `_directories` that contains an underscore is a variable used in the role and / or templates. Otherwise it part of the apache directive.

***i.e.***

`SSLEngine: On` is transformed in to directive `SSLEngine On`. However `server_name: localhost` is processed by the role / template.

***e.g.*** 

    apache_virtualhosts:
      - file_name: default-vhost-not-secure.conf
        listening_ip: "*"
        listening_port: 80
        server_name: localhost
        document_root: /var/www
        extra_directives: |
          LogLevel info
          ErrorLog {{ apache_log_dir }}/localhost-error.log
          CustomLog {{ apache_log_dir }}/localhost-access.log combined env=!dontlog
    
          RewriteEngine On
          RewriteCond %{HTTPS} off
          RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
    
      - file_name: default-vhost-secure.conf
        listening_ip: "*"
        listening_port: 443
        server_name: localhost
        document_root: /var/www
        SSLEngine: "On"
        SSLCACertificateFile: "{{ apache_certificates_dest.ca_cert }}"
        SSLCertificateKeyFile: "{{ apache_certificates_dest.server_key }}"
        SSLCertificateFile: "{{ apache_certificates_dest.server_cert }}"
        _directories:
          - _root: /var/www
            Options: Indexes FollowSymLinks
            AllowOverride: None
            extra_directives: |
              Require all granted

For more examples see the following role files:

- `./defaults/main.yaml`
- `./tests/vars/main_alias.yaml`
- `./tests/vars/main_proxy.yaml`

## Dependencies

None.

## Example Playbook

    - hosts: servers
      roles:
         - { role: axmac.apache }

## Testing

### Dependencies

- Docker

### Setup

    $ cd tests
    $ pip install docker-py
    $ ansible-galaxy install -r requirements.yml

### Run

    $ ANSIBLE_ROLES_PATH=../../:galaxy/roles ansible-playbook main.yaml

## License

MIT

## Author Information

Alex McClennan
