# Networking {#duckiebot-network status=ready}

This page is for the `DB18` configuration used in classes in 2018. For last year's instructions see [here](https://docs.duckietown.org/DT17/).

<div class='requirements' markdown="1">

Requires: A Duckiebot that is initialized according to [](#setup-duckiebot).

Requires: Patience (channel your inner Yoda).

Result: A Duckiebot that you can connect to and that is connected to the internet.

</div>

The instructions here are ordered in terms of preference, the first being the most preferable and best.

By default on boot your robot will look for a network with a "`duckietown`" SSID, unless you changed it in [the SD card flashing procedure](#burn-sd-card). You can connect to your robot wirelessly by connecting to that network.

This page describes how to get your robot connected to the wide-area network (internet).

## Testing if your Duckiebot is Connected to the Internet {#duckiebot-network-test}

Some networks block pings from passing through, so a better way is to execute on your duckiebot:

    duckiebot $ sudo curl google.com

which will try to download the Google homepage. If it is successful, you should see an output like:

    <HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
    <TITLE>301 Moved</TITLE></HEAD><BODY>
    <H1>301 Moved</H1>
    The document has moved
    <A HREF="http://www.google.com/">here</A>.
    </BODY></HTML>

## Option 1: Connect your Duckiebot to the internet through a Wifi router that you control

If you are working from your home, for example, you simply need to make the Duckiebot connect to your home network. You may have input the proper SSID and pwd when you initialized the SD card, in which case, your Duckiebot should be connected to the internet already.

If you didn't enter the right SSID and password for your network or you want to change you need to connect to your robot somehow (e.g. with ethernet) and then edit the file `/etc/wpa_supplicant/wpa_supplicant.conf` as explained in the [Duckiebot initialization procedure](#burn-sd-card).

This is the best option.

## Option 2: Bridge the internet connection through your laptop with ethernet

This method assumes that you can connect your laptop to a network but it is one that you don't control or is not open. For example, on campus many networks are more protected, e.g. with PEAP. In that case, it can be difficult to get your configurations right on the Duckiebot. An alternative is bridge the connection between your laptop and your Duckiebot whenever you need internet access on the robot.

### Ubuntu

1. Connect your laptop to a wireless network.
2. Connect the Duckiebot to your laptop via an ethernet cable.

3. Make a new ethernet connection:

4. 1. Network Settings… (or run the command `nm-connection-editor`)
   2. Click "Add"
   3. Type -> Ethernet
   4. Connection Name: "Shared to Duckiebot"
   5. Select "IPV4" tab
   6. Select Method
   7. Select “Shared to other computers”
   8. Click apply.

Now, you should be able to SSH to your Duckiebot:

```
laptop $ ssh ![hostname]
```

Check whether you can access the internet from your Duckiebot:

```
duckiebot $ sudo curl google.com
```

Now, try to pull a Docker image:

```
duckiebot $ sudo docker pull duckietown/rpi-simple-server # This should complete successfully
```

If the previous command does not work, you may need to change the system date. To do so, run the following command:

```
duckiebot $ sudo date -s "2018-09-18 15:00:00" # Where this is the current date in YYYY-MM-DD HH-mm-ss
```

### Mac

Untested instructions [here](https://medium.com/@tzhenghao/how-to-ssh-into-your-raspberry-pi-with-a-mac-and-ethernet-cable-636a197d055)

## Option 3: Push Docker Images from Laptop {#duckiebot-network-push status=ready}

Since we are primarily using the internet to pull Docker images, we can simply connect the laptop and the Duckiebot then push Docker images from the laptop over SSH like so:

```
laptop $ docker save duckietown/![image-name] | ssh -C ![hostname] docker load
```

Then the image will be available on your Duckiebot.

If you can connect to your laptop (e.g. through a router) but do not have internet access then you can proceed for now, but everytime you see a command starting with:

```
duckiebot $ docker run ...
```

note that you will need to pull onto your laptop and push to your Duckiebot in order to load the latest version of the image.

## Common Troubleshooting

### I cannot ping the Duckiebot

Symptom: `ping ![robot_name]` does not work.

Resolution: Check if your laptop and Duckiebot are connected to the same network.

Additional debugging steps:

- Step 1: Check that your Raspberry Pi is responsive by observing the blinking LED on the Raspberry Pi.

- Step 2: Connect your Duckiebot with the laptop using the ethernet cable. Check if you are able to ping the Duckiebot. This will provide you an hint if there is an issue with the robot or network.

- Step 3: Check that this file: `/etc/wpa_supplicant/wpa_supplicant.conf` contains all the wifi networks in the correct syntax that you want to connect.

- Step 4: If it's your private access point, then you can access your router, typically connecting to `192.168.0.1`, where you can see all the devices connected. Make sure that both your Duckiebot and your laptop are in the list.

- Step 5: Check the file `~/.ssh/config` has the correct name hostname with `hostname.local` defined.

## I cannot access my Duckiebot via SSH {#troubleshooting-mdns-ipv6 status=ready}

Symptom: When I run `ssh ![robot_name].local` I get the error `ssh: Could not resolve hostname ![robot_name].local`.

Resolution: Make sure that your Duckiebot is ON. Connect it to a monitor, a USB mouse and a keyboard. Run the command:

    duckiebot $ sudo service avahi-daemon status

You should get something like the following:

    ● avahi-daemon.service - Avahi mDNS/DNS-SD Stack
       Loaded: loaded (/lib/systemd/system/avahi-daemon.service; enabled; vendor preset: enabled)
       Active: active (running) since Sun 2017-10-22 00:07:53 CEST; 1 day 3h ago
     Main PID: 699 (avahi-daemon)
       Status: "avahi-daemon 0.6.32-rc starting up."
       CGroup: /system.slice/avahi-daemon.service
               ├─699 avahi-daemon: running [![robot_name_in_avahi].local
               └─727 avahi-daemon: chroot helpe

Avahi is the module that in Ubuntu implements the mDNS responder. The mDNS responder is responsible for advertising the hostname of the Duckiebot on the network so that everybody else within the same network can run the command `ping ![robot_name].local` and reach your Duckiebot. Focus on the line containing the hostname published by the `avahi-daemon` on the network (i.e., the line that contains `![robot_name_in_avahi].local`).
If `![robot_name_in_avahi]` matches the `![robot_name]`, go to the next Resolution point.
If `![robot_name_in_avahi]` has the form `![robot_name]-XX`, where `XX` can be any number, modify the file `/etc/avahi/avahi-daemon.conf` as shown below.

Identify the line

    use-ipv6=yes

and change it to

    use-ipv6=no

Identify the line

    #publish-aaaa-on-ipv4=yes

and change it to

    publish-aaaa-on-ipv4=no

Restart Avahi by running the command

    duckiebot $ sudo service avahi-daemon restart



Symptom: (for Arch users only) the Avahi module isn't installed

Resolution: Install Avahi with the following command:

    sudo pacman -S avahi

Install the nss-mdns package with the command

    sudo pacman -S nss-mdns

Edit the `/etc/nsswitch.conf` file and change the hosts line to include

    mdns_minimal [NOTFOUND=return] before `resolve` and `dns`

it should look something like

    hosts: ... mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] dns ...

Restart the avahi-daemon.service service

## I can SSH to the Duckiebot but not without a password

Check the file `~.ssh/config` and make sure you add your `ssh` key there, in case it doesn't exists.

The `init_sd_card` [procedure](#setup-duckiebot) should generate a paragraph in the above file in the following format:

    # --- init_sd_card generated ---
    Host duckiebot
        User duckie
        Hostname duckiebot.local
        IdentityFile /home/user/.ssh/DT18_key_00
        StrictHostKeyChecking no
    # ------------------------------

Do:

    $ ssh-keygen -f "/home/user/.ssh/known_hosts" -R hostname.local

It will generate a key for you, if it doesn't exists.

If there isn't any ssh key file in the `/.ssh/` directory create the file with

    $ touch /home/user/.ssh/DT18_key_00

Then copy and paste the following in the file:

    -----BEGIN OPENSSH PRIVATE KEY-----
    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
    -----END OPENSSH PRIVATE KEY-----


After creating the new file use the command

    chmod 600 /home/user/.ssh/DT18_key_00

to correctly configure the permissions for the file

If you are still unable to connect to the duckiebot without a password try using the following command to ssh into it anyways:

    ssh duckie@![hostname].local

Duckie is the default username, if you specified a different username in the `init_sd_card` command, type your username instead of duckie.

When promted for the password type `quackquack`or the password specified in the `init_sd_card` command if you specified one.

## Unable to communicate with Docker

Symptom: Error message appears saying `I cannot communicate with docker`. Also a warning `\"DOCKER_HOST\" is set to ![hostname].local` is present.

Resolution: Unset the `DOCKER_HOST`, running:

    laptop $ unset DOCKER_HOST

## Can ping but the robot doesn't move with the virtual joystick

Symptom: You can ping the robot, `ssh` into it, start the demos, but the commands from the virtual joystick do not seem to reach the robot.

A possible cause is that your computer's firewall is blocking the incoming traffic from the robot.

Resolution: Check the settings for the firewall on your computer and make sure that any incoming traffic from the IP address of the robot is allowed on all ports. Keep in mind that if your robot's IP address changes, you might need to update the rule.
