# Networking and File Transfer

A practical reference for inspecting Linux network configuration, troubleshooting connectivity, and transferring files in an authorised lab or assessment.

## Quick Reference

| Task | Command | Note |
|---|---|---|
| Show IP addresses | `ip addr` | Interface addresses |
| Show interface state | `ip link` | Link status |
| Show routing table | `ip route` | Gateways and routes |
| Show resolver configuration | `cat /etc/resolv.conf` | Resolver settings |
| Resolve a hostname | `getent hosts example.com` | Uses system name service |
| Query DNS | `dig example.com` | Requires `dig` |
| Test reachability | `ping -c 4 IP_ADDRESS` | ICMP may be blocked |
| Check route to destination | `ip route get IP_ADDRESS` | Local route decision |
| Test HTTP endpoint | `curl -i http://TARGET` | Shows response headers and body |
| Test TCP port | `nc -vz TARGET PORT` | Requires compatible `nc` |
| Connect by SSH | `ssh user@HOST` | Encrypted remote login |
| Copy with SCP | `scp file user@HOST:/path/` | Uses SSH |
| Download with wget | `wget URL` | Saves a remote file |
| Download with curl | `curl -o file URL` | Choose local filename |
| List listening sockets | `ss -lntup` | Local services |

## 1. Network Interfaces and Routes

### 1.1 Inspect Interfaces

```bash
ip addr
ip link
ip -4 addr
```

| Command | Purpose |
|---|---|
| `ip addr` | Addresses assigned to interfaces |
| `ip link` | Interface state and link information |
| `ip -4 addr` | IPv4 addresses only |
| `ip neigh` | Neighbour table |

Record interface names, addresses, prefix lengths, and interface state.

### 1.2 Inspect Routes

```bash
ip route
ip route get IP_ADDRESS
```

`ip route` shows the routing table. `ip route get` asks the kernel which route it would use for a destination.

## 2. DNS and Name Resolution

### 2.1 Resolver Configuration and Host Lookup

```bash
cat /etc/resolv.conf
getent hosts example.com
```

`getent hosts` uses the system's configured name service, which may involve more than DNS.

### 2.2 Query DNS Records

```bash
dig example.com
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com TXT
```

`dig` queries DNS through its configured resolver behaviour. A failed lookup may be caused by DNS configuration, connectivity, resolver restrictions, or a nonexistent name.

## 3. Connectivity Troubleshooting

### 3.1 Basic Tests

```bash
ping -c 4 IP_ADDRESS
ip route get IP_ADDRESS
nc -vz TARGET PORT
curl -i http://TARGET
```

A failed `ping` does not prove that a host is down. ICMP may be filtered while TCP services remain reachable.

### 3.2 Troubleshooting Checklist

1. Is the destination address correct?
2. Is there a route to the destination?
3. Does name resolution work?
4. Is the service listening on the expected port?
5. Could a firewall or access-control rule be blocking traffic?
6. Is the application using HTTP, HTTPS, or another protocol?

## 4. Listening Ports

### 4.1 Inspect Local Sockets

```bash
ss -lnt
ss -lntu
ss -lntup
ss -ant
```

| Option | Meaning |
|---|---|
| `-l` | Listening sockets |
| `-n` | Numeric addresses and ports |
| `-t` | TCP |
| `-u` | UDP |
| `-a` | All sockets |
| `-p` | Process information, where available |

A listening socket is not necessarily accessible from another network.

## 5. SSH and SCP

### 5.1 Connect with SSH

```bash
ssh username@HOST
ssh -p 2222 username@HOST
ssh -i ~/.ssh/id_ed25519 username@HOST
```

- `-p` specifies the remote port.
- `-i` specifies a private key.

### 5.2 Copy Files with SCP

```bash
# Copy a local file to a remote host
scp file.txt username@HOST:/tmp/

# Copy a remote file to the current directory
scp username@HOST:/path/file.txt .
```

SSH uses encrypted connections. Protect private keys and avoid putting credentials directly into command history or shared notes.

## 6. HTTP Downloads and Uploads

### 6.1 `wget`

```bash
wget http://HOST/file.txt
wget -O downloaded.txt http://HOST/file.txt
```

