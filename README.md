## Description

This repository demonstrates a direct WebRTC connection between two private JavaScript peers — each located behind separate NATs or VPNs — using libp2p. The goal is to showcase real-world feasibility of NAT traversal and hole punching with minimal relay involvement.

Key features include:
- Private-to-private WebRTC connections between NAT-ed or VPN-ed environments.
- Use of a single relay (either a JavaScript or Rust-based libp2p relay) to assist with initial discovery or fallback
- Exploration of interoperability across multiple libp2p implementations (JavaScript ↔ Rust)

## Usage

To run this experiment, you'll need three separate machines (or EC2 instances) — one for the relay and one for each peer — to simulate real-world NAT traversal conditions.

All machines should be running Ubuntu (e.g., on AWS EC2).

### 1. Set Up the Relay (Rust):
- Launch an EC2 Ubuntu instance for the relay.
  💡 To ensure the relay's IP address doesn’t change on reboot, [associate a static Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
 with your instance.
  🔐 Make sure to [open security group incoming TCP connections](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html#adding-security-group-rule) for the required ports (4884, 8080, 8081).
- Install Rust (if not already).
- Clone the libp2p Rust repository:
  ```sh
  git clone https://github.com/libp2p/rust-libp2p.git &&
  cd rust-libp2p/examples/relay-server/
  ```
- Run the relay server with WebSocket and WebRTC support:
  ```sh
  RUST_LOG=info cargo run -- --port 4884 --secret-key-seed 0 --websocket-port 8080 --webrtc-port 8081
  ```
  This command starts the relay node with:
  TCP on port 4884
  WebSocket on port 8080
  WebRTC on port 8081
  You will see output similar to:
  ```sh
  Listening on /ip4/127.0.0.1/tcp/8080/ws/p2p/12D3KooWDpJ7As7BWAwRMfu1VU2WCqNjvq387JEYKDBj4kx6nXTN
  ```
  To share this address with peers, replace 127.0.0.1 with your EC2 instance's public IP. You can find it:
  In the EC2 Console under "Instances" → "Public IPv4 address"
  Or from the command line on the instance:
  ```sh
  curl http://checkip.amazonaws.com
  ```
  For example, if the public IP address is 34.201.123.45, the full multiaddress will be:
  ```sh
  /ip4/34.201.123.45/tcp/8080/ws/p2p/12D3KooWDpJ7As7BWAwRMfu1VU2WCqNjvq387JEYKDBj4kx6nXTN
  ```
  This gives you a deterministic and reusable multiaddress that can be shared with peers running on other machines.

### 2. Set Up the web server for the Private Peers (JavaScript – Web Server)
- Launch a second EC2 Ubuntu instance to act as a browser-accessible web server with the js peer-to-peer example.
💡 To ensure the web server's IP address doesn’t change on reboot, [associate a static Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)
with your instance.
🔐 Make sure to [open security group incoming TCP connections](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html#adding-security-group-rule) for the required port 5173 — this is the port the Vite dev server listens on.
Open the following inbound rule in your EC2 instance’s security group:

| Type       | Protocol | Port Range | Source    |
| ---------- | -------- | ---------- | --------- |
| Custom TCP | TCP      | 5173       | 0.0.0.0/0 |

- Connect to the instance via SSH and install Git + Node.js (if not already installed):
- Clone the example JavaScript peer:
```sh
git clone https://github.com/libp2p/js-libp2p-example-webrtc-private-to-private.git &&
cd js-libp2p-example-webrtc-private-to-private
```
- Make sure the dev server binds to all network interfaces. Open package.json and verify that the start script is:
```sh
"start": "vite --host 0.0.0.0"
```
- Follow the [official instructions](https://github.com/libp2p/js-libp2p-example-webrtc-private-to-private/tree/main?tab=readme-ov-file#running-the-clients) to run the client
- Your browser peer will now be accessible via:
```sh
http://<public-ip>:5173
```

### 3. Start the First Private Peer (Listener)
Use a third machine — either a local computer or a cloud instance — to run the first private peer, which will act as the listener.

✅ Make sure the machine is behind a NAT (e.g., home network or VPN). You can optionally test NAT traversal more strictly by routing this machine through VPN

- In your browser on this machine, open the web interface served by the peer you set up in Section 2 (the web server):
```sh
http://<web-server-public-ip>:5173
```
- In the UI:

Locate the “Remote multiaddress” field.

Paste the multiaddress of the relay (from Section 1), for example:
```sh
/ip4/<relay-ip>/tcp/8080/ws/p2p/<relay-peer-id>
```
and press connect
- Once the listener peer connects to the relay, it will print a listening multiaddress that looks like this:
```sh
/ip4/<relay-ip>/tcp/8080/ws/p2p/<relay-peer-id>/p2p-circuit/p2p/<listener-peer-id>
```
- Copy this full multiaddress — you'll need it for the next step (the dialer peer) to connect.
### 4. Start the Second Private Peer (Dialer)
- On a fourth machine (or reuse another NATed/VPN machine), open a browser to run the second private peer, which will act as the dialer.
- Visit the same URL as before — the web server hosted on the first peer (from Section 2):
```sh
http://<web-server-public-ip>:5173
```
- In the UI:
Paste the multiaddress of the listener (from Section 3), for example:
```sh
/ip4/<relay-ip>/tcp/8080/ws/p2p/<relay-peer-id>/p2p-circuit/p2p/<listener-peer-id>
```
- Click “Connect” to initiate the connection.
- Once connected, you can start chatting directly between the two peers using the UI.