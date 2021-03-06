Ansible-openresty/nginx role
=========

Generic role to deploy install openresty/nginx servers. Extract configuration outside templates and make sure we can configure almost anything.


Role Variables
--------------

```
---

openresty_variant: openresty

openresty_repo_url: "{{ _openresty_repo_url }}"
openresty_repo_key: "{{ _openresty_repo_key }}"
openresty_server_pkg: "{{ _openresty_server_pkg }}"
openresty_service: "{{ _openresty_server_service }}"

openresty_packages:
  - wget
  - gnupg
  - ca-certificates
  - software-properties-common

openresty_confd_dir: "{{ openresty_conf_dir }}/conf.d"

openresty_pid_dir: "{{ _openresty_pid_dir }}"
openresty_pid_file: "{{ openresty_pid_dir }}/nginx.pid"

openresty_log_dir: /var/log/nginx
openresty_root_dir: "{{ _openresty_root_dir }}"


openresty_systemd_service_path: "{{ _openresty_systemd_service_path }}"
openresty_systemd_service: "{{ _openresty_systemd_service }}"

openresty_server_user: www-data
openresty_server_group: www-data

openresty_ssl_dhparam_size: 2048
openresty_ssl_dhparam_file: "{{ openresty_conf_dir }}/dh{{ openresty_ssl_dhparam_size }}.pem"

openresty_ssl_selfsigned_generate: true
openresty_ssl_selfsigned_name: default

openresty_ssl_files: {}
#   - certificate: "{{ openresty_ssl_cert_dir }}/ansible.com.crt"
#     certificate_value:
#     key: "{{ openresty_ssl_key_dir }}/ansible.com.key"
#     key_value:
#   - certificate: "{{ openresty_ssl_cert_dir }}/test2.com.crt"
#     certificate_value:  "{{ ssl_test2_cert }}"
#     key: "{{ openresty_ssl_key_dir }}/test2.com.key"
#     key_value: "{{ ssl_test2.com_key }}"
openresty_main_template:
  global_conf:
    user: "{{ openresty_server_user }}"
    worker_processes: auto
    #worker_rlimit_nofile: 1024
    error_log: "{{ openresty_log_dir }}/error.log debug"
    pid: "{{ openresty_pid_file }}"
  event_conf:
    worker_connections: 1024
    multi_accept: 'on'
  http_conf:
    include: "{{ openresty_conf_dir }}/mime.types"
    default_type:  application/octet-stream
    keepalive_timeout: 65
    access_log: 'off'
    log_format:  main  '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"'

    sendfile: 'on'
    tcp_nopush: 'on'
    tcp_nodelay: 'on'

    server_tokens: 'off'
    client_max_body_size: 20m
    client_body_buffer_size: 128k
    index: index.php index.htm index.html

    gzip: 'on'
    gzip_disable: "MSIE [1-6].(?!.*SV1)"
    gzip_vary: 'on'
    gzip_proxied: any
    gzip_comp_level: 6
    gzip_buffers: 16 8k
    gzip_http_version: 1.1
    gzip_types: text/plain text/css text/javascript text/xml application/json application/x-javascript application/xml application/xml+rss application/octet-stream

    ssl_protocols: 'TLSv1.2 TLSv1.3'
    ssl_session_tickets: 'off'
    ssl_prefer_server_ciphers: 'on'
    ssl_ciphers: "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256"
    ssl_dhparam: "{{ openresty_ssl_dhparam_file }}"

openresty_params:
  - filename: fastcgi_params
    directive: fastcgi_param
    include: true
    vars:
      QUERY_STRING: $query_string
      REQUEST_METHOD: $request_method
      CONTENT_TYPE: $content_type
      CONTENT_LENGTH: $content_length

      SCRIPT_FILENAME: $request_filename
      SCRIPT_NAME: $fastcgi_script_name
      REQUEST_URI: $request_uri
      DOCUMENT_URI: $document_uri
      DOCUMENT_ROOT: $document_root
      SERVER_PROTOCOL: $server_protocol

      GATEWAY_INTERFACE: GI/1.1
      SERVER_SOFTWARE: nginx/$nginx_version

      REMOTE_ADDR: $remote_addr
      REMOTE_PORT: $remote_port
      SERVER_ADDR: $server_addr
      SERVER_PORT: $server_port
      SERVER_NAME: $server_name

      HTTPS: $https

      REDIRECT_STATUS: 200
  - filename: proxy_params
    directive: proxy_set_header
    include: false
    vars:
      Host: $host
      X-Real-IP: $remote_addr
      X-Forwarded-For: $proxy_add_x_forwarded_for
  - filename: uwsgi_params
    directive: uwsgi_param
    include: false
    vars:
      QUERY_STRING: $query_string
      REQUEST_METHOD: $request_method
      CONTENT_TYPE: $content_type
      CONTENT_LENGTH: $content_length

      REQUEST_URI:  $request_uri
      PATH_INFO:  $document_uri
      DOCUMENT_ROOT:  $document_root
      SERVER_PROTOCOL:  $server_protocol
      UWSGI_SCHEME: $scheme

      REMOTE_ADDR:  $remote_addr
      REMOTE_PORT:  $remote_port
      SERVER_PORT:  $server_port
      SERVER_NAME:  $server_name
  - filename: error_codes
    include: true
    data: |
      map $status $status_text {
        400 'Bad Request';
        401 'Unauthorized';
        402 'Payment Required';
        403 'Forbidden';
        404 'Not Found';
        405 'Method Not Allowed';
        406 'Not Acceptable';
        407 'Proxy Authentication Required';
        408 'Request Timeout';
        409 'Conflict';
        410 'Gone';
        411 'Length Required';
        412 'Precondition Failed';
        413 'Payload Too Large';
        414 'URI Too Long';
        415 'Unsupported Media Type';
        416 'Range Not Satisfiable';
        417 'Expectation Failed';
        418 'I\'m a teapot';
        421 'Misdirected Request';
        422 'Unprocessable Entity';
        423 'Locked';
        424 'Failed Dependency';
        426 'Upgrade Required';
        428 'Precondition Required';
        429 'Too Many Requests';
        431 'Request Header Fields Too Large';
        451 'Unavailable For Legal Reasons';
        500 'Internal Server Error';
        501 'Not Implemented';
        502 'Bad Gateway';
        503 'Service Unavailable';
        504 'Gateway Timeout';
        505 'HTTP Version Not Supported';
        506 'Variant Also Negotiates';
        507 'Insufficient Storage';
        508 'Loop Detected';
        510 'Not Extended';
        511 'Network Authentication Required';
        default 'Something is wrong';
      }

openresty_addtional_params: []

openresty_default_listen_options: "{{ _openrest_default_listen_options + openresty_additional_listen_options }}"
openresty_additional_listen_options: []

openresty_default_sites_conf:
  - server_name: _
    root_dir: "{{ openresty_root_dir }}"
    error_pages:
      - 400 401 402 403 404 405 406 407 408 409 410 411 412 413 414 415 416 417 418 421 422 423 424 426 428 429 431 451 /40x.html
      - 500 501 502 503 504 505 506 507 508 510 511 /50x.html
    servers:
      - listen:
          port: 80
          options: "{{ openresty_default_listen_options | join(' ') }}"
        locations:
          - path: ~ /nginx_status
            conf:
              - stub_status: 'on'
              - access_log: 'off'
              - allow: 127.0.0.1
              - allow: 10.0.0.251/32
              - allow: 10.0.0.252/32
              - allow: 10.0.0.25/32
              - deny: all
          - path: /
            conf:
              - return: 404
          - path: = /50x.html
            conf:
              - ssi: 'on'
              - internal:
              - root: "{{ openresty_root_dir }}/errors"
          - path: = /40x.html
            conf:
              - ssi: 'on'
              - internal:
              - root: "{{ openresty_root_dir }}/errors"
      - listen:
          port: 443
          options: "ssl {{ openresty_default_listen_options | join(' ') }}"
        ssl:
          certificate: "{{ openresty_ssl_cert_dir }}/{{ openresty_ssl_selfsigned_name }}.crt"
          key: "{{ openresty_ssl_key_dir }}/{{ openresty_ssl_selfsigned_name }}.pem"
        locations:
          - path: ~ /nginx_status
            conf:
              - stub_status: 'on'
              - access_log: 'off'
              - allow: 127.0.0.1
              - allow: 10.0.0.251/32
              - allow: 10.0.0.252/32
              - allow: 10.0.0.25/32
              - deny: all
          - path: /
            conf:
              - return: 404
          - path: = /50x.html
            conf:
              - ssi: 'on'
              - internal:
              - root: "{{ openresty_root_dir }}/errors"
          - path: = /40x.html
            conf:
              - ssi: 'on'
              - internal:
              - root: "{{ openresty_root_dir }}/errors"
openresty_sites_conf: []
```

Dependencies
------------

Requires role:
  - name: ansible.systemd-service
    scm: git
    src: https://github.com/lablabs/ansible-role-systemd-service.git
    version: 1.0.0
    path: ./dist


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```
- name: "Deploy Openresty to {{ play_hosts }} static servers"
  hosts: web
  become: true
  tags: play:web
  environment: "{{ deploy_env_vars | default({}) }}"
  tasks:

    - name: Disable Healthcheck monitor on a server
      changed_when: false
      iptables:
        chain: INPUT
        protocol: tcp
        destination_port: "{{ item }}"
        jump: DROP
        state: present
      with_items:
        - "80"
        - "443"

    - include_role:
        name: deploy.openresty
        apply:
          tags: role:deploy.openresty
      tags: role:deploy.openresty
```

License
-------

BSD