`-O` specifies the local output filename.

### 6.2 `curl`

```bash
# Display response headers and body
curl -i http://HOST/

# Save response content to a chosen filename
curl -o downloaded.txt http://HOST/file.txt

# Save using the remote filename
curl -O http://HOST/file.txt

# Show response headers only
curl -I http://HOST/
```

Check the response and verify downloaded files before executing them. An HTTP status code does not establish that content is valid or safe.

### 6.3 Uploads

The upload method depends on the destination server. An HTTP upload may require a particular endpoint, request method, and content type. A generic `curl` command will not upload a file successfully unless the server supports that request.

## 7. Netcat

### 7.1 Check Availability and Test a Port

```bash
command -v nc
nc -h
nc -vz TARGET PORT
```

Netcat can create TCP or UDP connections and is useful for basic connectivity checks and simple data transfer.

### 7.2 Simple Lab Listener

In a controlled lab, a listener can receive text:

```bash
nc -lv 9001
```

A client can connect to it:

```bash
nc 127.0.0.1 9001
```

Options vary between OpenBSD Netcat, traditional Netcat, and Ncat. Check local help before relying on a particular flag. Only open listeners on systems and interfaces where you are authorised to do so.

## 8. Socat

### 8.1 Basic Address Syntax

```bash
# Connect standard input/output to a TCP service
socat - TCP4:192.0.2.10:80

# Listen locally and display received data
socat TCP4-LISTEN:9000 STDOUT
```

General form:

```text
socat [options] ADDRESS1 ADDRESS2
```

- `TCP4:IP:PORT` describes a TCP client endpoint.
- `TCP4-LISTEN:PORT` describes a TCP listening endpoint.
- `STDOUT` represents standard output.
- `-` commonly represents standard input/output.

### 8.2 File Transfer Caution

Socat address options and file-opening behaviour matter. A `file:` address with `create` permits file creation but does not automatically send file contents. Verify data direction, file mode, network connectivity, and overwrite behaviour in a lab before use.

## 9. FTP and Firewall Basics

### 9.1 FTP Concepts

When FTP is in scope, understand:

- Anonymous login as a server configuration.
- Directory listing and navigation.
- Upload and download permissions.
- ASCII versus binary transfer modes.
- How server-side permissions affect operations.

FTP may transmit credentials in plaintext. Prefer secure transfer protocols in real environments when available.

### 9.2 `iptables` Rule Inspection

```bash
sudo iptables -L -n -v --line-numbers
sudo iptables -S
```

Rules are evaluated in order. Inserting rules can change behaviour and may interrupt SSH or other connections. Inspect existing rules and ensure a recovery path before making changes.

Example rule syntax from a controlled lab:

```bash
sudo iptables -I INPUT 1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -I INPUT 2 -m conntrack --ctstate INVALID -j DROP
sudo iptables -I INPUT 1 -p icmp --icmp-type echo-request -j DROP
```

These examples modify firewall state; do not run them on a system unless you understand the impact and are authorised to make the change. The ICMP rule shown targets IPv4 echo requests only; it does not disable all ICMP or configure IPv6 ICMPv6.

## 10. Common Mistakes

- A failed `ping` does not prove a host is down.
- DNS resolution and IP connectivity are separate checks.
- A listening port does not prove remote reachability.
- `curl -I` requests headers; it does not download the full response body.
- `curl -o` chooses a local filename; `curl -O` uses the remote filename.
- Netcat options differ between implementations.
- Socat file addresses require careful attention to read/write direction.
- FTP may expose credentials in plaintext.
- Firewall rule order matters; inserting a rule may affect existing connections.

## Related Notes

- [Kali and Shell Basics](01-Kali-and-Shell-Basics.md)
- [Pipes, Redirection and Bash](03-Pipes-Redirection-and-Bash.md)
- [Users, Permissions and Privilege Checks](04-Users-Permissions-and-Privilege-Checks.md)
- [Processes, Services and System Inspection](05-Processes-Services-and-System-Inspection.md)
- [Linux Enumeration Checklist](Linux-Enumeration-Checklist.md)
