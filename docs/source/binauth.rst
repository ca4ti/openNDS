BinAuth Option
=================

**Key: BinAuth**

**Value: /path/to/executable/script**

Authenticate a client using an external program that get passed the (optional) username and password value.
The exit code and output values of the program decide if and how a client is to be authenticated.

For the following examples, setting `binauth` in nodogsplash.conf is set to `/etc/nds_auth.sh`:

.. code-block:: sh

    #!/bin/sh

    METHOD="$1"
    MAC="$2"

    case "$METHOD" in
      "client_auth")
        USERNAME="$3"
        PASSWORD="$4"
        if [ "$USERNAME" = "Bill" -a "$PASSWORD" = "tms" ]; then
          # Allow client to access the Internet for one hour (3600 seconds)
          # Further values are upload and download limits in bytes. 0 for no limit.
          echo 3600 0 0
        fi
        ;;
      "idle_timeout")
        INGOING_BYTES="$3"
        OUTGOING_BYTES="$4"
        DURATION_SECONDS="$5"
        # The client was deauthenticated after DURATION_SECONDS seconds because of inactivity
        ;;
      "session_end")
        INGOING_BYTES="$3"
        OUTGOING_BYTES="$4"
        DURATION_SECONDS="$5"
        # The client was deauthenticated after DURATION_SECONDS seconds because the session ended
        ;;
    esac

    exit 0


The splash.html page contains the following code:

.. code-block:: html

    <form method='GET' action='$authaction'>
    <input type='hidden' name='tok' value='$tok'>
    <input type='hidden' name='redir' value='$redir'>
    username: <input type='text' name='username' value='' size='12' maxlength='12'>
    <br>
    password: <input type='password' name='password' value='' size='12' maxlength='10'>
    <br>
    <input type='submit' value='Enter'>
    </form>

If a client enters a username 'Bill' and password 'tms', then the configured `binauth` script is executed:

.. code::

   /etc/nds_auth.sh client_auth 12:34:56:78:90 'Bill' 'tms'

For the authentication to be successful, the exit code of the script must be 0. The output can be up to three values. First the number of seconds the client is to authenticated, second and third the maximum number of upload and download bytes. Values not given to NDS will resort to default values. Note that the traffic shaping feature does not work right now.

Nodogsplash will also call the script when the client is deathenticated.

Client is deauthenticated due to inactivity:

.. code::

   /etc/nds_auth.sh idle_timeout <mac> <incoming_bytes> <outgoing_bytes> <duration_seconds>

Client is deauthenticated due to the session end:

.. code::

   /etc/nds_auth.sh session_end <mac> <incoming_bytes> <outgoing_bytes> <duration_seconds>