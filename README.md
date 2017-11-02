# TCP Tunnels Through Azure Relay Hybrid Connections for CloudFoundry

This sample is an adaptation of the [hyco websocket tunnel](https://github.com/Azure/azure-relay/tree/master/samples/Hybrid%20Connections/Node/hyco-websocket-tunnel) to be used in a Cloud Foundry environment. The original itself is a fork and adaptation of the node-websocket-tunnel](https://github.com/sstur/node-websocket-tunnel)
project by [Simon Sturmer](https://github.com/sstur). 

For simplicity, the additional security has been removed and the configuration was moved to a config file instead of command line arguments. In addition, to play well with CloudFoundry, it needs to listen to a socket so that CloudFoundry can make sure the process is still responsive. We added that. 

## Architecture
This architecture was used as a Proof of Concept to show how one can access a database in a [General Electric Predix](https://www.predix.io/) environment. In our situation we linked to a Predix Postgres database from a an on-prem SQL server to insert records in the Postgres database. 

Note, that this solution as-is does not work yet when the SQL server is behind a proxy. 

## Usage
Set up a `config.json` file like so:

``` JSON
{
  "ns": "{Namespace}.servicebus.windows.net",
  "path": "{Hyrid Connection name}",
  "keyrule": "{Shared access policy name}",
  "key": "{Shared access policy key}"
}
```

### Server
On a server, run `server.js`. The SAS rule name and key need to grant "Listen" permission for that path on the Server. 

If you want to run this in a Cloud Foundry environment, I added the required yml file. You can just push it with [cf push](https://docs.cloudfoundry.org/buildpacks/node/index.html).

### Client
On a client, run `connect.js`. The SAS rule name and key on the client need to grant "Send" permission. Here I shared the key between client and server for simplicity, but you'd want to change that. 

You are presented with a command shell where you can create and destroy tunnels, e.g:

``` cmd
tunnel 12345 myremoteurl.com:12346
```

This will tell the client to listen on port 12345 and tunnel all the traffic to port 12346 on myremoteurl.com. Note that the tunnel server we started previously needs to be able to reach myremoteurl. It does `not` have to be on the same system. 

> ** Security note from the original sample:**
> This sample (not Hybrid Connections per-se) permits any authorized client to establish a tunnel 
> to any TCP network destination that the server can connect to. The client program "connect.js"
> will expose a raw TCP connectivity path. Be aware that once you have started the *connect.js* 
> program with a valid send key *and* have explicitly 
> opened the tunnel, there is no further protection mechanism for the destination 
> other than what it itself provided as a native security model. 