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
- Install Rust (if not already).
- Clone the libp2p Rust repository:
  ```sh
  git clone https://github.com/libp2p/rust-libp2p.git
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
  Important: Replace 127.0.0.1 in the multiaddress with the public IP address of your EC2 machine to share it with the private peers. for example if the public IP address is 34.201.123.45:
  ```sh
  /ip4/34.201.123.45/tcp/8080/ws/p2p/12D3KooWDpJ7As7BWAwRMfu1VU2WCqNjvq387JEYKDBj4kx6nXTN
  ```
  This gives you a deterministic and reusable multiaddress that can be shared with peers running on other machines.