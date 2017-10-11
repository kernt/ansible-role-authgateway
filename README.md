Role Name
=========
Ansible role to install and configure
[AuthGateway](https://github.com/stefano-pogliani/auth-gateway).


Requirements
------------
AuthGateways requiers three things to work:

  * An HTTPS proxy (nginx by default).
  * An Auth Proxy (oauth2_proxy by default).
  * NodeJS

AuthGateway supports non-default proxes through its flexible configuration
(more details on this in the AuthGateway repo/docs).

To support such a felxible system this role will not attepmt to
install most prerequisites but will check that they are installed
by the user prior to executing the current role.

For examples see below.


Role Variables
--------------
### Optional OAuth2Proxy installation
These options allow to optionally download oauth2_proxy.
By default oauth2_proxy is not installed and must be provided by the user.

As this is a common requirement that is simple to satisfy in a
cross-distribution way (unlike nginx or nodejs) the role can take
care of the installation.

| Variable | Description | Default |
| -------- | ----------- | ------- |
| `authgateway_configure_oauth2_proxy` | Boolean to optionally install oauth2_proxy | `false` |
| `authgateway_oauth2_proxy_prefix` | Prefix to install `oauth2_proxy` in | `'/opt/oauth2_proxy'` |
| `authgateway_oauth2_proxy_checksum` | Checksum of the release archive | See `defaults/main.yml` |
| `authgateway_oauth2_proxy_release` | URL prefix to fetch the release archive from | See `defaults/main.yml` |
| `authgateway_oauth2_proxy_version` | The version to install (gives the name to the archive) | See `defaults/main.yml` |

### Paths to dependencies
These are the paths to the three dependencies of AuthGateway:

  * AuthProxy (oauth2_proxy by default)
  * HttpProxy (nginx by default)
  * NPM (to install and run AuthGateway)

| Variable | Description | Default |
| -------- | ----------- | ------- |
| `authgateway_auth_exec` | Path to the auth proxy executable | See `defaults/main.yml` |
| `authgateway_http_exec` | Path to the http proxy executable | `'nginx'` |
| `authgateway_npm_exec` | Path to the npm executable | `'npm'` |

### Install options
| Variable | Description | Default |
| -------- | ----------- | ------- |
| `authgateway_install_path` | Location to install AuthGateway to | `/opt/authgateway` |
| `authgateway_install_url` | URL of the GitHub release (tag) to install | See `defaults/main.yml` |
| `authgateway_user` | User to run AuthGateway | `'authgateway'` |
| `authgateway_group` | Group to run AuthGateway | `'authgateway'` |

### Configuration options
| Variable | Description | Default |
| -------- | ----------- | ------- |
| `authgateway_conf_apps` | List of apps to configure (see AuthGateway configuration docs) | `[]` |
| `authgateway_conf_session_ttl` | Time To Live of cookie sessions | `'168h'` |
| `authgateway_conf_bind` | Local address to bind the HTTP proxy to | `'*'` |
| `authgateway_conf_port` | Local port to bind the HTTP proxy to | `443` |
| `authgateway_conf_crt_file` | Path to the HTTPS certificate file | `'/opt/authgateway/gate.crt'` |
| `authgateway_conf_key_file` | Path to the HTTPS certificate key | `'/opt/authgateway/gate.key'` |
| `authgateway_conf_tls_terminate` | Perform TLS termination at the http proxy (diable when TLS is terminated at LB level) | `true` |
| `authgateway_conf_auditor_provider` | Provider of the audit log store | `'null'` |
| `authgateway_conf_auditor_endpoint` | When the audit provider is set to `http`, this endpoint is the target of audit `POST` | `'http://localhost:8092/audit'` |
| `authgateway_extra_emails` | List of emails that are allowed to connect | `[]` |

The following options are all **REQUIRED**:

| Variable | Description |
| -------- | ----------- |
| `authgateway_conf_domain` | Top domain of your gateway (i.e, gate.example.com) |
| `authgateway_conf_client_id` | OAuth client id issued by the provider |
| `authgateway_conf_client_secret` | OAuth client secrer issued by the provider |
| `authgateway_conf_provider` | OAuth provider |
| `authgateway_conf_session_secret` | Secret used by the Auth proxy to encrypt and decrypt the session data |

### Extra features
| Variable | Description | Default |
| -------- | ----------- | ------- |
| `authgateway_acmetool_target` | When set, it is the path to an acmetool target for the domain and all subdomains | `None` |


Dependencies
------------
This role comes without any explict dependency so it can be cross-platform
and more flexible to the needs and opinions of users.

That said, AuthGateway has some requirements that must be fulfilled before
this role is invoked:

  * A compatible HTTP proxy must be installed.
  * A compatible Auth proxy must be installed
    (the role can optionally install ouath2_proxy for you).
  * NodeJS runtime (tested with node 8.4+).


Example Playbook
----------------
The example playbook below uses:

  * My nginx from official repos (dedicated role) for the HTTP proxy.
  * The builtin oauth2_proxy install tasks.
  * A quick and dirty play (`tests/nodejs.yml`) to install NodeJS (**NOT recommended**).

```yaml
# Configure required dependencies.
- hosts: servers
  roles:
    - role: stefano-pogliani.upstream-nginx
      upstream_nginx_service_enabled: False
      upstream_nginx_service_state: 'stopped'
    #- role: some.nodejs.role
    #  some_nodejs_option: ...
  tasks:
    - import_tasks: 'tests/nodejs.yml'

# Install and configure AuthGateway itself.
- hosts: servers
  roles:
    - role: stefano-pogliani.authgateway
      authgateway_configure_oauth2_proxy: true
      authgateway_conf_domain: 'gate.example.com'
      authgateway_conf_client_id: 'some id'
      authgateway_conf_client_secret: 'some secret'
      authgateway_conf_provider: 'github'
      authgateway_conf_session_secret: 'some secret generated by user'
      authgateway_extra_emails:
        - 'test1@example.com'
        - 'test2@example.com'
      authgateway_conf_apps:
        - name: Google
          url: 'https://www.google.co.uk/'
        - name: Test
          upstream:
            host: 'localhost:1234'
            protocol: 'http'
```


License
-------
MIT


Testing
-------
This role uses [test-kitchen](http://kitchen.ci/) and
[kitchen-ansible](https://github.com/neillturner/kitchen-ansible).

```bash
bundle install --path=.vendor --binstubs
./bin/kitchen test
```
