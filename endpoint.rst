##################
 Remote Endpoints
##################

********
Overview
********

The ``remote`` command group allows users to manage the service
endpoints {Singularity} will interact with for many common command
flows. This includes managing credentials for image storage services,
remote builders, and keyservers used to locate public keys for SIF
image verification. Currently, there are three main types of remote
endpoints managed by this command group: the public Sylabs Cloud (or
local {Singularity} Enterprise installation), OCI registries, and
keyservers.

*******************
Public Sylabs Cloud
*******************

Sylabs introduced the online `Sylabs Cloud
<https://cloud.sylabs.io/home>`_ to enable users to `Create
<https://cloud.sylabs.io/builder>`_, `Secure
<https://cloud.sylabs.io/keystore?sign=true>`_, and `Share
<https://cloud.sylabs.io/library>`_ their container images with others.

A fresh, default installation of {Singularity} is configured to connect to the
public services available at `cloud.sylabs.io <https://cloud.sylabs.io>`__. If
you only want to use these public services, all you need to do is obtain an
authentication token, which you then provide to ``singularity remote login``:

   #. Go to: https://cloud.sylabs.io/
   #. Click "Sign In" and follow the sign-in steps.
   #. Click on your login id, which, after sign-in, should appear on the right
      side of the navigation-bar at the top of the page.
   #. Select "Access Tokens" from the drop down menu.
   #. Enter a name for your new access token, such as "test token".
   #. Click the "Create a New Access Token" button.
   #. Click "Copy token to Clipboard" from the "New API Token" page.
   #. Run ``singularity remote login`` and paste the access token when
      prompted.

Once your token is stored, you can check that you are able to connect to
the services with the ``status`` subcommand:

.. code:: console

   $ singularity remote status
   INFO:    Checking status of default remote.
   SERVICE    STATUS  VERSION                  URI
   Builder    OK      v1.6.9-rc.4-0-g87336319  https://build.sylabs.io
   Consent    OK      v1.7.0-0-g66ba1a9        https://auth.sylabs.io/consent
   Keyserver  OK      v1.18.12-0-gab541fb      https://keys.sylabs.io
   Library    OK      v0.3.8-rc.6-0-g630cdaa   https://library.sylabs.io
   Token      OK      v1.7.0-0-g66ba1a9        https://auth.sylabs.io/token

   Logged in as: myname <myemail@example.com>

   INFO:    Access Token Verified!

   Valid authentication token set (logged in).

If you see any errors you may need to check if your system requires the setting
of environment variables for a network proxy, or if a firewall may be blocking
access to ``*.sylabs.io``. Talk to your system administrator.

You can interact with the public Sylabs Cloud using various
{Singularity} commands:

`pull
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_pull.html>`__,
`push
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_push.html>`__,
`build --remote
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_build.html#options>`__,
`key
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_key.html>`__,
`search
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_search.html>`__,
`verify
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_verify.html>`__,
`exec
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_exec.html>`__,
`shell
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_shell.html>`__,
`run
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_run.html>`__,
`instance
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_instance.html>`__

.. note::

   Using the commands listed above will not interact with the Sylabs cloud if
   given URIs beginning with ``docker://``, ``oras://`` or ``shub://``.

*************************
Managing Remote Endpoints
*************************

Users can setup and switch between multiple remote endpoints, which are
stored in their ``~/.singularity/remote.yaml`` file. Alternatively,
remote endpoints can be set system-wide by an administrator.

A remote endpoint may be the public Sylabs Cloud, a private installation of
Singularity Enterprise, or any community-developed service that is
API-compatible.

Generally, users and administrators should manage remote endpoints using
the ``singularity remote`` command, and avoid editing ``remote.yaml``
configuration files directly.

List and Login to Remotes
=========================

To ``list`` existing remote endpoints, run this:

.. code:: console

   $ singularity remote list

   Cloud Services Endpoints
   ========================

   NAME         URI              ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud  cloud.sylabs.io  YES     YES     NO

   Keyservers
   ==========

   URI                     GLOBAL  INSECURE  ORDER
   https://keys.sylabs.io  YES     NO        1*

