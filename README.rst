dtool lookup webapp
===================

- GitHub: https://github.com/jic-dtool/dtool-lookup-webapp
- Free software: MIT License

Web application to query and display information about datasets stored in one
or more base URIs.

Dependencies
------------

This web application depends on having two other web services to talk to:

1. `dtool-lookup-server <https://github.com/jic-dtool/dtool-lookup-server>`_
2. `token-generator-ldap <https://github.com/jic-dtool/token-generator-ldap>`_

The first provides a means to search and display dataset metadata.
The latter provides a means to authenticate users.

Preamble
-------

::

    cd dtool-lookup-webapp


Create a file `.env` in the `dtool-lookup-webapp` directory with the following contents:

::

    VUE_APP_DTOOL_LOOKUP_SERVER_URL="http://localhost:5000"
    VUE_APP_DTOOL_LOOKUP_SERVER_TOKEN_GENERATOR_URL="http://localhost:5001/token"

For deployment, replace these URLs with the actual endpoints of the lookup server and the token generator, i.e.

::

    VUE_APP_DTOOL_LOOKUP_SERVER_URL="https://demo.dtool.dev/lookup"
    VUE_APP_DTOOL_LOOKUP_SERVER_TOKEN_GENERATOR_URL="https://demo.dtool.dev/token"

for the public demo.dtool.dev test server instance.

Start a development server
--------------------------

::

    cd dtool-lookup-webapp
    npm run serve


Compile into a static single page website
-----------------------------------------

::

    cd dtool-lookup-webapp
    npm run build


More information
----------------

See dtool-lookup-webapp/README.md for more information about how to develop and build.
See provision/README.rst for instructions on how to deploy using Ansible.
