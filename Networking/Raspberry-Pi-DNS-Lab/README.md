# Local DNS Troubleshooting with Raspberry Pi 3

![Networking](https://img.shields.io/badge/Networking-DNS-14283F?style=flat-square) ![Lab executed](https://img.shields.io/badge/Lab-Verified-059669?style=flat-square)

[← Networking](../README.md) · [INC-003 simulated case](../../IT-Support/Service-Desk-Incident-Management/incidents/INC-003-DNS-Resolution.md) · [Versión en español](https://github.com/fefemora/it-portfolio-es/blob/main/Networking/Raspberry-Pi-DNS-Lab/README.md)

I set up a local DNS server on a Raspberry Pi 3 Model B and queried it from Windows. I wanted to see what a missing name looks like from the client when the DNS server is still responding. I temporarily removed an A record, observed the response, and checked that the name resolved again after I restored it.

## Environment and configuration

- **Server** · Raspberry Pi 3 Model B, Raspbian GNU/Linux 13 (Trixie), `dnsmasq`, Wi-Fi (`wlan0`).
- **Resources before installation** · 767 MiB of available RAM and 11 GiB of free space on the microSD card.
- **Client** · Windows, PowerShell, and `nslookup`.

I confirmed that port 53 was free before installing `dnsmasq`. The initial interface-based configuration also listened on IPv6. I replaced `interface=wlan0` with `listen-address=192.168.1.17` and used `sudo ss -luntp 'sport = :53'` to confirm that the process was listening only on that IPv4 address over UDP and TCP. The lab service ran without DHCP or upstream DNS forwarding.

`/etc/dnsmasq.d/pi3-lab.conf`

```ini
listen-address=192.168.1.17
bind-dynamic
no-dhcp-interface=wlan0
no-resolv
no-hosts
local=/pi-lab.test/
host-record=intranet.pi-lab.test,192.168.1.17
```

## Windows validation

I used the same direct query to the Pi for all three checks.

```powershell
nslookup -type=A intranet.pi-lab.test 192.168.1.17
```

### 1. Baseline

The A record returned the configured IPv4 address.

```text
Servidor:  intranet.pi-lab.test
Address:  192.168.1.17

Nombre:  intranet.pi-lab.test
Address:  192.168.1.17
```

### 2. Controlled failure

I commented out the `host-record` line and restarted `dnsmasq`. The server returned `NXDOMAIN` (`Non-existent domain`).

```text
Servidor:  UnKnown
Address:  192.168.1.17

*** UnKnown no encuentra intranet.pi-lab.test: Non-existent domain
```

### 3. Recovery

I restored the `host-record` line, restarted `dnsmasq`, and ran the query again. The A record returned the expected address.

```text
Servidor:  intranet.pi-lab.test
Address:  192.168.1.17

Nombre:  intranet.pi-lab.test
Address:  192.168.1.17
```

## What the test showed

`NXDOMAIN` confirmed that the DNS server responded during the controlled failure. In this case, the missing record caused the failure. The `UnKnown` label refers to the server name that `nslookup` tries to obtain through a reverse lookup. I used the `NXDOMAIN` response for the diagnosis.

The validation covered a direct A-record query from one Windows client on the local network and the subsequent recovery. The corporate INC-003 scenario remains a simulation.
