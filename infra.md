# Infra

## Nmap

Network scanner used to discover hosts, services, ports and fingerprint OS/services

- Host discovery (ping, ARP)
- Port scan (SYN/ACK handling)
- Service fingerprint (banner of service)
- OS detection 

SYN scan -> sends SYN and reads responses without completing the handshake
Connect -> completes handshakes

Start a TCP session --> SYN, SYN-ACK, and ACK for SYNchronize, SYNchronize-ACKnowledgement, and ACKnowledge

After DNS lookup and before TLS handshake
