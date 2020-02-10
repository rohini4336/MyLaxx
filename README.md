# MyLaxx


### Options

```
Usage:
  mylaxxd [OPTIONS] start [start-OPTIONS]

The start command starts the mylaxx-Server

Application Options:
  -v, --version                   Print the version number and exit

Help Options:
  -h, --help                      Show this help message

[start command options]
      -p, --password=             the encryption password if the database is encrypted
      -t, --testnet               use the test network
      -r, --regtest               run in regression test mode
      -l, --loglevel=             set the logging level [debug, info, notice, warning, error, critical]
                                  (default: debug)
      -f, --nologfiles            save logs on disk
      -a, --allowip=              only allow API connections from these IPs
      -s, --stun                  use stun on µTP IPv4
      -d, --datadir=              specify the data directory to be used
      -c, --authcookie=           turn on API authentication and use this specific cookie
      -u, --useragent=            add a custom user-agent field
      -v, --verbose               print mylaxx logs to stdout
          --torpassword=          Set the tor control password. This will override the tor password in
                                  the config.
          --tor                   Automatically configure the daemon to run as a Tor hidden service and
                                  use Tor exclusively. Requires Tor to be running.
          --dualstack             Automatically configure the daemon to run as a Tor hidden service IN
                                  ADDITION to using the clear internet. Requires Tor to be running.
                                  WARNING: this mode is not private
          --disablewallet         disable the wallet functionality of the node
          --disableexchangerates  disable the exchange rate service to prevent api queries
          --storage=              set the outgoing message storage option [self-hosted, dropbox]
                                  (default=self-hosted)
          --bitcoincash           use a Bitcoin Cash wallet in a dedicated data directory
          --zcash=                use a ZCash wallet in a dedicated data directory. To use this you must
                                  pass in the location of the zcashd binary.
```

## Documentation

`mylaxxd` exposes an HTTP API which permits high-level interactions on the network and the internal wallet. Find the HTTP API documentation at [https://mylaxx.com](https://mylaxx.com).


