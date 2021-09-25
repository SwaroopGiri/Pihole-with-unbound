# Set up Pi-hole as truly self-contained DNS resolver
***
**Important Information**

This tutorial assumes that you have a Raspberry Pi setup with Rasberry Pi OS and Pi-hole cofigured on it.
***
### What is unbound?
[Unbound](https://www.nlnetlabs.nl/projects/unbound/about/) is a validating, recursive, caching DNS resolver developed by NLnet Labs, VeriSign Inc., Nominet, and Kirei.

Follow [this](https://github.com/SwaroopGiri/Pi_hole_configuration) tutorial if you haven't already configured Pi-hole.

### Setting up Pi-hole as a recursive DNS server solution

Install Unbound recursive DNS resolver:
```
sudo apt install unbound
```
For recursively querying a host that is not cached as an address, the resolver needs to start at the top of the server tree and query the root servers, to know where to go for the top level domain for the address being queried. Unbound comes with default builtin hints. Remember to update this file every 6 months.
```
wget -O root.hints https://www.internic.net/domain/named.root
sudo mv root.hints /var/lib/unbound/
```
### Configure `unbound`

Create and edit the config file  
`sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf`

Add the folowing contents:

```yaml
server:

    # The  verbosity  number, level 0 means no verbosity, only errors.
    # Level 1 gives operational information. Level  2  gives  detailed
    # operational  information. Level 3 gives query level information,
    # output per query.  Level 4 gives  algorithm  level  information.
    # Level 5 logs client identification for cache misses.  Default is
    # level 1.
    verbosity: 0
    
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    
    # May be set to yes if you have IPv6 connectivity
    do-ip6: no

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no
    
    # Use this only when you downloaded the list of primary root servers!
    # Read  the  root  hints from this file. Make sure to 
    # update root.hints evry 5-6 months.
    root-hints: "/var/lib/unbound/root.hints"
    
    # Trust glue only if it is within the servers authority
    harden-glue: yes
    
    # Ignore very large queries.
    harden-large-queries: yes
    
    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    # If you want to disable DNSSEC, set harden-dnssec stripped: no
    harden-dnssec-stripped: yes
    
    # Number of bytes size to advertise as the EDNS reassembly buffer
    # size. This is the value put into  datagrams over UDP towards
    # peers. The actual buffer size is determined by msg-buffer-size
    # (both for TCP and UDP).
    edns-buffer-size: 1232
    
    # Rotates RRSet order in response (the pseudo-random 
    # number is taken from Ensure privacy of local IP 
    # ranges the query ID, for speed and thread safety).  
    # private-address: 192.168.0.0/16
    rrset-roundrobin: yes
    
    # Time to live minimum for RRsets and messages in the cache. If the minimum
    # kicks in, the data is cached for longer than the domain owner intended,
    # and thus less queries are made to look up the data. Zero makes sure the
    # data in the cache is as the domain owner intended, higher values,
    # especially more than an hour or so, can lead to trouble as the data in
    # the cache does not match up with the actual data anymore
    cache-min-ttl: 300
    cache-max-ttl: 86400
    
    # Have unbound attempt to serve old responses from cache with a TTL of 0 in
    # the response without waiting for the actual resolution to finish. The
    # actual resolution answer ends up in the cache later on. 
    serve-expired: yes
    
    # Harden against algorithm downgrade when multiple algorithms are
    # advertised in the DS record.
    harden-algo-downgrade: yes
    
    # Ignore very small EDNS buffer sizes from queries.
    harden-short-bufsize: yes
    
    # Refuse id.server and hostname.bind queries
    hide-identity: yes
    
    # Report this identity rather than the hostname of the server.
    identity: "Server"
    
    # Refuse version.server and version.bind queries
    hide-version: yes
    
    # Prevent the unbound server from forking into the background as a daemon
    do-daemonize: no
    
    # Number  of  bytes size of the aggressive negative cache.
    neg-cache-size: 4m
    
    # Send minimum amount of information to upstream servers to enhance privacy
    qname-minimisation: yes
    
    # Deny queries of type ANY with an empty response.
    # Works only on version 1.8 and above
    deny-any: yes

    # Do no insert authority/additional sections into response messages when
    # those sections are not required. This reduces response size
    # significantly, and may avoid TCP fallback for some responses. This may
    # cause a slight speedup
    minimal-responses: yes
    
    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    # This flag updates the cached domains
    prefetch: yes
    
    # Fetch the DNSKEYs earlier in the validation process, when a DS record is
    # encountered. This lowers the latency of requests at the expense of little
    # more CPU usage.
    prefetch-key: yes
    
    # One thread should be sufficient, can be increased on beefy machines. In reality for 
    # most users running on small networks or on a single machine, it should be unnecessary
    # to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # more cache memory. rrset-cache-size should twice what msg-cache-size is.
    msg-cache-size: 50m
    rrset-cache-size: 100m
   
    # Faster UDP with multithreading (only on Linux).
    so-reuseport: yes
    
    # Ensure kernel buffer is large enough to not lose messages in traffix spikes
    so-rcvbuf: 4m
    so-sndbuf: 4m
    
    # Set the total number of unwanted replies to keep track of in every thread.
    # When it reaches the threshold, a defensive action of clearing the rrset
    # and message caches is taken, hopefully flushing away any poison.
    # Unbound suggests a value of 10 million.
    unwanted-reply-threshold: 100000
    
    # Minimize logs
    # Do not print one line per query to the log
    log-queries: no
    # Do not print one line per reply to the log
    log-replies: no
    # Do not print log lines that say why queries return SERVFAIL to clients
    log-servfail: no
    # Do not print log lines to inform about local zone actions
    log-local-actions: no
    # Do not print log lines that say why queries return SERVFAIL to clients
    logfile: /dev/null
    
    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```
to save conf file: 
press Ctrl+X --> Y --> Enter

### Check unbound config file for errors
This is optional. Check the config file for errors by `unbound-checkconf  /etc/unbound/unbound.conf.d/pi-hole.conf` it should return `no errors in in /etc/unbound/unbound.conf.d/pi-hole.conf`. 

Start unbound service and check whether the domain is resolving. The first few queries will be slow but the subsequent queries will resolve under 1ms.

### Test validation
You can test DNSSEC validation using
```
dig sigfail.verteiltesysteme.net @127.0.0.1 -p 5335
dig sigok.verteiltesysteme.net @127.0.0.1 -p 5335
```
The first command should give a status report of `SERVFAIL` and no IP address.
The second should give `NOERROR` plus an IP address.

### Configure Pi-hole
Configure Pi-hole to use unbound as your recursive DNS server:

Login to Pi-hole interface, Goto settings --> DNS --> Select Custom 1 (IPv4) and Type the unbound listening IP `127.0.0.1#5335` below it.

Uncheck everything else in Upstream DNS Servers section.

Make sure Never forward reverse lookups for private IP ranges and Never forward non-FQDNs is checked in Advanced DNS section.
Uncheck use DNSSEC option since unbound does that for us.

Only check Listen on all interfaces, permit all origins in Interface listening behavior if you're planning to setup a VPN server. For all other connections in local network, Listen on all interfaces should suffice.

Click save.

### Troubleshooting

Check if unbound is working without any errors by following command.

`sudo service unbound status`

If it throws anchor not OK error, restart unbound service by following command.

`sudo service unbound restart`

If unbound is still failing to start/trying to listen to port #53 which should be reserved for Pi-hole. Check is config file is appropriately configured by following command and give pi a reboot.

`cat /etc/unbound/unbound.conf.d/pi-hole.conf`

to edit conf file: 

`sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf`

to save: 
press Ctrl+X --> Y --> Enter

To reboot pi:
`sudo reboot`
