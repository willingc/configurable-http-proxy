Introduction
============

**configurable-http-proxy** (CHP) provides you with a way to update and manage
a proxy table using a command line interface or REST API.
It is a simple wrapper around [node-http-proxy][]. node-http-proxy is an HTTP
programmable proxying library that supports websockets and is suitable for
implementing components such as reverse proxies and load balancers. By
wrapping node-http-proxy, **configurable-http-proxy** extends this
functionality to [JupyterHub] deployments.
