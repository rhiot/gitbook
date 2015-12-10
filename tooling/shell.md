# Rhiot shell

**Available since Rhiot 0.1.3.** Rhiot shell is a background process exposed via SSH protocol. You can connect to it using the following credentials:

    ssh rhiot@localhost -p 200
    password: rhiot

To start a shell background process, execute the following Rhiot [CMD](cmd.md) command:

    $ rhiot shell-start
    
You can execute all [CMD](cmd.md) commands in a Rhiot shell as well. For example to install Raspbian to `/dev/sdd1` SD 
card, execute the following command after logging-in via SSH:

    $ raspbian-install ssd1
    