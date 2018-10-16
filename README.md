Asterisk Introduction
---------------------
- Two Telephones
- one server
- SIP protocol

One telephone want to call the other, he asks the server
The server answer telephone A is in this office
                    telephone B is in that office

Asterisk as a huge feature set, it can do lots of things for you
- Voice mail box
- queue (call is waiting)
- it can replace a traditional PBX completely


Download
--------
- Always Download the LTS and the Certified version 
- The best practice is to downloda the source code, unpackage it, and then re-compile it
- sudo su
- apt-get install build-essential wget libssl-dev libncurses5-dev libnewt-dev libxml2-dev linux-headers-$(uname -r) libsqlite3-dev uuid-dev

- zxfv asterisk-certified-13.21-current.tar.gz

##############################################
Note:The certified release is traditionally for support agreement customers, as a result any module which is not supported under that agreement is not built by default. This includes chan_sip. You have to manually go into “make menuselect” and enable it.


so to enable chan_sip.co, type
$ make menuselect

Then choose chan_sip and enable it, then save and exit.

type 
$ make

then type
$ make install
##############################################

- cd asterisk-certified-13.21
- ./configure (figures out what is the environments and what are the libraries able to be compliled)
- make menuconfig (optional step - For advanced users only/ or if u need something special)
- make 
- make install

- cd /etc/asterisk 
- No configuration yet in this directory
- cd asterisk-certified-13.21
- make samples   ( it is dangeroues to use this on a production environment
                    because there might be something turns ON which you don't
                    want it to be ON and vice versa )

- cd /etc/asterisk

start asterisk
-------------
$ asterisk -cvvv 
*CLI> 
*CLI> 
*CLI> sip show peers

CTRL+C  exit and stop the process
We need a start script to automatically start the asterisk after reboot the server


vi /etc/asterisk/asterisk.conf
uncomment live-dangerously     on






Network setup
-------------
- All telephones must have ip addresses, and to achieve this you need to setup a dhcp server
- the time must be in sync so make sure to have your NTP set correctly

you might have to change the linux network setting

sudo vi /etc/network/interface
iface eth0 inet static
    address 10.1.1.10
    netmask 255.255.255.0
    gateway 10.1.1.1

sudo service networking restart

- Then install dhcp server
 sudo apt-get install dhcpd
 sudo vi /etc/default/udhcp
    # comment the following line to enable
    DHCPD_ENABLED="NO"    <<<< Change this to "YES"
 sudo vi /etc/udhcp.conf
 # The start and end of the IP lease block
 start      10.1.1.11
 end        10.1.1.200

 # the interface that udhcpd will use
 interface      eth0    <<< change it to your interaface enp3s0

 #Examples
 opt    dns     1.1.1.1
 opion  subnet  255.255.255.0
 opt    router  10.1.1.1
 option win     x.x.x.x    <<< delete this
 option dns     x.x.x.x     <<< delete this
 option domain  local
 option lease   864000

-Then start the dhcpd
 sudo /etc/init.d/udhcpd start


- Now check your time
$ date
$ sudo apt-get install ntp
$ sudo /etc/init.d/ntp stop
$ ntpdate pool.ntp.org
$ sudo /etc/init.d/ntp start







peer configuration
------------------
everything about peers is stored in /etc/asterisk/sip.conf
$ sudo cp /etc/asterisk/sip.conf /etc/asterisk/sip.conf.orig
$ sudo vi /etc/asterisk/sip.conf
 CTRL+C
 :g/^\s*;/d              # this is to delete any comment
 :g/^$/d                # this is to delete any empty line



add this line
[general]
qualify=yes    # this is to qualify the connection before it builts up


Now let's create the peer
[ahmed]
    type=friend     # means that can recieve/send calls (bidirection)
    context=phones
    allow=ulaw,alaw    #isdn codecs
    secret=123456789
    host=dynamic       # no need to know the ip address of the peer
    

then create another use
[usama]
    type=friend     # means that can recieve/send calls (bidirection)
    context=phones
    allow=ulaw,alaw    #isdn codecs
    secret=123456789
    host=dynamic       # no need to know the ip address of the peer



save and exit


make sip show peers
do sip reload


if you find that the satate is unknown , then this means that both peers are not
regestered yet. you have to go to your applicaiton and enter the username and passwrod
example; the username is the same one you entered in your sip_config file between the []
and the password is the same value you typed in the secret filed


Now download ZOIPER
enter the below
SIP Credentials:
    Domain:     <asterisk sip server ip>:<port>
    Username:   <username>
    password:   <secret>

Optional SIP credentails
    use autusername:    ibrahim 
    use outbound proxy: 192.168.48.179
if you try to call someone now, you will get an error, because there is no number
has been set yet 

0x7fc5a8010970 -- Strict RTP learning after remote address set to: 192.168.48.179:8000
[Oct  3 14:11:01] NOTICE[19756][C-00000000]: chan_sip.c:26459 handle_request_invite: Call from 'ibrahim' (192.168.48.179:57290) to extension '100' rejected because extension not found in context 'phones'.

Now this is just for registration, now let's jump into how to setting up 
phone numbers for each user
------------------------

Goto 
$ cd /etc/asterisk
$ cp extensions.conf extensions.conf.orig
$ echo "" > extensions.conf
$ sudo vi extensions.conf

add the below short dial plan, which the context that we put in the sip.conf

