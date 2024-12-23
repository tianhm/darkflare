# DarkFlare - TCP-over-CDN Tunnel

A stealthy command line tool to create TCP-over-CDN(http) tunnels that keep your connections cozy and comfortable. Now with public test relay servers!

Join the Discord server for support and discussion: https://discord.gg/7wVKeP88

## 🤔 What is this sorcery?

DarkFlare is a clever little tool that disguises your TCP traffic as innocent HTTPS requests, letting them pass through corporate firewalls like a VIP at a nightclub. It's like a tunnel, but with more style and less dirt.

It has two parts: a client-side proxy (darkflare-client) that encodes TCP data into HTTPS requests and sends it to a Cloudflare-protected domain, and a server-side proxy (darkflare-server) that decodes the requests and forwards the data to a local service (like SSH on port 22). It’s protocol-agnostic, secure, and uses Cloudflare's encrypted infrastructure, making it stealthy and scalable for accessing internal resources or bypassing network restrictions.

When using this remember the traffic over the tunnel is only as secure as the Cloudflare protection. Use your own encryption.

## 🧱 Why CDNs?
Services like Cloudflare, Akamai Technologies, Fastly, and Amazon CloudFront are not only widely accessible but also integral to the global internet infrastructure. In regions with restrictive networks, alternatives such as CDNetworks in Russia, ArvanCloud in Iran, or ChinaCache in China may serve as viable proxies. These CDNs support millions of websites across critical sectors, including government and healthcare, making them indispensable. Blocking them risks significant collateral damage, which inadvertently makes them reliable pathways for bypassing restrictions.

## ⛓️‍💥 Stop Network Censorship
Internet censorship is a significant issue in many countries, where governments restrict access to information by blocking websites and services. For instance, China employs the "Great Firewall" to block platforms like Facebook and Twitter, while Iran restricts access to social media and messaging apps. In Russia, authorities have intensified efforts to control information flow by blocking virtual private networks (VPNs) and other tools that citizens use to bypass censorship.

AP NEWS
 In such environments, a tool that tunnels TCP traffic over HTTP(S) through a Content Delivery Network (CDN) like Cloudflare can be invaluable. By disguising restricted traffic as regular web traffic, this method can effectively circumvent censorship measures, granting users access to blocked content and preserving the free flow of information.

```
                                FIREWALL/CENSORSHIP
                                |     |     |     |
                                v     v     v     v

[Client]──────┐                ┌──────────────────┐                ┌─────────[Target Service]
              │                │                  │                │       (e.g., SSH Server)
              │                │   CLOUDFLARE     │                │tcp      localhost:22
              │tcp             │     NETWORK      │                │
[darkflare    │                │                  │                │ [darkflare
 client]──────┼───HTTPS───────>│ (looks like      │─-HTTPS-───────>│  server]
localhost:2222│                │  normal traffic) │                │ :8080
              │                │                  │                │
              └────────────────┼──────────────────┼────────────────┘
                               │                  │
                               └──────────────────┘

Flow:
1. TCP traffic ──> darkflare-client
2. Wrapped as HTTPS ──> Cloudflare CDN (or any CDN)
3. Forwarded to ──> darkflare-server
4. Unwrapped back to TCP ──> Target Service
```

### CA Root Notes
In some cases the direct server might fail TLS. If that happens you can use the CDN server or make sure you have the CA certs:

For Debian-based Systems (e.g., Ubuntu)

   sudo apt install ca-certificates

For Red Hat-based Systems (e.g., CentOS, Fedora)

   sudo yum install ca-certificates



##  Usecases
ssh, rdp, or anything tcp to bypass restrictive firewalls or state controled internet.

Tunneling ppp or other vpn services that leverage TCP.

darkflare-server can launch applications like sshd or pppd. Note that there are issues with host keys and certificate validation on sshd if you don't configure it properly.

Linux's popular pppd daemon will also not run as non-root in some cases, which would require a more complex configuration with sudo.

Breaking past blocked sites! 

