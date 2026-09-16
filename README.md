# docker-ssh-tunnel
SSH tunnel that supports remote and local port forwarding in docker.

For local port forwarding the tunnel permits only a single endpoint to be exposed to the
client, and for remote port forwarding the tunnel permits only a single listening port to
be opened. Running shell commands is disallowed.

## Configuration
The tunnel is configured through the following environment variables:

- `TUNNEL_DIRECTION`: Set to either *local or *remote*. *local* is suitable when the client
   wishes to connect to a private service that it cannot reach, and *remote* is suitable
   when the client wishes to expose a private service it can reach to another host.
- `TUNNEL_USER`: The user that the tunnel runs under.
- `TUNNEL_ENDPOINT`: For *local* port forwarding, this should be set to the
   private service that the client wishes to connect to, for example `www.mysite.internal:80`.
   For *remote* port forwarding, this should be set to the wildcard IP and the port that
   the client wishes to open on the ssh-tunnel container, for example `0.0.0.0:80`.
- `TUNNEL_PUBLIC_KEY`: The public key for the user the tunnel runs under.

## Usage
Docker compose example:
```yml
services:
  tunnel:
    image: ghcr.io/william-stacken/ssh-tunnel:latest
    environment:
      TUNNEL_USER: tunnel
      TUNNEL_PUBLIC_KEY: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIDO2tvIOSuhNvK7fkE7gAkUXPbFa5hRujmnH0G1Wdy0 only-used-for-testing
      TUNNEL_DIRECTION: local # or remote
      TUNNEL_ENDPOINT: www.mysite.internal:80
    volumes:
      - my-ssh-config:/etc/ssh
    ports:
      - "127.0.0.1:2222:22"
```
The client sets up local port forwarding using this command:
```sh
ssh -NL "localhost:5555:$TUNNEL_ENDPOINT" "$TUNNEL_USER@$TUNNEL_HOST"
```
Where `5555` is the port to open on the client's machine and `$TUNNEL_ENDPOINT` is the service
reachable from the tunnel container that the client wishes to connect to. After setting up
the tunnel, the client can connect to `localhost:5555` to access `$TUNNEL_ENDPOINT`.

The client can set up remote port forwarding as such:
```sh
ssh -NR "$TUNNEL_ENDPOINT:www.mysite.internal:80" "$TUNNEL_USER@$TUNNEL_HOST"
```
Where `$TUNNEL_ENDPOINT` is the wildcard IP and port that should be opened on the
`$TUNNEL_HOST` and `www.mysite.internal:80` is the private service the client wishes to
expose. After setting up the tunnel, `www.mysite.internal:80` can be reached by connecting
to `$TUNNEL_ENDPOINT` on `$TUNNEL_HOST`.

## Demo

Start by running `cd test && ./setup && docker compose up -d`.

- **Local port forwarding**: Run
  `ssh -NL 5555:private-server:4444 -o "IdentitiesOnly=yes" -i test/keys/id_ed25519 tunneluser@localhost -p 2222`
  in one terminal and then run `nc localhost 5555` in another terminal. You should receive a
  message from a private service.
- **Remote port forwarding**: Run `nc -l 5555` in one terminal and then run
  `ssh -NR 0.0.0.0:4444:localhost:5555 -o "IdentitiesOnly=yes" -i test/keys/id_ed25519 tunneluser@localhost -p 2223`
  in another temrinal. You should receive a message from a client in the terminal that
  ran `nc -l 5555`.

## Publishing
`GITHUB_ACTOR=$YOUR_NAME TAG=1.0 GITHUB_TOKEN=$YOUR_TOKEN ./publish`