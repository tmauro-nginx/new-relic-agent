# NGINX Plug-In for New Relic

## Preface

Visualize performance metrics in New Relic for the open source NGINX software and NGINX&nbsp;Plus. 

The plug&#8209;in collects and reports various important metrics from an NGINX or NGINX&nbsp;Plus instance, such as:

  * Number of active client connections
  * Nubmer of idle (keepalive) client connections
  * Client connection acceptance and drop rates
  * Request rate

NGINX&nbsp;Plus customers can track additional metrics, including many related to upstream groups (groups of backend servers):

   * Number of upstream servers, broken down by state
   * Connections to upstream servers
   * Bandwidth usage
   * Upstream server response rate, broken down by HTTP status code
   * Status of health checks
   * Summary statistics for virtual servers, including request rate, response rate, and bandwidth usage
   * Cache statistics (responses broken down by status, traffic from the cache)

Metrics are displayed in the New Relic user interface and alerts can be configured based on values reported by the plug&#8209;in.

## Requirements

* An active New&nbsp;Relic account.

* A Linux or Unix environment with the following
software components installed:

  * Python (2.6, 2.7)
  * `python-daemon`
  * `make`
  * `python-setproctitle` (optional)

### Additional Requirements for RHEL/CentOS

  * `rpm-build`
  * `initscripts`

### Additional Requirements for Ubuntu/Debian

  * `dpkg-dev`
  * `debhelper`

## Building the Plug-In from Source

You can build the plug&#8209;in from source using the Makefile provided in this repo. Output is placed in the **build_output/** directory.

For RHEL/CentOS:

```
$ make rpm
```

For Ubuntu/Debian:

```
$ make debian
```

## Configuring the Plug-In

### Configuring Open Source NGINX

To collect and display metrics from the open source NGINX software, activate the [Stub Status](https://nginx.org/en/docs/http/ngx_http_stub_status_module.html) module by including the `stub_status on` directive in a dedicated `location` block in the NGINX configuration.

Example 1&nbsp;&ndash; Listen on localhost, allow access to the metrics from 127.0.0.1 only:

```nginx
server {
    listen 127.0.0.1:80;
    server_name localhost;

    location = /nginx_stub_status {
        stub_status on;
        allow 127.0.0.1;
        deny all;
    }
}
```

Example 2&nbsp;&ndash; Listen on `*:80`, restrict access to the metrics using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617):

```nginx
server {
    listen 80;
    server_name example.com;

    location = /nginx_stub_status {
        stub_status on;
        auth_basic "nginx status";
        auth_basic_user_file /path/to/auth_file;
    }
}
```

[Reference documentation for `auth_basic*` directives](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)

You must reload the configuration after modifying it:

```none
$ nginx -t
$ nginx -s reload
```

### Configuring NGINX Plus

To collect and display metrics from NGINX&nbsp;Plus, activate the [NGINX Plus API](http://nginx.org/en/docs/http/ngx_http_api_module.html) module by including the `api` directive in a dedicated `location` block in the NGINX&nbsp;Plus configuration.

Example&nbsp;&ndash; Listen on `*:80`, restrict access to the metrics using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc2617):

```nginx
server {
    listen 80;
    server_name example.com;

    location /api {
        api;
        auth_basic "nginx api";
        auth_basic_user_file /path/to/auth_file;
    }
}
```

[Reference documentation for `auth_basic*` directives](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)

You must reload the configuration after modifying it:

```none
$ nginx -t
$ nginx -s reload
```

### Configuring the New Relic Agent

For both NGINX Open Source and NGINX&nbsp;Plus, you must also edit the <span style="white-space: nowrap;"> **nginx-nr-agent.ini**</span> configuration file:

  * Add your New Relic license key.

  * Define each data source (NGINX Open Source or NGINX Plus instance) with the following parameters:
    * `url` (required)&nbsp;&ndash; Full URL to output from the `stub_status` or `api` module. In the preceding examples using HTTP Basic Authentication, the URLs are **http://www.example.com/nginx\_stub\_status** and **http://www.example.com/api**.
    * `name` (required)&nbsp;&ndash; Name of the instance to display in the New Relic UI.
    * `http_user`, `http_pass` (optional)&nbsp;&ndash; Credentials used for HTTP Basic Authentication.


#### Configuring an HTTPS Proxy to Access the New Relic API Endpoint

If the host for the **nginx-nr-agent** plug&#8209;in is behind an HTTPS proxy, add the proxy URL to the startup configuration script. The script location varies by operating system:

* On RHEL/CentOS: **/etc/sysconfig/nginx-nr-agent**
* On Ubuntu/Debian: **/etc/default/nginx-nr-agent**

To set the URL, export the `HTTPS_PROXY` variable, as in this example:

```none
$ export HTTPS_PROXY="<your-proxy-host>.example.com:3128"
```


## Running the Plug-In

The plug&#8209;in can be started as a daemon (for normal operation) or in foreground mode (for debugging). In normal operation, by default the plug&#8209;in runs as the `nobody` user and writes its log to **/var/log/nginx-nr-agent.log**.

To start the plug&#8209;in as a daemon, run the following command with `root` privilege:

```
$ service nginx-nr-agent start
```

When startup is successful, collected data appears in the New Relic
UI within a short time. If there is a problem, check the plug&#8209;in's output for error messages.

To check the plug&#8209;in's status, run:

```
$ service nginx-nr-agent status
```

To stop the plug&#8209;in, run:

```
$ service nginx-nr-agent stop
```

For debugging purposes, launch the plug&#8209;in in foreground mode, directing all output to `stdout`:

```
$ nginx-nr-agent.py -f start
```

## Support
The plug&#8209;in works with NGINX Open Source version&nbsp;1.13.4 and later, and NGINX&nbsp;Plus Release&nbsp;13 and later. NGINX,&nbsp;Inc. no longer officially maintains or provides free support for the plug&#8209;in.
