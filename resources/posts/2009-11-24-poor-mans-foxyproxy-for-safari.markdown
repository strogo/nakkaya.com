---
title: Poor Man's Foxyproxy for Safari
tags: apple bash ssh
---

Safari is a lot faster than Firefox on Mac OS X, I have been thinking
about switching to Safari but not having a Foxyproxy equivalent was a
big problem.

Fortunately Apple does provide a command to set/enable/disable proxy
settings. Following script implements a poor man's version of Foxyproxy
when you run it, it will setup a SSH SOCKS proxy to the server and
enable proxy settings for safari, when killed with Ctrl-C, it will kill
the SSH connection and disable proxy settings.

    #!/bin/bash

    DEVICE="Airport"
    HOST="127.0.0.1"
    PORT="9999"


    echo "[+] Connecting"
    ssh -ND $PORT user@server.com &
    FIND_PID=$!
    sleep 5

    echo "[+] Enabling Proxy"
    sudo networksetup -setsocksfirewallproxy $DEVICE $HOST $PORT off

    function quit {
        echo "[+] Disabling Proxy"
        sudo networksetup -setsocksfirewallproxystate $DEVICE off
        kill -9 $FIND_PID
        exit
    }

    trap "quit" SIGINT SIGTERM

    while :
    do
            sleep 60
    done

Save it somewhere on your machine, and make it executable.

    chmod 755 foxy-proxy.sh

Now you are ready to defeat that evil proxy.