[phones]                            ; make sure the extension name is exactly the same name 
                                    ; you put inside the sip.conf file under the [user] context= field
                                    ; otherwise you will have problems
exten => 100,1,NoOp(First Line)     ; the server looks for the phone number 100
exten => 100,2,NoOp(Second Line)    ; the server looks again for the phone number 100
exten => 100,3,Hangup               ; the server hangs up

[bla]




Then asterisk -cvvv    <<< you have to increase the verbosity level
CLI> dialplan reload
CLI>



Adding phone numbers
--------------------
Note: NoOp and Hangup are applicaitons we use to instruct the asterisk what to do
if a certain number has been dialed
NoOp  means no operation, just echo the message between ()

so we need to know what is the applicaiton we have to use for dialing
we can use the below commands

CLI> core show application <application-name>
CLI> core show applications
CLI> core show applications like dial
CLI> core show applications describing dial


$ sudo vi extendsion.conf
[phones]
exten => 100,1,NoOp(First Line)     ; the server looks for the phone number 100
exten => 100,2,NoOp(Second Line)    ; the server looks again for the phone number 100
exten => 100,3,Dial(SIP/ziad)       ; to dial 100
exten => 100,4,Hangup               ; the server hangs up


CLI> dialplan reload

now try to call hahaha



---------


Customizing the Dial plan
------------------------
you can change the number of the dial plan as asterisk has been updated to be 
like this, but note that the first one has to start with 1

$ sudo vi extendsion.conf
[phones]
exten => 100,1,NoOp(First Line)     ; the server looks for the phone number 100
exten => 100,n,NoOp(Second Line)    ; the server looks again for the phone number 100
exten => 100,n,Dial(SIP/ziad)       ; to dial 100
exten => 100,n,Hangup               ; the server hangs up


Another thing, is that instead of repeating the same extension number
100 as example, you can use same as follows:

$ sudo vi extendsion.conf
[phones]
exten => 100,1,NoOp(First Line)     ; the server looks for the phone number 100
same =>  n,NoOp(Second Line)    ; the server looks again for the phone number 100
same =>  n,Dial(SIP/ziad)       ; to dial 100
same =>  n,Hangup               ; the server hangs up



Calling the other user
--------------------
[phones]
exten => 100,1,NoOp(First Line)     ; the server looks for the phone number 100
same =>  n,NoOp(Second Line)    ; the server looks again for the phone number 100
same =>  n,Dial(SIP/ziad)       ; to dial 100
same =>  n,Hangup               ; the server hangs up

exten => 200,1,NoOp(First Line)     ; the server looks for the phone number 100
same =>  n,NoOp(Second Line)    ; the server looks again for the phone number 100
same =>  n,Dial(SIP/ibrahim)       ; to dial 100
same =>  n,Hangup               ; the server hangs up






playback
-------

    exten => 200,1,NoOp(First Line)     ; the server looks for the phone number 100
    same =>  n,NoOp(Second Line)    ; the server looks again for the phone number 100
    same =>  n,Playback( ? )          ; to dial 100
    same =>  n,Dial(SIP/ibrahim)       ; to dial 100
    same =>  n,Hangup               ; the server hangs up


Go to 
cd /var/lib/asterisk
all your playback are in here under the sounds directory
they have tt at the beginning

    cd /var/lib/asterisk/sounds/en/
    ls -la tt-*
    
    sudo vi /etc/asterisk/extension.conf
    [phones]
    exten => 200,1,NoOp(First Line)     ; the server looks for the phone number 100
    same =>  n,NoOp(Second Line)    ; the server looks again for the phone number 100
    same =>  n,Playback(tt-monkeys)          ; to dial 100
    same =>  n,Dial(SIP/ibrahim)       ; to dial 100
    same =>  n,Hangup               ; the server hangs up


Incoming call simulation
---------------------
To allow someone from outside world to call someone from the inside

we have to update both the sip.conf and the extensions.conf


[Update sip.conf]

Append the below line to your previously configured users
    [outside]
    type=friend
    context=incoming
    allow=ulaw,alaw
    secret=1111111103   ;this could be any number
    host=dynamic

    
Don't forget to reload the sip from inside asterisk 
   CLI> sip reload
   
   
   
Now you have to add the 'incoming' context to your extensions.conf file

add the below to your extensions.conf

    [incoming]
    exten => 991123100,1,Goto(phones,100,1)    ; The external number 99123100 is only able to 
                                               ; only call 100 from the phones context


Asterisk Outgoing Call Configuration:
-------------------------------------
It is a good practice to have at least 3 contexts

- incomming
- outgoing
- internal / or in out case phones

This esential for country policy, because some companies/countries don't allow

 certain country numbers to call their phones and vice versa, so having more than 1 extension
 
 gives you a greate control of what can be called and what can't be called ( filtration )
 
 
 Add the below to your extensions.conf
 
    [outgoing]
    exten => 8888, 1, Dial(SIP/outside)

Now what is going to happen if someone from internal want to call 88888

is that, astersik will search inside the phones context looking for 88888

and it will find nothing and it will stop...

So we have to add another ( route ) if you want to say, to forward unknown routes to

the another exit point....

Add the below to your extensions.conf

    [phones]
    .
    .
    .
    exten => 8888,1,Goto(outgoing,8888,1)    


CLI> dialplan reload


   