odtone-openwrt
==============

ODTONE package and configuration files for OpenWrt

Overview
--------

ODTONE-OpenWrt enables IEEE 802.21 capabilities in a commercial wireless
router/Access Point.

Supported Devices
-----------------

This list is supposed to grow with time. Contact us if you were able to
use ODTONE in a device not listed here:

* TP-LINK TL-WR1043ND
* Netgear WGT634U
  
Update your Device
------------------

To enable IEEE 802.21 capabilities in your router you need to:

* Get an appropriate image for it.
* Load this image to your device.
* Verify that everything works. 

## Getting the image

#### Get pre-compiled binary image

Not available yet.

#### Building from Sources

If you prefer to build your own image rather than using one of those given,
follow the instructions below. It is strongly recommended that you build
and load a vanilla OpenWrt tree before adding any ODTONE-related functionality.

This release is based on the Attitude Adjustment OpenWrt release.

##### Building OpenWrt

Install packages required by the OpenWrt buildsystem

    apt-get install build-essential binutils flex bison autoconf gettext texinfo sharutils \
                subversion libncurses5-dev ncurses-term zlib1g-dev gawk

Checkout and prepare Attitude Adjustment and the packages. For the rest of
this section we assume that `~/odtonewrt` is your working directory. 

    cd ~/odtonewrt
    svn co svn://svn.openwrt.org/openwrt/tags/attitude_adjustment_12.09/
    cd attitude_adjustment_12.09
    ./scripts/feeds update -a
    ./scripts/feeds install -a

Configure Attitude Adjustment (select target system) and the packages

    make menuconfig

If you select extra packages you should check if you have all prerequisites
installed. Check with:

    make prereq

Finally build Attitude Adjustment

    make

Load the image to your device. All images can be found under
`~/odtonewrt/attitude_adjustment_12.09/bin/` After the device has rebooted make
sure that you can login. Typically, this can be done as following:

    Connect to one of the "LAN" ports, not the Internet port (if there is any).
    Give your PC IP address 192.168.1.10, netmask 255.255.255.0
    Try to ping the device at 192.168.1.1
    Login to the device using "telnet 192.168.1.1" 

You should see an OpenWrt welcome message.

##### Add ODTONE extensions

Configure the PATH to use the toolchain gcc instead off the host gcc:

      http://wiki.openwrt.org/doc/devel/crosscompile

Go to your working directory and download the ODTONE extension.

    cd ~/odtonewrt/
    git clone https://github.com/ATNoG/odtone-openwrt.git

Add the ODTONE extensions to the `attitude_adjustment_12.09` directory.

    cd ~/odtonewrt/attitude_adjustment_12.09/package/
    ln -s ~/odtonewrt/odtone-openwrt/odtone-0.6/

Add the related package to your configuration

    cd ~/odtonewrt/attitude_adjustment_12.09
    make menuconfig

Choose the following:

    Select your platform for Target System (Broadcom BRCM47xx/953XX,Atheros AR71xx, etc)
    Select your platform at Target Profile (i.e. TP-Link-WR1043ND, Broadcom BRCM43xx Wifi, etc)
    Select ODTONE package under network
    Save and Exit 

Build the image

    make

Load the new image to your device.



## Loading the Image

There are different ways of loading the binary image to your device.
Please consult the related OpenWrt page and/or the OpenWrt info page
for your specific device for appropriate instructions.

#### Verifying update

By default, pre-compiled images and images build from the source code
will have the port labeled "internet" as management port (out-of-band),
with the static IP 192.168.1.1. You should be able to login through
that port as long as you configure your PC to the 192.168.1.0/24 subnet.
After you have configured your PC, try to login:

    telnet 192.168.1.1

By that time, you should be connected to the OpenWrt box which runs
ODTONE. Verify that the relevant processes are running:

    ps aux | grep odtone-mihf
    ps aux | grep sap_80211

If sap_80211 is not running, please check configuration section.


Configuration
-------------

ODTONE configuration files are located at `/etc/odtone folder`. It
contains two configuration files (`odtone.conf` and `sap_80211.conf`)
which are used to configure the `odtone-mihf` and the `sap_80211`
respectively.