The ``YES`` in the ``ACTIVE`` column for ``SylabsCloud`` shows that this
is the current default remote endpoint.

To ``login`` to a remote for the first time, or when a token
needs to be replaced (if it has expired or been revoked):

.. code:: console

   # Login to the default remote endpoint
   $ singularity remote login

   # Login to another remote endpoint
   $ singularity remote login <remote_name>

   # example...
   $ singularity remote login SylabsCloud
   singularity remote login SylabsCloud
   INFO:    Authenticating with remote: SylabsCloud
   Generate an API Key at https://cloud.sylabs.io/auth/tokens, and paste here:
   API Key:
   INFO:    API Key Verified!

If you ``login`` to a remote that you already have a valid token for, you will
be prompted for confirmation that you indeed want to replace the current token,
and the new token will be verified before it replaces your existing credential.
If you enter an incorrect token your existing token will not be replaced,

.. code:: console

   $ singularity remote login
   An access token is already set for this remote. Replace it? [N/y] y
   Generate an access token at https://cloud.sylabs.io/auth/tokens, and paste it here.
   Token entered will be hidden for security.
   Access Token:
   FATAL:   while verifying token: error response from server: Invalid Credentials

   # Previous token is still in place

.. note::

   It is important for users to be aware that the ``remote login`` command will
   store the supplied credentials or tokens unencrypted in your home directory.

Add & Remove Remotes
====================

To ``add`` a remote endpoint (for the current user only):

.. code:: console

   $ singularity remote add <remote_name> <remote_uri>

For example, if you have an installation of {Singularity} enterprise
hosted at enterprise.example.com:

.. code:: console

   $ singularity remote add myremote https://enterprise.example.com

   INFO:    Remote "myremote" added.
   INFO:    Authenticating with remote: myremote
   Generate an API Key at https://enterprise.example.com/auth/tokens, and paste here:
   API Key:

You will be prompted to setup an API key as the remote is added. The ``add``
subcommand will provide you with the web address you need to visit to generate
your new key.

To ``add`` a global remote endpoint (available to all users on the
system) an administrative user should run:

.. code:: console

   $ sudo singularity remote add --global <remote_name> <remote_uri>

   # example...
   $ sudo singularity remote add --global company-remote https://enterprise.example.com
   INFO:    Remote "company-remote" added.
   INFO:    Global option detected. Will not automatically log into remote.

.. note::

   Global remote configurations can only be modified by the root user and are
   stored in the ``etc/singularity/remote.yaml`` file under the {Singularity}
   installation location.

Conversely, to ``remove`` an endpoint:

.. code:: console

   $ singularity remote remove <remote_name>

Use the ``--global`` option as the root user to remove a global
endpoint:

.. code:: console

   $ sudo singularity remote remove --global <remote_name>

Insecure (HTTP) Endpoints
-------------------------

Starting with {Singularity} 3.9, if you are using a endpoint that only exposes
its service discovery file over an insecure HTTP connection, it can be added by
specifying the ``--insecure`` flag:

.. code:: console

   $ sudo singularity remote add --global --insecure test http://test.example.com
   INFO:    Remote "test" added.
   INFO:    Global option detected. Will not automatically log into remote.

This flag controls HTTP vs HTTPS only for service discovery. The
protocol used to access individual library-, build- and keyservice-URLs is
determined by the contents of the service discovery file.

Set the Default Remote
======================

To use a given remote endpoint as the default for commands such as ``push``,
``pull``, etc., use the ``remote use`` command:

.. code:: console

   $ singularity remote use <remote_name>

The remote designated as default shows up with a ``YES`` under the ``ACTIVE``
column in the output of ``remote list``:

