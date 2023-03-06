# Update #1

### What I have done...

I decided to have a server both Mike and I have on digital ocean be the server for telegraf to query via snmp. I am using my raspberry pi as the client to collect the data it gets from the snmp query.

1. My first step was to download telegraf on the client side (my Raspberry pi). I did this by running the following commands
   
       wget -q https://repos.influxdata.com/influxdata-archive_compat.key
       echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
       echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
       sudo apt-get update && sudo apt-get install telegraf

2. The next step is to create a configuration file for telegraf, so we can establish an input via snmp and an output of prometheus. You can see from the telgraf.conf what that would look like.

3. In order for us to recieve the information from our Rocky Linux server, we needed to enable snmp on that server. I installed with this command.

    dnf install net-snmp net-snmp-utils

I then wanted to check if it was running by running a...

    sudo systemctl status snmpd

If it wasn't running, I simply started it by running...

    sudo systemctl start snmpd
    sudo systemctl enable snmpd

The enabled command has it start automatically everytime the system boots.

### What I am having trouble with...

After running through the steps and having telegraf installed, telegraf configured, and snmp running on ther server, I am having trouble getting telegraf to connect to the server. I run the command...

    telegraf --config telegraf.conf

and it runs the way it should. The error I get is a timeout error because it is having a hard time connecting via snmp. To troubleshoot, I have sent UDP packets from the raspberry pi to the Rocky Linux server to make sure it was listening and recieving those packets. It wasn't at first, so I ran the packets through port 5000 and went into the Rocky Linux firewall, adding port 5000 to allow traffic.

    sudo firewall-cmd --add-port=5000/udp --permanent
    sudo firewall-cmd --reload

After doing that, I was able to communicate between the two via UDP. My next step was to run a 'snmpwalk' command on the Rocky Linux machine, just to make sure I could get an output. I ran...

    snmpwalk -v 2c -c public 137.184.91.116 .1.3.6.1.2.1.1.1.0

which came back with system information. SO I knew this was working correctly. When I run that same command from the Raspberry pi, I get a Timeout error, telling me I am not connected via snmp.



### What I am going to do next...

This could be a configuration problem on the client side. I have done extensive troubleshooting from the server side, and have come up with nothing. All the ports are listening correctly and the snmpd.conf is configured to listen for my raspberry pi's ip address. 

Once I figure out the connection problem, I can go on and have this output so that it is readable for prometheus. It should be written to do that already, but haven;t been able to test it just do to the connection problems between the server and client. 

From there, it should be smooth sailing to get this connected to grafana and having real time data being graphed.

P.S. The digital ocean server is being very weird and not connecting via ssh right now, so I wasn't able to steal those files from FileZilla to upload to this repo. They will be up as soon as I can connect via ssh. SOOOOO many connection problems.
