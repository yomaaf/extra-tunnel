<b align="center">
<pre>
.______       __    __    ______        _______.___________.
|   _  \     |  |  |  |  /  __  \      /       |           |
|  |_)  |    |  |__|  | |  |  |  |    |   (----`---|  |----`
|      /     |   __   | |  |  |  |     \   \       |  |     
|  |\  \----.|  |  |  | |  `--'  | .----)   |      |  |     
| _| `._____||__|  |__|  \______/  |_______/       |__|     
</pre>
</b>
<p align="center">
  <a href="https://nodei.co/npm/rhost/">
    <img alt="NPM" src="https://nodei.co/npm/rhost.png">
  </a>
</p>

A proxying system, where the *proxy* acts both as a *middle-man* and an
*HTTP server*. This enables *users* to access an HTTP server running
**locally**, through a *public-ip proxy server*, which can be hosted on a
*cloud server*, like *[Heroku]*. The proxy also supports **channels**, other
than HTTP which enables users to access *TCP servers*, like *SSH/FTP*,
running locally.

The system has 3 parts:
- **Proxy**: acts as the proxy server
- **Server**: enables local server to be hosted through *Proxy*
- **Client**: enables local clients to request through *Proxy*

Think of *Proxy* like a *school*. It has multiple *channels*, like a school has
multiple *classrooms*. Each *channel* has a *Server*, like each classroom has a
*class teacher*.. Any number of *Clients* can connect to a *channel* and send
requests to the *Server*, and so can any number of *students* in a *classroom*
ask questions to their *class teacher*.

![rhost](http://oi63.tinypic.com/2uqm5bl.jpg)


## Setup

### Proxy

In order to start, we need a *Proxy* first. Let's set it up:
1. Get *Proxy* to your [GitHub].
    1. Create an account on [GitHub].
    2. Goto [rhost] repository, and fork it.
2. Create *Proxy* application in cloud.
    1. Create an account on [Heroku].
    2. On [Heroku dashboard], create a new app, like `proxywebapp`.
    3. Select the *Deploy* tab, and the choose *GitHub* as deployment method.
    4. In *Connect to GitHub*, type in *rhost* and *Connect*.
    5. In *Manual Deploy*, *Deploy Branch* when *master* is selected.

### Server/Client

We need to install [rhost] locally in order to use it as *Server* or *Client*
(a private-ip *Proxy* would only be useful in testing).

```bash
# to use from command line
npm install -g rhost

# to use from node.js
npm install rhost
```


## Usage

### Host local HTTP server

Assuming your [Heroku] app name is `proxywebapp`, and your local HTTP server is
running on port 80. The following command starts up a *Server*, which acts as a
bridge between your local server `localhost:80` and the *Proxy* `proxywebapp`.
Try opening `https://proxywebapp.herokuapp.com` in your browser, after running
this command.

```bash
rhost server --proxy proxywebapp.herokuapp.com --server 80
```

### Host local SSH server

All *channels* other than default `/` for HTTP are disabled by default. Lets
enable it first by going to *Proxy* setting on [Heroku]:
1. Goto [Heroku dashboard], and then choose *Settings* tab.
2. In *Config Variables*, we need to add one, so select *Reveal Config Vars*.
3. Set *Key* as `KEYS_SSH`, and *Value* as `admin` (or whatever you want).
4. Select *Add*, this restarts the app with new config.
5. You can see app logs at *More -> View Logs*.

Now that we have setup the key for `/ssh` *channel*, it is enabled and we are
ready to setup the server. Assuming your [Heroku] app name is `proxywebapp`,
and your local SSH server is running on port 22. The following command starts
up a *Server*, which acts as a bridge between your local server `localhost:22`
and the *Proxy* `proxywebapp`, on *channel* `/ssh`.

```bash
rhost server -p proxywebapp.herokuapp.com -s 22 --channel /ssh --key admin
```

The common use of SSH is to access the terminal of a remote computer. In our
case, since we are using *Proxy*, we would now be able to access it, not just
from LAN, but from anywhere in the world (with an internet connection). Unlike
HTTP however, *Proxy* is unable to act as an SSH server and hence you cannot
connect directly to it with your SSH client.

To solve this problem, we have a *Client*. Any number of *Clients* can connect
to a *channel* on the *Proxy*. So, on a separate machine, install [rhost] using
the command `npm install -g rhost`, and then start *Client* using the following
command:

```bash
rhost client -p proxywebapp.herokuapp.com -c 22 -n /ssh
```


## Concept

### Proxy

It acts as a server on a single port, and manages communication between
*Clients* and *Servers* through *channels*. Each *Server* registers to a unique
*channel* (like `/` or `/ssh`), and any number of *Clients* can then connect to
the *Proxy* on that *channel*. The *Proxy* also itself acts as a client on
*channel* `/` forwarding any HTTP requests it receives on its port to the
*Server* registered to *channel* `/`.

### Server

It connects to the *Proxy*, and registers to a unique *channel* using a *key*
and a *token*. The *key* must match the one stored on the *Proxy* for that
*channel*. Once registered, the *token* is used to accept *Clients*. *Server*
then acts a multiple local clients for forwarding requests to local server from
specified *channel*, thus making you **feel** as if the *Clients* are running
locally (even if its not). A *Server* registered to *channel* `/` will also
receive *HTTP requests* from *Proxy*, becuase *Proxy* also acts as a *Client*
to *channel* `/`.

### Client

It connects to the *Proxy*, and subscribes to a *channel* using a *token*. This
*token* must match the one provided by the *Server* registered to this
*channel*. *Client* then acts as a local server for forwarding requests of
local clients to specified *channel*, thus making you **feel** as if the
*Server* is running locally (even if its not). Any *Client* can also register
to *channel* `/`, but this is **unnecessary** since you can directly request
the *Proxy* server instead.


## Reference

### Command Line

```bash
$ rhost [<mode>] [options]
# stdin: can be JSON object with keys of channels
# mode: this is 'proxy', 'server', or 'client'
# -p | --proxy: address of proxy
# -s | --server: address of server
# -c | --client: address of client
# -n | --channel: channel to register/subscribe
# -k | --key: key for registering server
# -t | --token: token for subscribing client
# -e | --keys: JSON object with keys of channels
# environment variables are also accepted
# PORT: port number for proxy
# PROXY: address of proxy
# SERVER: address of server
# CLIENT: address of client
# CHANNEL: channel to register/subscribe
# KEY: key for registering server
# TOKEN: token for subscribing client
# KEYS: JSON object with keys of channels
# KEYS_CH1: key for channel /ch1
# KEYS_CH1_CH2: key for channel /ch1/ch2
```

### javascript

```javascript
const rhost = require('rhost');

rhost.<Mode>([<prefix>], [<options>]);
// Mode: this is Proxy, Server, or Client
// prefix: name of Proxy in logs
// the following options are accepted (object)
// proxy: address of proxy ('localhost')
// server: address of server ('localhost:81')
// client: address of client ('localhost:82')
// keys: keys for each allowed channel ({'/': ''})
// channel: channel to register/subscribe ('/')
// key: key for registering server ('')
// token: token for subscribing client ('')
// ping: ping-pong period (8000)
```

```javascript
// start proxy on port 3212
rhost.Proxy(null, {
  'proxy': 'localhost:3212'
});

// start proxy on port 80 with name 'Thug'
rhost.Proxy('Thug', {
  'proxy': '80'
});

// start Proxy on port 80
rhost.Proxy();

// start Proxy on env PORT with a few channel keys
rhost.Proxy(null, {
  'proxy': process.env.PORT
  'keys': {
    '/': 'newhttpchannelkey'
    '/ssh': 'newsshchannelkey'
  }
})
```

```javascript
// start server to Proxy on rhost.herokuapp.com to local server on channel '/'
rhost.Server(null, {
  'proxy': 'rhost.herokuapp.com',
  'server': 'localhost:8080',
  'channel': '/'
});

// start server to Proxy with client token '1234' on channel '/'
rhost.Server('ThugServer', {
  'proxy': 'rhost.herokuapp.com',
  'server': '8080',
  'token': '1234'
});

// start server to Proxy with channel key 'abcd' on channel '/ssh'
rhost.Server(null, {
  'proxy': 'rhost.herokuapp.com',
  'server': '22',
  'key': 'abcd',
  'token': '1234',
  'ping': 8000
});
```

```javascript
// start client to Proxy on rhost.herokuapp.com with local server to channel '/'
rhost.Client(null, {
  'proxy': 'rhost.herokuapp.com',
  'client': 'localhost:9090',
  'channel': '/'
});

// start client to Proxy with client token '1234' to channel '/ssh'
rhost.Client('ThugClient', {
  'proxy': 'rhost.herokuapp.com',
  'client': '22',
  'channel': '/ssh'
  'token': '1234',
  'ping': 8000
});
```


[GitHub]: https://github.com
[Heroku]: https://www.heroku.com
[rhost]: https://github.com/nodef/rhost
[Heroku dashboard]: https://dashboard.heroku.com/apps