[How to use NordVPN over TCP](https://support.nordvpn.com/hc/en-us/articles/19683394518161-OpenVPN-connection-on-NordVPN#:~:text=With%20NordVPN%2C%20you%20can%20connect,differences%20between%20TCP%20and%20UDP. "Configure NordVPN over TCP")

## NordVPN

1. Download the OpenVPN client (cli is better) 
2. Under Manual setup in your NordVPN web account download the .ovpn file for TCP
3. Also in Manual setup select username and password authentication.
4. Edit the .ovpn file and change the IP and port to your darkflare server IP and Port.
5. Configure darkflare-server to use the IP and port defined in the .ovpn file.
6. Import the .ovpn file to OpenVPN and setup your username and password.

I did provide an ./examples/nordvpn.ovpn for you to use. Also two scrips for up/down to fix some of the routing issues.

Using the OpenVPN commandline client you can embed the username, password, and it runs the scripts properly for you:
```
& openvpn --config 127.0.0.1.tcp2222.ovpn --script-security 2
```

Note: OpenVPN by default screws up the default gateway/route. For testing purposes I added: pull-filter ignore "redirect-gateway" to the .ovpn file. That allows me to force the tunnel to not change the routing. The routing can be fixed by the OpenVPN-up.sh and OpenVPN-down.sh scripts. This is due to the fact that the VPN is connecting to the whole CDN range of IP addresses. 


## 🔐 Few Obscureation Techniques

Requests are randomized to look like normal web traffic with jpg, php, etc... with random file names.

Client and server headers are set to look like normal web traffic. 

If you have other ideas please send them my way.


## 🌩️ Cloudflare Configuration 
Add your new proxy hostname into a free Cloudflare account.

Setup your origin rules to send that host to the origin server (darkflare-server) via the proxy port you choose. 

I used 8080 with a Cloudflare proxy via HTTP for the firs test. Less overhead.

## ✨ Features

- **Sneaky TCP Tunneling**: Wraps your TCP connections in a fashionable HTTPS outfit
- **Cloudflare Integration**: Because who doesn't want their traffic to look like it's just visiting Cloudflare?
- **Debug Mode**: For when things go wrong and you need to know why (spoiler: it's always DNS)
- **Session Management**: Keeps your connections organized like a Type A personality
- **TLS Security**: Because we're sneaky, not reckless
- **Client-controlled destination addressing**: The destination (-d) is now specified on the client side and securely transmitted to the server
- **Base64 encoded destination transmission**: The server no longer requires a destination parameter (-d has been removed)
- **Reverse Proxy Support**: The client now supports SOCKS5 and HTTP(s) proxies.
- **Custom 302**: Server now has defined 302 redirects for non-auth users.
- **stdin:stdout**: stdin:stdout client mode for client to avoid firewall restrictions and binding to local ports.
- **Fileless Execution on Windows**: PowerShell script to execute the client without saving any files to disk.

## 🚀 Quick Start

### Installation

1. Download the latest release from the [GitHub Releases page](https://github.com/doxx/darkflare/releases)
   - Choose the appropriate binary for your system:
     - `darkflare-client-darwin-arm64` - macOS Apple Silicon
     - `darkflare-client-darwin-amd64` - macOS Intel
     - `darkflare-client-linux-amd64` - Linux x64
     - `darkflare-client-windows-amd64.exe` - Windows x64
     - `darkflare-server-*` - corresponding server binaries
2. Verify the checksums against `checksums.txt` (recommended)
3. Make the binaries executable (Unix systems):
```bash
chmod +x darkflare-client-* darkflare-server-*
```

### Running the Client
```bash
./darkflare-client -l 2222 -t https://cdn.miami.us.doxx.net:443 -d <my ssh server>:22
```

Or with direct mode:
```bash
./darkflare-client -l 2222 -t https://direct.miami.us.doxx.net:443 -d <my ssh server>:22
```

Add `-debug` flag for debug mode

### Notes
If you want to debug and go directly to the psudo server you can use the `-allow-direct` flag on the server.

You can replace the doxx.net server with your own and setup your own server:

### Running the Server

```bash
# HTTPS Server (recommended for production)
./darkflare-server -o https://direct.miami.us.doxx.net:443 -c /path/to/cert.pem -k /path/to/key.pem

# HTTP Server (for testing)
./darkflare-server -o http://direct.miami.us.doxx.net:8080 -allow-direct
```

### Notes
- The `-allow-direct` flag allows direct connections without Cloudflare headers (not recommended for production)
- Debug mode (`-debug`) provides verbose logging of connections and data transfers
- Under SSL/TLS configuration in Cloudflare you need to set ssl encryption mode to Full.

### SSL/TLS Certificates

For HTTPS mode, you'll need to obtain origin certificates from Cloudflare:

1. Log into your Cloudflare dashboard
2. Go to SSL/TLS > Origin Server
3. Create a new certificate (or use an existing one)
4. Download both the certificate and private key
5. When starting the server in HTTPS mode, provide both certificate files:

Note: Keep your private key secure and never share it. The certificate provided by Cloudflare is specifically for securing the connection between Cloudflare and your origin server.

### Testing the Connection
```bash
ssh user@localhost -p 2222
```

## 🔌 stdin:stdout Client Mode

DarkFlare now supports stdin:stdout mode, allowing you to use the client without binding to local ports. This is particularly useful when:
- You don't have privileges to bind to local ports
- Local firewalls restrict port binding
- You want to integrate with SSH's ProxyCommand

### Using with SSH
The most common use case is with SSH's ProxyCommand. Add to your ~/.ssh/config:
```bash
Host my-remote
    HostName remote-server.example.com
    User myuser
    ProxyCommand darkflare-client -l stdin:stdout -t cdn.example.com -d localhost:22
```

Then simply connect:
```bash
ssh my-remote
```

Or use directly from the command line:
```bash
ssh -o ProxyCommand="darkflare-client -l stdin:stdout -t cdn.example.com -d localhost:22" user@remote-server
```

### Benefits
- No local port binding required
- Works without root/admin privileges
- Bypasses local firewall restrictions
- Integrates seamlessly with SSH and other tools
- Maintains end-to-end encryption
- Traffic still appears as normal HTTPS to observers

## 🧙 Fileless Execution

DarkFlare supports fileless execution on Windows systems using PowerShell, allowing you to run the client without saving any files to disk. This is particularly useful in restricted environments where:
- You don't have write permissions to the local system
- Security policies prevent executing downloaded binaries
- You need to leave no traces on the filesystem
- You want to run the client without installation or cleanup

### PowerShell Memory Execution
Save this as `memory-exec.ps1` or download from examples/:
```powershell
# See examples/memory-exec.ps1 in the repository
param (
    [Parameter(Mandatory=$true)]
    [string]$t,
    [Parameter(Mandatory=$true)]
    [string]$d,
    [Parameter(Mandatory=$false)]
    [string]$l = "stdin:stdout",
    [Parameter(Mandatory=$false)]
    [string]$p
)

$url = "https://github.com/doxx/darkflare/releases/latest/download/darkflare-client-windows-amd64.exe"
$webClient = New-Object System.Net.WebClient
$bytes = $webClient.DownloadData($url)
$assembly = [System.Reflection.Assembly]::Load($bytes)
$args = @("-l", $l, "-t", $t, "-d", $d)
if ($p) { $args += @("-p", $p) }
$assembly.EntryPoint.Invoke($null, @(,[string[]]$args))
```

### Usage Examples

1. Direct SSH connection using ProxyCommand:
```bash
ssh -o ProxyCommand="powershell -ExecutionPolicy Bypass -File memory-exec.ps1 -t cdn.example.com -d localhost:22" user@remote
```

2. One-liner for immediate execution (no script file needed):
```powershell
$script = (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/doxx/darkflare/main/examples/memory-exec.ps1'); 
powershell -Command $script -t cdn.example.com -d localhost:22
```

3. With a SOCKS5 proxy:
```powershell
powershell -ExecutionPolicy Bypass -File memory-exec.ps1 -t cdn.example.com -d localhost:22 -p socks5://proxy:1080
```

### Benefits
- **No Installation Required**: Run directly from memory without installing
- **No Filesystem Traces**: Leaves no artifacts on the local system
- **Bypass Restrictions**: Works in environments with strict file execution policies
- **Easy Cleanup**: No files to remove after use
- **Latest Version**: Always downloads the latest release
- **Portable**: Can be run from any PowerShell prompt with internet access

### Security Considerations
- Only download from trusted sources over HTTPS
- Consider adding checksum verification for enhanced security
- Be aware that some security software may detect/block memory execution
- Use only in environments where you have permission to do so
- The binary is still downloaded, just not saved to disk
- Network administrators may still see the download traffic

### SSH Configuration
For persistent SSH configuration, add to your `~/.ssh/config`:
```
Host remote.example.com
    ProxyCommand powershell -ExecutionPolicy Bypass -File C:/path/to/memory-exec.ps1 -t cdn.example.com -d localhost:22
```

Or for truly fileless operation:
```
Host remote.example.com
    ProxyCommand powershell -Command "$script = (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/doxx/darkflare/main/examples/memory-exec.ps1'); powershell -Command $script -t cdn.example.com -d localhost:22"
```

### Linux/Unix Memory Execution
On Linux and Unix-like systems, you can use curl and bash to achieve similar fileless execution:

```bash
# Basic usage with curl
curl -s https://github.com/doxx/darkflare/releases/latest/download/darkflare-client-linux-amd64 | bash -s -- -l stdin:stdout -t cdn.example.com -d localhost:22

# Direct SSH ProxyCommand usage
ssh -o ProxyCommand="curl -s https://github.com/doxx/darkflare/releases/latest/download/darkflare-client-linux-amd64 | bash -s -- -l stdin:stdout -t cdn.example.com -d localhost:22" user@remote

# With a SOCKS5 proxy
curl -s https://github.com/doxx/darkflare/releases/latest/download/darkflare-client-linux-amd64 | bash -s -- -l stdin:stdout -t cdn.example.com -d localhost:22 -p socks5://proxy:1080
```

For macOS, replace `linux-amd64` with `darwin-amd64` (Intel) or `darwin-arm64` (Apple Silicon).

### SSH Configuration for Unix Systems
Add to your `~/.ssh/config`:
```
Host remote.example.com
    ProxyCommand curl -s https://github.com/doxx/darkflare/releases/latest/download/darkflare-client-linux-amd64 | bash -s -- -l stdin:stdout -t cdn.example.com -d localhost:22
```

### Security Note for Unix Systems
While this method works, it's important to note:
- The binary is executed with your current user permissions
- Consider using checksum verification for enhanced security:
```bash
# Verify checksum before execution
curl -s https://github.com/doxx/darkflare/releases/latest/download/checksums.txt | grep linux-amd64 | sha256sum -c - && \
curl -s https://github.com/doxx/darkflare/releases/latest/download/darkflare-client-linux-amd64 | bash -s -- [options]
```


## 📖 Command Line Reference

### Client Usage
```
DarkFlare Client - TCP-over-CDN tunnel client component
(c) 2024 Barrett Lyon

Usage:
  darkflare-client [options]

Options:
  -l        Local port or stdin:stdout for ProxyCommand mode
            Format: <port> or stdin:stdout
            Examples: 2222 or stdin:stdout

  -t        Target URL for the DarkFlare server
            Format: [http|https]://host[:port]
            Default port: 443 for HTTPS, 80 for HTTP

  -d        Destination address to connect to
            Format: host:port
            Example: localhost:22

  -debug    Enable detailed debug logging
            Shows connection details and errors

  -p        Proxy URL (http://host:port or socks5://host:port)
            Optional SOCKS5 or HTTP proxy for outbound connections

Examples:
  Basic port forwarding:
    darkflare-client -l 2222 -t cdn.example.com -d localhost:22

  SSH ProxyCommand mode:
    ssh -o ProxyCommand="darkflare-client -l stdin:stdout -t cdn.example.com -d localhost:22" user@remote

  Through a SOCKS5 proxy:
    darkflare-client -l 2222 -t cdn.example.com -d localhost:22 -p socks5://proxy:1080

Notes:
  - Proxy authentication is supported via URL format user:pass@host
  - SOCKS5 variant will resolve hostnames through the proxy
  - Debug mode will show proxy connection details and errors
```

### Server Usage
```
DarkFlare Server - TCP-over-CDN tunnel server component
(c) 2024 Barrett Lyon

Usage:
  darkflare-server [options]

Options:
  -o        Listen address for the server
            Format: proto://[host]:port
            Default: http://0.0.0.0:8080

  -allow-direct
            Allow direct connections not coming through Cloudflare
            Default: false (only allow Cloudflare IPs)

  -c        Path to TLS certificate file
            Default: Auto-generated self-signed cert

  -k        Path to TLS private key file
            Default: Auto-generated with cert

  -debug    Enable detailed debug logging
            Shows connection details and errors

  -s        Silent mode
            Suppresses all non-error output

  -redirect Custom URL to redirect unauthorized requests
            Default: GitHub project page

  -override-dest
            Override client destination with server-side setting
            Format: host:port
            Default: Use client-provided destination

Examples:
  Basic setup:
    darkflare-server -o http://0.0.0.0:8080

  With custom TLS certificates:
    darkflare-server -o https://0.0.0.0:443 -c /path/to/cert.pem -k /path/to/key.pem

  Debug mode with metrics:
    darkflare-server -o http://0.0.0.0:8080 -debug

Notes:
  - Server accepts destination from client via X-Requested-With header
  - Destination validation is performed for security
  - Use with Cloudflare as reverse proxy for best security
```

## ⚠️ Security Considerations

- Always use end-to-end encryption for sensitive traffic
- The tunnel itself provides obscurity, not security
- Monitor your Cloudflare logs for suspicious activity
- Regularly update both client and server components

## ⚠️ Disclaimer

This tool is for educational purposes only. Please don't use it to bypass your company's firewall - your IT department has enough headaches already.

## 🤝 Contributing

Found a bug? Want to add a feature? PRs are welcome! Just remember:
- Keep it clean
- Keep it clever

## 📜 License

MIT License - Because sharing is caring, but attribution is nice.

---
*Built with ❤️ and a healthy dose of mischief*

Join the Discord server for support and discussion: https://discord.gg/7wVKeP88
