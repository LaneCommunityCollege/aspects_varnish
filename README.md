# aspects_varnish

A role to configure Varnish. If `aspects_packages` is in use, Varnish's official apt and yum repos can be installed and varnish will be installed from them.

# Requirements
Set ```hash_behaviour=merge``` in your ansible.cfg file.

# Setup
If you chose not to use `aspects_packages`, you will need to install `varnish` before running this role.
## Use `aspects_packages` to install apt repos
### Configure Varnish repo
You can configure the official Varnish package repositories for apt and yum via aspects_packages. 
Just add the appropriate dictionary configuration to group_var or host_vars.

For example, the Varnish 6.0 repos would look like:
```yaml
aspects_packages_add_repo_apt_repos:
  varnish:
    enabled: True
    key_url: "https://packagecloud.io/varnishcache/varnish60lts/gpgkey"
    repo: "deb https://packagecloud.io/varnishcache/varnish60lts/ubuntu/ {{ ansible_lsb.codename }} main"
  varnish-src:
    enabled: True
    key_url: "https://packagecloud.io/varnishcache/varnish60lts/gpgkey"
    repo: "deb-src https://packagecloud.io/varnishcache/varnish60lts/ubuntu/ {{ ansible_lsb.codename }} main"

aspects_packages_add_repo_yum_repos:
  varnish:
    state: "present"
    name: "varnishcache_varnish60lts"
    description: "varnishcache_varnish60lts"
    gpgkey: "https://packagecloud.io/varnishcache/varnish60lts/gpgkey"
    baseurl: "https://packagecloud.io/varnishcache/varnish60lts/el/7/$basearch"
    sslverify: "1"
    repo_gpgcheck: "0"
    gpgcheck: "0"
    enabled: "yes"
    sslcacert: "/etc/pki/tls/certs/ca-bundle.crt"
    metadata_expire: "300"
  varnishsource:
    state: "present"
    name: "varnishcache_varnish60lts-source"
    description: "varnishcache_varnish60lts-source"
    gpgkey: "https://packagecloud.io/varnishcache/varnish60lts/gpgkey"
    baseurl: "https://packagecloud.io/varnishcache/varnish60lts/el/7/SRPMS"
    sslverify: "1"
    repo_gpgcheck: "0"
    gpgcheck: "0"
    enabled: "yes"
    sslcacert: "/etc/pki/tls/certs/ca-bundle.crt"
    metadata_expire: "300"
```

# Role Variables
### `aspects_varnish_enabled`
Default is `False`, so this role will not do anything. Set to `True` to enable the role.

### `aspects_varnish_service_file_enabled`
Default: `False`. Set to `True` if you wish to override the default service file.

