Using the REST API
==================

The configurable-http-proxy REST API is documented and available as:

- a nicely rendered, interactive version at the `petstore swagger site <http://petstore.swagger.io/?url=https://raw.githubusercontent.com/jupyterhub/configurable-http-proxy/master/doc/rest-api.yml#/default>`_
- a `swagger specification file <https://github.com/jupyterhub/configurable-http-proxy/blob/master/doc/rest-api.yml>`_ in this repo

Basics
------

API Root
~~~~~~~~

+-------------+----------+----------+
| HTTP method | Endpoint | Function |
+=============+==========+==========+
| GET         | /api/    | API Root |
+-------------+----------+----------+

Routes
~~~~~~

+-------------+--------------------------+----------------------------------------------------+
| HTTP method | Endpoint                 | Function                                           |
+=============+==========================+====================================================+
| GET         | /api/routes              | :ref:`Get all routes in routing table <get-table>` |
+-------------+--------------------------+----------------------------------------------------+
| POST        | /api/routes/{route_spec} | :ref:`Add a new route <add-route>`                 |
+-------------+--------------------------+----------------------------------------------------+
| DELETE      | /api/routes/{route_spec} | :ref:`Remove the given route <delete-route>`       |
+-------------+--------------------------+----------------------------------------------------+


Authenticating via passing a token
----------------------------------

The REST API is authenticated via passing a token in the ``Authorization``
header. The API is served under the ``/api/routes`` base URL.

For example, this ``curl`` command entered in the terminal
passes this header ``"Authorization: token $CONFIGPROXY_AUTH_TOKEN"`` for
authentication and this endpoint ``http://localhost:8001/api/routes`` to
retrieve the current routing table:

.. code-block:: bash

    curl -H "Authorization: token $CONFIGPROXY_AUTH_TOKEN" http://localhost:8001/api/routes

.. _get-table:

Getting the routing table
-------------------------

Request
~~~~~~~

.. code-block:: bash

    GET /api/routes[?inactive_since=ISO8601-timestamp]


Parameters
~~~~~~~~~~

``inactive_since``
^^^^^^^^^^^^^^^^^^

If the ``inactive_since`` URL parameter is given as an
`ISO8601 <http://en.wikipedia.org/wiki/ISO_8601>`_ timestamp, only routes whose
``last_activity`` is earlier than the timestamp will be returned. The
``last_activity`` timestamp is updated whenever the proxy passes data to or from
the proxy target.

Response
~~~~~~~~

Status code
^^^^^^^^^^^

status: 200 OK

Response body
^^^^^^^^^^^^^

A JSON dictionary of the current routing table. This JSON
dictionary *excludes* the default route.

Behavior
~~~~~~~~

The current routing table is returned to the user if the request is
successful.

.. _add-route:

Adding new routes
-----------------

POST requests create new routes. The body of the request should be a JSON
dictionary with at least one key: ``target``, the target host to be proxied.

Request
~~~~~~~

.. code-block:: bash

    POST /api/routes/[:path]

Required input
^^^^^^^^^^^^^^

`target`: The host URL

Example request body
^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    {
      "/user/fred": {
        "target": "http://localhost:8002"
      },
      "/user/barbara": {
        "target": "http://localhost:8003"
      }
    }

Response
~~~~~~~~

status: 201 Created

Behavior
~~~~~~~~

After adding the new route, any request to ``/path/prefix`` on the proxy's
public interface will be proxied to ``target``.

.. _delete-route:

Deleting routes
---------------

Request
~~~~~~~

.. code-block:: bash

    DELETE /api/routes/[:path]

Response
~~~~~~~~

status: 204 No Content

Behavior
~~~~~~~~

Removes a route from the proxy's routing table.