.. code:: console

   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME            URI                     ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud     cloud.sylabs.io         YES     YES     NO
   company-remote  enterprise.example.com  NO      YES     NO
   myremote        enterprise.example.com  NO      NO      NO

   Keyservers
   ==========

   URI                     GLOBAL  INSECURE  ORDER
   https://keys.sylabs.io  YES     NO        1*

   * Active cloud services keyserver

   $ singularity remote use myremote
   INFO:    Remote "myremote" now in use.

   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME            URI                     ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud     cloud.sylabs.io         NO      YES     NO
   company-remote  enterprise.example.com  NO      YES     NO
   myremote        enterprise.example.com  YES     NO      NO

   Keyservers
   ==========

   URI                       GLOBAL  INSECURE  ORDER
   https://keys.example.com  YES     NO        1*

   * Active cloud services keyserver

{Singularity} 3.7 introduces the ability for an administrator to make a remote
the only usable remote for the system, using the ``--exclusive`` flag:

.. code:: console

   $ sudo singularity remote use --exclusive company-remote
   INFO:    Remote "company-remote" now in use.
   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME            URI                     ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud     cloud.sylabs.io         NO      YES     NO
   company-remote  enterprise.example.com  YES     YES     YES
   myremote        enterprise.example.com  NO      NO      NO

   Keyservers
   ==========

   URI                       GLOBAL  INSECURE  ORDER
   https://keys.example.com  YES     NO        1*

   * Active cloud services keyserver

This, in turn, prevents users from changing the remote they use:

.. code:: console

   $ singularity remote use myremote
   FATAL:   could not use myremote: remote company-remote has been set exclusive by the system administrator

If you do not want to switch remote with ``remote use``, you can:

