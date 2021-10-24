---
subtitle: Documentation
layout: page
menubar: userdocs_menu
email: niacatn@gmail.com
hero_height: is-fullwidth
hero_darken: true

---
## Command Line Usage

Until headless tracing mode is implemented, usage of rgat from the command line is limited to setting up the tracer for remote tracing connections.

```
  -r address:port, --remote=address:port                              Run rgat in headless network mode (connecting out) which allows the rgat to control tracing from another computer.
                                                                      Requires the address:port of an rgat instance in GUI mode with listening activated.
                                                                      Not compatible with the listening mode optins. --key parameter is mandatory if no preconfigured key is set.
                                                                      This mode does not require a GPU.

  -p [port number], --port=[port number]                              Run rgat in headless network bridge mode (listening) which allows an rgat client to connect and control tracing on
                                                                      this computer.
                                                                      Takes an  optional TCP port to listen on, or chooses a random available port.
                                                                      Not compatible with the 'remote' option. See notes for the --key parameter, which is optional for this mode.
                                                                      This mode does not require a GPU

  -i IP/ID/MAC/name, --interface=IP/ID/MAC/name                       A network interface to use for remote control options (r or p).
                                                                      By default all available interfaces will be used, so it's a good idea to pick the one you will be using.
                                                                      The argument can be an interface name, ID, MAC or IP address.
                                                                      Use without an argument to list valid interfaces.

  -k, --key                                                           Pre-shared key for remote control tracing. This key is stored so it is not required in future invocations.
                                                                      ------Security note------
                                                                       Network tracing is intended to facilitate tracing between VM Host/Guest or between machines on a private analysis
                                                                       network.
                                                                       While rgat expects malicious traffic and  deactivates on receiving a bad key, exposing the listener port to
                                                                       the internet is not advisable. Anyone able to connect to this port with the specified key can execute abitrary
                                                                       code. Standard sensible password choice warnings apply.

  -c ["path_to_config.json"], --configfile=["path_to_config.json"]    A path or current directory filename of a file containing a JSON configuration blob. Values in this configuration
                                                                      can be used instead of (or be overidden by) command line arguments.

  --help                                                              Display this help screen.

  --version                                                           Display version information.
```