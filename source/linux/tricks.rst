Hack and Slash
==============

There are a lot of things to remember while using a linux system.
Please, hack and slash your way through.

Monitor file system changes with ``inotifywait``
------------------------------------------------

.. code-block:: sh
    
    #!/bin/bash

    # Usage suggetion: cd $WORK_DIR; . auto-rebuild.sh &; disown -h; 

    inotifywait -mrq source -e modify,create,moved_to --exclude ".*\.(swp|bak|tmp|log)" |

        while read path action name; do

            if [ -e $path$name ] 
            then
            # exclude hidden files, since putting this in the intify command will cause wired bug
                if ! echo $name | egrep -q "^\..+"
                then
                    # DO YOUR THING HERE
                    echo A LOT TODO
                fi
            fi

        done

As stated in the comment, execute the following commands to run it 'kind of as a daemon':

.. code-block:: sh

    cd /YOUR/WORKING/DIR
    . NAME_OF_THE_SCRIPT.sh
    disown -h

Find and kill a process
-----------------------
One liner command:

.. code-block:: sh

    ps aux | grep -i some_program | awk '{print $2}' | xargs sudo kill -9


In bash:

.. code-block:: sh
    
    kill $(ps aux | grep '[p]ython some_program.py' | awk '{print $2}')

``grep '[p]ython'`` prevents the command from finding itself.

Using ``certbot``
-----------------

Expand or remove subdomains:

.. code-block:: sh
    
    /opt/certbot-auto certonly --webroot-path /usr/share/nginx/html  --cert-name gorgor.it -d gorgor.it,src.gorgor.it,test.gorgor.it,www.gorgor.it
    chown -R www-data:www-data /etc/letsencrypt/live/gorgor.it
    service nginx reload