-  Instruct ``push`` and ``pull`` commands to use an alternative library server
   using the ``--library`` option (for example:
   ``singularity pull -F --library https://library.example.com library://alpine``).
   Note that the URL provided to the ``--library`` option is the URL of the
   library service itself, not the service discovery URL for the entire remote.
-  Instruct the ``build --remote`` commands to use an alternative remote builder
   using the ``--builder`` option.
-  Instruct certain subcommands of the ``key`` command to use an alternative
   keyserver using the ``--url`` option (for example:
   ``singularity key search --url https://keys.example.com foobar``).

************************
Keyserver Configurations
************************

By default, {Singularity} will use the keyserver defined by the active remote's
service discovery file. This behavior can be changed or supplemented via the
``add-keyserver`` and ``remove-keyserver`` subcommands. These commands allow an
administrator to create a global list of keyservers that will be used to verify
container signatures by default, where ``order 1`` will be the first in the
list. Other operations performed by {Singularity} that reach out to a keyserver
will only use the first, or ``order 1``, keyserver.

When listing the default remotes, we can see that the default keyserver is
``https://keys.sylabs.io`` and the asterisk next to its order indicates that it
is the keyserver associated with the current remote endpoint. We can also see
the ``INSECURE`` column indicating that {Singularity} will use TLS when
communicating with the keyserver.

.. code:: console

   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME         URI              ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud  cloud.sylabs.io  YES     YES     NO

   Keyservers
   ==========

   URI                     GLOBAL  INSECURE  ORDER
   https://keys.sylabs.io  YES     NO        1*

   * Active cloud services keyserver

We can add a key server to list of keyservers as follows:

.. code:: console

   $ sudo singularity remote add-keyserver https://pgp.example.com
   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME         URI              ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud  cloud.sylabs.io  YES     YES     NO

   Keyservers
   ==========

   URI                      GLOBAL  INSECURE  ORDER
   https://keys.sylabs.io   YES     NO        1*
   https://pgp.example.com  YES     NO        2

   * Active cloud services keyserver

Here, we see that the ``https://pgp.example.com`` keyserver was
added to the list. We can specify the order in the list in which this keyserver
should be added, by using the ``--order`` flag:

.. code:: console

   $ sudo singularity remote add-keyserver --order 1 https://pgp.example.com
   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME         URI              ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud  cloud.sylabs.io  YES     YES     NO

   Keyservers
   ==========

   URI                      GLOBAL  INSECURE  ORDER
   https://pgp.example.com  YES     NO        1
   https://keys.sylabs.io   YES     NO        2*

   * Active cloud services keyserver

Since we specified ``--order 1``, the ``https://pgp.example.com`` keyserver was
added as the first entry in the list, and the default keyserver was moved to
second in the list. With this keyserver configuration, all default image
verification performed by {Singularity} will, when searching for public keys,
reach out to ``https://pgp.example.com`` first, and only then to
``https://keys.sylabs.io``.

If a keyserver requires authentication prior to being used, users can login
as follows, supplying the password or an API token at the prompt:

.. code:: console

   $ singularity remote login --username myname https://pgp.example.com
   Password (or token when username is empty):
   INFO:    Token stored in /home/myname/.singularity/remote.yaml

The output of `remote list` will now show that we are logged in to
``https://pgp.example.com``:

.. code:: console

   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME         URI              ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud  cloud.sylabs.io  YES     YES     NO

   Keyservers
   ==========

   URI                      GLOBAL  INSECURE  ORDER
   https://pgp.example.com  YES     NO        1
   https://keys.sylabs.io   YES     NO        2*

   * Active cloud services keyserver

   Authenticated Logins
   =================================

   URI                     INSECURE
   https://pgp.example.com NO

.. note::

   It is important for users to be aware that the ``remote login`` command will
   store the supplied credentials or tokens unencrypted in your home directory.

***********************
Managing OCI Registries
***********************

It is common for users of {Singularity} to use
`OCI <https://opencontainers.org/>`__ registries as sources for their container
images. Some registries require credentials to access certain images or even the
registry itself. Previously, the only method in {Singularity} to supply
credentials to registries was to supply credentials for each command or set
environment variables to contain the credentials for a single registry. See
:ref:`Authentication via Interactive Login
<sec:authentication_via_docker_login>` and :ref:`Authentication via Environment
Variables <sec:authentication_via_environment_variables>`.

Starting with {Singularity} 3.7, users can supply credentials
on a per-registry basis with the ``remote`` command.

Users can login to an OCI registry with the ``remote login`` command by
specifying a ``docker://`` prefix to the registry hostname:

.. code:: console

   $ singularity remote login --username myname docker://docker.io
   Password (or token when username is empty):
   INFO:    Token stored in /home/myname/.singularity/remote.yaml

   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME         URI              ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud  cloud.sylabs.io  YES     YES     NO

   Keyservers
   ==========

   URI                     GLOBAL  INSECURE  ORDER
   https://keys.sylabs.io  YES     NO        1*

   * Active cloud services keyserver

   Authenticated Logins
   =================================

   URI                 INSECURE
   docker://docker.io  NO

An entry for ``docker://docker.io`` now shows up under ``Authenticated Logins``,
and {Singularity} will automatically supply the configured credentials when
interacting with DockerHub. We can also see the ``INSECURE`` column indicating
that {Singularity} will use TLS when communicating with the registry.

We can be logged-in to multiple OCI registries at the same time:

.. code:: console

   $ singularity remote login --username myname docker://registry.example.com
   Password (or token when username is empty):
   INFO:    Token stored in /home/myname/.singularity/remote.yaml

   $ singularity remote list
   Cloud Services Endpoints
   ========================

   NAME         URI              ACTIVE  GLOBAL  EXCLUSIVE
   SylabsCloud  cloud.sylabs.io  YES     YES     NO

   Keyservers
   ==========

   URI                     GLOBAL  INSECURE  ORDER
   https://keys.sylabs.io  YES     NO        1*

   * Active cloud services keyserver

   Authenticated Logins
   =================================

   URI                            INSECURE
   docker://docker.io             NO
   docker://registry.example.com  NO

{Singularity} will supply the correct credentials for the registry based
on the hostname used, whenever using the following commands with a
``docker://`` or ``oras://`` URI:

`pull
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_pull.html>`__,
`push
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_push.html>`__,
`build
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_build.html>`__,
`exec
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_exec.html>`__,
`shell
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_shell.html>`__,
`run
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_run.html>`__,
`instance
<https://www.sylabs.io/guides/{version}/user-guide/cli/singularity_instance.html>`__

.. note::

   It is important for users to be aware that the ``remote login`` command will
   store the supplied credentials or tokens unencrypted in your home directory.