To apply any changes in the ODTONE configuration you need to restart
your device or the daemon:

    /etc/init.d/odtone restart

### odtone-mihf

The odtone-mihf can be configured based on a set of parameters, which
can be configured using the configuration file located at
`/etc/odtone/odtone.conf`. The available configurable parameters are
presented next:

    MIHF Configuration Options:
      --help                                Display configuration options
      --conf.file arg (=odtone.conf)        Configuration file
      --conf.recv_buff_len arg (=4096)      Receive buffer length
      --mihf.id arg (=mihf)                 MIHF ID
      --mihf.ip arg (=127.0.0.1)            MIHF IP
      --mihf.remote_port arg (=4551)        Remote MIHF communication port
      --mihf.local_port arg (=1025)         Local SAPs communication port
      --mihf.peers arg                      List of peer MIHFs
      --mihf.users arg                      List of local MIH-Users
      --mihf.links arg                      List of local Links SAPs
      --mihf.transport arg (=udp, tcp)      List of supported transport protocols
      --mihf.link_response_time arg (=3000) Link SAP response time (milliseconds)
      --mihf.link_delete arg (=2)           Link SAP response fails to forget
      --mihf.discover arg                   MIHF Discovery Mechanisms Order
      --enable_broadcast                    Allows broadcast messages
      --enable_unsolicited                  Allows unsolicited discovery
      --log arg (=1)                        Log level [0-4]

All configurable parameters are self-explained and, therefore, we will
only mention those that are more complex to configure.

    List of peer MIHFs: Comma separated list of remote MIHF's.
    Usage: mihf.peers = <mihf id> <ip> <port> <list of supported transport protocols>, ...
    
    List of local MIH-Users: Comma separated list of local MIH-User SAP.
    Usage: mihf.users = <user sap id> <port> [supported commands> <supported queries>], ...
    
    List of local Link SAPs: Comma separated list of local MIH Link SAPs.
    Usage: mihf.links = <link sap id> <port> <technology type> <interface>, ...
    
    List of supported transport protocols: Comma separated list of the transport protocols
    available. For now UDP and TCP protocols are supported.
    Usage: mihf.transport = <udp/tcp>, ... 

### sap_80211

IMPORTANT: By default, the wireless is disable. You must turn it
on in the `/etc/config/wireless` by changing disabled 1 to disabled
0 in order to be able to run the sap_80211.

The sap_80211 can be configured based on a set of parameters, which
can be configured using the configuration file located at
`/etc/odtone/sap_80211.conf`. The available configurable parameters
are presented next:

    MIH Link SAP IEEE 80211 Configuration:
      --help                                Display configuration options
      --conf.file arg (=sap_8023.conf)      Configuration File
      --conf.recv_buff_len arg (=4096)      Receive Buffer Length
      --mihf.ip arg (=127.0.0.1)            Local MIHF Ip
      --mihf.local_port arg (=1025)         MIHF Local Communications Port
      --mihf.id arg (=local-mihf)           Local MIHF Id
      --link.verbosity arg (=2)             Log level [0-2]
      --link.default_th_period arg (=1000)  Default threshold checking interval (milliseconds)
      --link.link_addr arg                  Interface address
      --link.port arg (=1235)               Port
      --link.id arg (=link)                 Link SAP Id

Some parameters are mostly self-explanatory. The configuration should
usually hold the same parameters of the example configuration file,
except that the MAC address of the interface to be controlled needs to
be indicated for each link/system.

    default_th_period: When an MIH User configures a threshold on the device, indicating
    a value and crossing direction, the Link SAP is expected to trigger a Link_Parameters_Report
    message notifying the situation. Checking whether a certain parameter has crossed a
    threshold requires periodic polling of the attribute. This parameter sets the interval
    of that poll operation, in milliseconds.
    Usage: default_th_period = <time in ms>

External contributors
---------------------

If you want to contribute code, please try to:

1. Write good commit messages, explain what your patch does, and why it is
   needed.
2. Keep it simple: Any patch that changes a lot of code or is difficult to
   understand should be discussed before you put in the effort.
3. Make sure it works! :)

Once you have tried the above, tou can create a GitHub pull request to notify
us of your changes.

Ordered by date of the first contribution:

    Hugo Alves
