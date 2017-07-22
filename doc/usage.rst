Usage
=====

The configurable proxy runs two HTTP(S) servers:

- The public-facing interface to your application, controlled by ``--ip`` and
  ``--port``, listens on **all interfaces** by default.
- The inward-facing REST API, using ``--api-ip`` and ``--api-port``, listens on
  **localhost** by default. The REST API uses token authorization, where the
  token is set by the ``CONFIGPROXY_AUTH_TOKEN`` environment variable.


.. _default-target:

Setting a default target
------------------------

When you start the proxy from the command line, you can set a
default target, using the ``--default-target`` option, which will be used when
a matching route is not found in the proxy table:

.. code-block:: bash

    configurable-http-proxy --default-target=http://localhost:8888

.. _command-line:

Command line options
--------------------

.. code-block:: bash

    Usage: configurable-http-proxy [options]

    Options:

      -h, --help                       output usage information
      -V, --version                    output the version number
      --ip <ip-address>                Public-facing IP of the proxy
      --port <n> (defaults to 8000)    Public-facing port of the proxy

      --ssl-key <keyfile>              SSL key to use, if any
      --ssl-cert <certfile>            SSL certificate to use, if any
      --ssl-ca <ca-file>               SSL certificate authority, if any
      --ssl-request-cert               Request SSL certs to authenticate clients
      --ssl-reject-unauthorized        Reject unauthorized SSL connections (only meaningful if --ssl-request-cert is given)
      --ssl-protocol <ssl-protocol>    Set specific HTTPS protocol, e.g. TLSv1_2, TLSv1, etc.
      --ssl-ciphers <ciphers>          ':'-separated ssl cipher list. Default excludes RC4
      --ssl-allow-rc4                  Allow RC4 cipher for SSL (disabled by default)
      --ssl-dhparam <dhparam-file>     SSL Diffie-Helman Parameters pem file, if any

      --api-ip <ip>                    Inward-facing IP for API requests
      --api-port <n>                   Inward-facing port for API requests (defaults to --port=value+1)
      --api-ssl-key <keyfile>          SSL key to use, if any, for API requests
      --api-ssl-cert <certfile>        SSL certificate to use, if any, for API requests
      --api-ssl-ca <ca-file>           SSL certificate authority, if any, for API requests
      --api-ssl-request-cert           Request SSL certs to authenticate clients for API requests
      --api-ssl-reject-unauthorized    Reject unauthorized SSL connections (only meaningful if --api-ssl-request-cert is given)

      --default-target <host>          Default proxy target (proto://host[:port])
      --error-target <host>            Alternate server for handling proxy errors (proto://host[:port])
      --error-path <path>              Alternate server for handling proxy errors (proto://host[:port])
      --redirect-port <redirect-port>  Redirect HTTP requests on this port to the server on HTTPS
      --pid-file <pid-file>            Write our PID to a file
      --no-x-forward                   Don't add 'X-forward-' headers to proxied requests
      --no-prepend-path                Avoid prepending target paths to proxied requests
      --no-include-prefix              Don't include the routing prefix in proxied requests
      --insecure                       Disable SSL cert verification
      --host-routing                   Use host routing (host as first level of path)
      --statsd-host <host>             Host to send statsd statistics to
      --statsd-port <port>             Port to send statsd statistics to
      --statsd-prefix <prefix>         Prefix to use for statsd statistics
      --log-level <loglevel>           Log level (debug, info, warn, error)
      --proxy-timeout <n>              Timeout (in millis) when proxy receives no response from target


Custom error pages
------------------

Custom error pages can be provided when the proxy encounters an error and has no
proxy target to handle a request. There are two typical errors that CHP may hit,
along with their status code:

- 404: a client has requested a URL for which there is no routing target.
  This error can be prevented by setting a :ref:`default target <default-target>`
  before starting the configurable-http-proxy.

- 503: a route exists, but the upstream server isn't responding.
  This is more common, and can be due to any number of reasons,
  including the target service having died or not finished starting.

Setting the path for custom error pages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Specify an error path ``--error-path /usr/share/chp-errors`` when
starting the CHP::

.. code-block:: bash

    configurable-http-proxy --error-path /usr/share/chp-errors

When a proxy error occurs, CHP will look in the following location for a
custom html error page to serve:

.. code-block:: bash

    /usr/share/chp-errors/{CODE}.html

where ``{CODE}`` is a status code number for an html page to serve. If there is
a 503 error, CHP will look for a custom error page in this location
``/usr/share/chp-errors/503.html``.

If no custom error html file exists for the error code, CHP will use the
``error.html``. If you specify an error path, **make sure** you also create
an ``error.html`` file.

Setting a target for custom error handling
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can specify a target to use when errors occur by using ``--error-target {URL}``
when starting the CHP.
If, for example, CHP starts with ``--error-target http://localhost:1234``,
then when the proxy encounters an error, it will make a GET request to
the ``error-target`` server, with URL ``http://localhost:1234`` and status code
``/{CODE}``, and failing request's URL escaped in a URL parameter, e.g.:

.. code-block:: bash

    GET /404?url=%2Fescaped%2Fpath


Host-based routing
------------------

If the CHP is started with the ``--host-routing`` option, the proxy will
use the hostname of the incoming request to select a target.

When using host-based routes, the API uses the target in the same way as if
the hostname were the first part of the URL path, e.g.:

.. code-block:: python

    {
      "/example.com": "https://localhost:1234",
      "/otherdomain.biz": "http://10.0.1.4:5555",
    }


Troubleshooting
---------------

Q: My proxy is not starting. What could be happening?

- If this occurs on Ubuntu/Debian, check that the you are using a recent
  version of node. Some versions of Ubuntu/Debian come with a version of node
  that is very old, and it is necessary to update node.
