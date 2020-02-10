

Using mylaxx over Tor
=========================
mylaxx can be configured to run over the Tor network using either the Tor-only or dual-stack modes.

### Tor-only

In Tor-only mode the daemon will only make outgoing network connections using the SOCKS5 proxy. It will also automatically configure itself as a [Tor hidden service](https://www.torproject.org/docs/hidden-services.html.en)
and accept incoming network connections on a .onion address. All bitcoin related networking connections or external API queries will also use Tor.

In this mode you will be able to connect to both Tor and clearnet nodes, communicate with them in real time and browse their stores. However, clearnet nodes
will not be able to make outgoing connections to your node and may not be able to see your store. The lone exception here would be if someone running a dual-stack node
visits your store, then your store data would be cached and seeded by that node ― making it visible to clearnet nodes.

### Dual-stack

Dual-stack mode allows you to listen simultaneously on both Tor (via an onion address just like above) and also the clearnet. The reason you might want to run in this configuration
is to allow you to browse stores behind Tor without preventing clearnet nodes from viewing your store. You will also serve as a bridge between Tor and the clearnet as noted
above.

**WARNING**: Dual-stack mode is NOT private! Your IP will be visible on the network.

## Configuring Tor
Tor can be configured via the config file or a runtime option. You must have Tor running (either the Tor daemon or browser-bundle) on the same machine to use mylaxx over Tor.

##### Via config file
Edit the config file found in the mylaxx data directory to the following for Tor-only mode:
```
"Addresses": {
    "Swarm": [
      "/onion/erhkddypoy6qml6h:4003"
    ]
  },
```
Or the following for dual-stack mode:
```
"Addresses": {
    "Swarm": [
      "/onion/erhkddypoy6qml6h:4003",
      "/ip4/0.0.0.0/tcp/4001",
      "/ip6/::/tcp/4001"
    ]
  },
```
In both cases substituting the onion address above for your onion address found as the prefix of the `.onion_key` file in the same data directory.

##### Via runtime option
For Tor-only mode run mylaxxd with the `--tor` flag.
Example:
```
./mylaxxd start --tor
```
For dual-stack mode use the `--dualstack` flag.
Example:
```
./mylaxxd start --dualstack
```
The runtime option will override the swarm address configuration in the config file and use default ports.

## Advanced Tor configuration
If you changed the tor control port in your `torrc` file or you require authentication you can set both the control port and your tor control password in the mylaxx config file:
```
"Tor-config": {
    "Password": "yourpassword", 
    "TorControl": "127.0.0.1:9000"
},
```

Aternatively you can pass the tor control password in as a start up option:
```
./mylaxxd start --torpassword yourpassword
```

## Configuring the client
The mylaxx-desktop client **must** also be configured to run over Tor as some html tags, such as `IMG`, are allowed in the profile and store data and will trigger the client to make outgoing network calls.

To set tor in the reference client select `Manage Servers` from the menu then check `Use Tor` and make sure the socks5 proxy url is correct. 

<img src="https://i.imgur.com/Ht2ZRMd.png">

### Important Privacy Considerations

All nodes in mylaxx are identified by a peer ID such as `QmNgBZN7z1CfMLbwyEwnGoixjbSaBcP9fS5ecMzZwCq3Ku`. Other nodes in the network will associate your peer ID with your
network addresses (whether IPv4, IPv6, or onion). If you run mylaxxd in the clear even *once*, you must assume *someone* has recorded the mapping between your
peer ID and your IP address. Therefore using a given peer ID in the clear, *then* switching to Tor-only mode will almost certainly blow your privacy.

Therefore if you wish to run in Tor-only mode, it is *highly recommended* that you use a fresh peer ID which has never been used on the network and has not had a chance
to get associated with your actual IP address. To get a new peer ID you can just delete your data folder and restart mylaxxd. It will create a new peer ID on start up.