> Note: You like will want to do this. See [the docs on listening to port 80.](https://varnish-cache.org/docs/6.0/tutorial/putting_varnish_on_port_80.html#debian-v8-ubuntu-v15-04)

### `aspects_varnish_service_file`
A dictionary of values that are used in the service file. Only used when `aspects_varnish_service_file_enabled`
is `True`.
I have configured the following defaults in [defaults/main.yml](defaults/main.yml).
```yaml
aspects_varnish_service_file:
  listen_port: "8080"
  management_host: "localhost"
  management_port: "6082"
  vcl_path: "/etc/varnish/default.vcl"
  secret_path: "/etc/varnish/secret"
  storage_backend_type: "default"
  storage_backend_options: "256m"
```

### `aspects_varnish_secret`
Default is not set. You should set this in an encrypted file.

Not setting this could cause issues. Currently it is required.

### `aspects_varnish_vcl_format_version`
Default is: `vcl 4.0;` pulled from the example file that is installed with Varnish.

### `aspects_varnish_vcl_backends`
A dictionary/hash of backend configuration blocks.

The template runs the Jinja sort filter on it before writing anything.

See the example playbook for implementation examples.

### `aspects_varnish_vcl_recv_section_enabled`
Default is `False`. Set to `True` if you are using the `aspects_varnish_vcl_recv_section`.

### `aspects_varnish_vcl_recv_section`
A dictionary/hash of vcl_recv configuration blocks.

The template runs the Jinja sort filter on it before writing anything.

See the example playbook for implementation examples.

### `aspects_varnish_vcl_backend_response_section_enabled`
Default is `False`. Set to `True` if you are using the `aspects_varnish_vcl_backend_response_section`.

### `aspects_varnish_vcl_backend_response_section`
A dictionary/hash of vcl_backend_response configuration blocks.

The template runs the Jinja sort filter on it before writing anything.

See the example playbook for implementation examples.

### `aspects_varnish_vcl_deliver_section_enabled`
Default is `False`. Set to `True` if you are using the `aspects_varnish_vcl_deliver_section`.

### `aspects_varnish_vcl_deliver_section`
A dictionary/hash of vcl_deliver configuration blocks.

The template runs the Jinja sort filter on it before writing anything.

See the example playbook for implementation examples.

### `aspects_varnish_vcl_synth_section_enabled`
Default is `False`. Set to `True` if you are using the `aspects_varnish_vcl_synth_section`.

### `aspects_varnish_vcl_synth_section`
A dictionary/hash of vcl_synth configuration blocks.

The template runs the Jinja sort filter on it before writing anything.

See the example playbook for implementation examples.

### `aspects_varnish_vcl_backend_error_section_enabled`
Default is `False`. Set to `True` if you are using the `aspects_varnish_vcl_backend_error_section`.

### `aspects_varnish_vcl_backend_error_section`
A dictionary/hash of vcl_backend_error configuration blocks.

The template runs the Jinja sort filter on it before writing anything.

See the example playbook for implementation examples.

### `aspects_varnish_vcl_extra_section_enabled`
Default is `False`. Set to `True` if you are using the `aspects_varnish_vcl_extra_section`.

### `aspects_varnish_vcl_extra_section`
A dictionary/hash of extra configuration blocks that don't fit in a predefined section.

The template runs the Jinja sort filter on it before writing anything.

See the example playbook for implementation examples.
# Example Playbook

```yaml
- hosts:
    - aspects_varnish
  vars:
    ansible_become: True
    aspects_varnish_enabled: True
    aspects_packages_enabled: True
    aspects_packages_add_repo_apt_repos:
      varnish:
        enabled: True
      varnish-src:
        enabled: True
    aspects_packages_add_repo_yum_repos:
      varnish:
        enabled: "yes"
      varnish-source:
        enabled: "yes"
    aspects_varnish_service_file_enabled: True
    aspects_varnish_service_file:
      storage_backend_options: "1g"
    aspects_varnish_vcl_imports:
      - "import std;"
    aspects_varnish_vcl_backends:
      000000default:
        enabled: False
        block: |
          backend default {
              .host = "127.0.0.1";
              .port = "80";
          }
      111111testdrupal:
        enabled: True
        block: |
          backend testdrupal {
              .host = "testdrupal2.example.org";
              .port = "80";
              .connect_timeout = 360s;
              .first_byte_timeout = 360s;
              .between_bytes_timeout = 360s;
          }
    aspects_varnish_vcl_recv_section_enabled: True
    aspects_varnish_vcl_recv_section:
      000000set_x-forwarded-for:
        enabled: True
        block: |
          # Set x-forwarded-for header
          if (req.restarts == 0) {
              if (req.http.x-forwarded-for) {
                  set req.http.X-Forwarded-For = req.http.X-Forwarded-For + ", " + client.ip;
              } else {
                  set req.http.X-Forwarded-For = client.ip;
              }
          }
      111111configure_compression:
        enabled: True
        block: |
          # Handle compression correctly. Different browsers send different
          # "Accept-Encoding" headers, even though they mostly all support the same
          # compression mechanisms. By consolidating these compression headers into
          # a consistent format, we can reduce the size of the cache and get more hits.=
          # @see: http:// varnish.projects.linpro.no/wiki/FAQ/Compression
          if (req.http.Accept-Encoding) {
              if (req.http.Accept-Encoding ~ "gzip") {
                  # If the browser supports it, we'll use gzip.
                  set req.http.Accept-Encoding = "gzip";
              } else if (req.http.Accept-Encoding ~ "deflate") {
                  # Next, try deflate if it is supported.
                  set req.http.Accept-Encoding = "deflate";
              } else {
                  # Unknown algorithm. Remove it and send unencoded.
                  unset req.http.Accept-Encoding;
              }
          }
      222222set_backend_hint:
        enabled: True
        block: |
          if (req.http.host == "testdrupal.example.org") {
              set req.backend_hint = testdrupal;
          }
      555555never_cache:
        enabled: False
        block: |
          # Do not cache these paths.
          if (req.url ~ "^/update\.php$" || req.url ~ "^.*/ajax/.*$"){
              return (pass);
          }
    aspects_varnish_vcl_backend_response_section_enabled: True
    aspects_varnish_vcl_backend_response_section:
      000000static_files_no_cookies:
        enabled: True
        block: |
          #Don't allow static files to set cookies.
          if (bereq.url ~ "(?i)\.(png|gif|jpeg|jpg|ico|swf|css|js|html|htm)(\?[a-zA-Z0-9]+)?$") {
              #beresp == Back-end response from the web server.
              unset beresp.http.set-cookie;
              set beresp.ttl = 30m;
          }
          else if (beresp.status == 404) {
              set beresp.ttl = 60m;
          }
          else {
              set beresp.ttl = 5m;
          }
      111111allow_stale_items:
        enabled: True
        block: |
          #Allow items to be stale if needed.
          set beresp.grace = 4h;
    aspects_varnish_vcl_deliver_section_enabled: True
    aspects_varnish_vcl_deliver_section:
      000000default_drupal:
        enabled: True
        block: |
          if (obj.hits > 0) {
              set resp.http.X-Cache = "HIT";
          } else {
              set resp.http.X-Cache = "MISS";
          }
          set resp.http.Access-Control-Allow-Origin = "*";
          #Have some fun
          set resp.http.X-Powered-By = "Bits and Bots";
          #The value drupal sets for this is often used for fingerprinting:
          set resp.http.Expires = "Mon, 19 October 1964 00:00:00 GMT";
          #set a header to try and force most recent rendering mode for IE
          set resp.http.X-UA-Compatible = "IE=Edge,chrome=1";
          #Remove some http headers that aren't really necessary:
          #unset resp.http.Server;
          unset resp.http.X-Generator;
          unset resp.http.X-Drupal-Dynamic-Cache;
          unset resp.http.X-Drupal-Cache;
          #unset resp.http.Cache-Control;
          #unset resp.http.X-Frame-Options;
          #Add a Cache-Control Header
          if (req.url ~ "(?i)\.(png|gif|jpeg|jpg|ico)$"){
              set resp.http.Cache-Control = "max-age=2678400, private";
          }
          elseif (req.url ~ "(?i)\.(css|js)$") {
              set resp.http.Cache-Control = "max-age=1200, private";
          }
          else {
              set resp.http.Cache-Control = "max-age=150, private";
          }
    aspects_varnish_vcl_synth_section_enabled: True
    aspects_varnish_vcl_synth_section:
      000000resp_750:
        enabled: True
        block: |
          if (resp.status == 750){
              set resp.http.Location = resp.reason;
              set resp.status = 301;
              return(deliver);
          }
          return(deliver);
    aspects_varnish_vcl_backend_error_section_enabled: True
    aspects_varnish_vcl_backend_error_section:
      000000return_custom_404:
        enabled: True
        block: |
          set beresp.http.Content-Type = "text/html; charset=utf-8";
          synthetic({"<!DOCTYPE html><html lang='en'><head><title>404</title></head><body>404</body></html>"});
          return(deliver);
    aspects_varnish_vcl_extra_section_enabled: False
    aspects_varnish_vcl_extra_section:
      000000test:
        enabled: True
        block: |
          # This is a test of the vcl_extra section.
          # Comment line 2
  roles:
    - aspects_varnish
```
# License

MIT