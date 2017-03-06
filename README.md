Forked from jpnewman.apache to install Apache on Centos 6.

# axmac.apache

This is an Ansible role to install apache.

## Requirements

Ansible 2.x

## Role Variables

|Variable|Description|Default|
|---|---|:--|
|```apache_path```||/etc/httpd|
|```apache_default_delete_vhost```||true|
|```apache_vhosts_file```||"axmac.conf"|
|```ssl_cert_directory```||/etc/pki/tls/certs|
|```apache_certificates.ca_cert_src```||```"{{ ssl_cert_directory }}/ca_notsecure.crt"```|
|```apache_certificates.server_cert_src```||```"{{ ssl_cert_directory }}/notsecure.crt"```|
|```apache_certificates.server_key_file```||```"{{ ssl_cert_directory }}/notsecure.key"```|
|```apache_log_dir```||/var/log/apache2|
|```server_name```||localhost|
|```apache_virtualhosts```|List of ```vhost``` Objects||

|```vhost``` Object|Description|
|---|:--|
|```listening_ip```|listening IP|
|```listening_port```|listening port|
|```server_name```|Server Name|
|```document_root```|Document Root|
|```SSLEngine```|SSL Engine enabled (On / Off)|
|```SSLCertificateFile```|SSL Certificate File|
|```SSLCertificateKeyFile```|SSL Certificate Key File|
|```extra_directives```|Contains raw apache directives, one per line|
|```_directories```|List of ```_directory``` Objects|

|```_directory``` Object|Description|
|---|:--|
|```_root```|Directory location|

**NOTES: -**

- Any key under ```apache_virtualhosts``` or ```_directories``` that contains an underscore is a variable used in the role and / or templates. Otherwise it part of the apache directive.

***i.e.***

```SSLEngine: On``` is transformed in to directive ```SSLEngine On```. However ```server_name: localhost``` is processed by the role / template.

***e.g.*** ```apache_virtualhosts```

~~~
apache_virtualhosts:
  - listening_ip: "*"
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

  - listening_ip: "*"
    listening_port: 443
    server_name: localhost
    document_root: /var/www
    SSLEngine: "On"
    SSLCertificateFile: "{{ apache_certificates_dir.server_cert }}"
    SSLCertificateKeyFile: "{{ apache_certificates_dir.server_key }}"
    _directories:
      - _root: /var/www
        Options: Indexes FollowSymLinks
        AllowOverride: None
        extra_directives: |
          Require all granted
~~~

For more examples see the following role files: -

- ```./defaults/main.yml```
- ```./tests/templates/defaults/main_alias.yml```
- ```./tests/templates/defaults/main_proxy.yml```

## Dependencies

none

## Example Playbook

    - hosts: servers
      roles:
         - { role: axmac.apache, tags: ["apache"] }

## Testing

For more information on testing the template review readme ```./tests/templates/README.md```

## License

MIT / BSD

## Author Information

Alex McClennan
