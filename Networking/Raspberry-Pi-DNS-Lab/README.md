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

## Follow-up — Automated monitoring with systemd

I automated the query to the same DNS server with `dig` and a systemd timer. The check runs on the Raspberry Pi and records the outcome in the local journal. It reads the DNS response status and the `dig` exit code separately. `NXDOMAIN` is a valid server response, while exit code 9 indicates that the query received no DNS reply.

### Installed configuration

I installed `bind9-dnsutils` and saved this executable as `/usr/local/bin/pi3-dns-check` with mode `755`.

```sh
#!/bin/sh

server=192.168.1.17
name=intranet.pi-lab.test.
expected=192.168.1.17

reply=$(/usr/bin/dig -r "@$server" "$name" A \
    +time=2 +tries=1 +noall +comments +answer 2>&1)
result=$?

case "$result" in
    0) ;;
    9) echo "DNS_FAIL reason=NO_REPLY name=$name server=$server"; exit 1 ;;
    *) echo "DNS_FAIL reason=DIG_ERROR code=$result name=$name"; exit 1 ;;
esac

status=$(printf '%s\n' "$reply" | sed -n 's/^.*status: \([A-Z0-9]*\),.*$/\1/p')
if [ "$status" != NOERROR ]; then
    echo "DNS_FAIL reason=${status:-UNKNOWN_STATUS} name=$name server=$server"
    exit 1
fi

address=$(printf '%s\n' "$reply" | awk -v name="$name" '$1 == name && $4 == "A" {print $5}')
if [ "$address" != "$expected" ]; then
    echo "DNS_FAIL reason=UNEXPECTED_ANSWER name=$name expected=$expected"
    exit 1
fi

echo "DNS_OK name=$name server=$server answer=$address"
```

`/etc/systemd/system/pi3-dns-check.service`

```ini
[Unit]
Description=Raspberry Pi lab DNS check

[Service]
Type=oneshot
DynamicUser=yes
ExecStart=/usr/local/bin/pi3-dns-check
```

`/etc/systemd/system/pi3-dns-check.timer`

```ini
[Unit]
Description=Run the lab DNS check every minute

[Timer]
OnActiveSec=5s
OnUnitActiveSec=1min
AccuracySec=1s

[Install]
WantedBy=timers.target
```

I loaded the units with `sudo systemctl daemon-reload` and enabled the timer with `sudo systemctl enable --now pi3-dns-check.timer`. The service runs one query per activation under a dynamic systemd user.

### Tests and evidence

First I commented out the test A record, restarted `dnsmasq`, and waited for the scheduled result. I restored the record and restarted the service to verify recovery. For the no-reply test, I scheduled service restoration before stopping the resolver:

```bash
sudo systemd-run --unit=pi3-dns-restore --on-active=90s \
  --timer-property=AccuracySec=1s \
  /usr/bin/systemctl start dnsmasq.service &&
sudo systemctl stop dnsmasq.service
```

| Local time (UTC−03:00) | Observed evidence |
|---|---|
| 19:15:56 and 19:16:57 | `DNS_OK`, `answer=192.168.1.17` |
| 19:17:58 | `DNS_FAIL reason=NXDOMAIN` |
| 19:18:59 | `DNS_OK`, `answer=192.168.1.17` |
| 19:20:17 | systemd recorded `dnsmasq.service` stopping. |
| 19:21:00 | `DNS_FAIL reason=NO_REPLY` |
| 19:21:47 | The transient recovery service brought `dnsmasq.service` back up. |
| 19:24:03 | `DNS_OK`, `answer=192.168.1.17` in the later excerpt. |

The test ran on November 25, 2026. The `dnsmasq` journal shows 90 seconds between shutdown and startup. The 19:24:03 `DNS_OK` is the later check visible in the excerpt. The first successful query after startup was not established.

Evidence query:

```bash
sudo journalctl -u pi3-dns-check.service \
  --since '2026-11-25 19:15:00' \
  --grep='DNS_(OK|FAIL)' -o short-iso --no-pager
```

The validation covers an A-record query from the same host as the resolver. It checks the local DNS service response. Availability from another network host requires an external probe.

## Follow-up — DNS backup and restore drill

I backed up the resolver configuration, the DNS check script, and the systemd service and timer as relative paths in a compressed archive. The backup scope was limited to these four lab files:

- `etc/dnsmasq.d/pi3-lab.conf`
- `usr/local/bin/pi3-dns-check`
- `etc/systemd/system/pi3-dns-check.service`
- `etc/systemd/system/pi3-dns-check.timer`

1. **Backup on the Pi.** I created `pi3-dns-20261125-200756.tar.gz` with `tar`, listed its contents, and verified its separate SHA-256 manifest with `sha256sum -c`. The result was `pi3-dns-20261125-200756.tar.gz: OK`.
2. **Off-device copy.** I transferred the archive and its manifest to Windows over SFTP with PSCP using the existing PuTTY key. Comparing the manifest with `Get-FileHash -Algorithm SHA256` returned `True`.
3. **Restore from the Windows copy.** I transferred that archive back under a different name, compared its bytes with `cmp`, and extracted it into a new staging directory. I checked the recovered configuration with `dnsmasq --test --conf-file` and the recovered script with `sh -n`. The completion message was conditional on all checks succeeding.

```text
dnsmasq: syntax check OK.
Restauración de prueba validada en: /home/<usuario>/pi3-dns-restore.7Btq9O
```

The username in this excerpt was redacted. The drill verified transfer integrity, extraction, and DNS and shell syntax. The restored files stayed in the staging directory while the live configuration stayed in place.

### Functional check with the restored DNS configuration

After extracting the archive copied back from Windows, I made a test copy of its `pi3-lab.conf`. I changed the listener to `127.0.0.1` and started a separate `dnsmasq` process on port `1053` in the foreground with a 120-second timeout. The test kept the original A record and the active resolver configuration.

```text
dnsmasq: syntax check OK.
dnsmasq: started, version 2.91 cachesize 150
```

From a second PuTTY session, I queried the test instance first and the active resolver second. The query commands and answers below are shown as separate pairs for clarity.

```console
$ dig @127.0.0.1 -p 1053 intranet.pi-lab.test. A +short +time=2 +tries=1
192.168.1.17
$ dig @192.168.1.17 -p 53 intranet.pi-lab.test. A +short +time=2 +tries=1
192.168.1.17
```

The restored configuration served the expected A record on the test port. The active resolver also answered the check on port 53. This verifies the restored DNS configuration in an isolated process. The recovered systemd service and timer remain outside the functional check.

## Follow-up — Windows probe and state-change alerts

I ran a probe from a second host on the network. PowerShell queried the A record for `intranet.pi-lab.test.` directly at `192.168.1.17` and compared the reply with the expected address. The loop waits ten seconds between queries. The console shows every `CHECK`. `Tee-Object` appends the initial `BASELINE` and state changes (`ALERT` or `RECOVERY`) to a local file.

### Controlled test

I removed the test A record and restored it after observing two consecutive failures from Windows. The console output shows one alert at **23:32:02**, a second failed check without another alert at **23:32:12**, and one recovery notice at **23:32:22** (UTC−03:00). The following check also reported `OK`.

```text
2026-11-25 23:31:31 -03:00 BASELINE OK A=192.168.1.17
2026-11-25 23:32:02 -03:00 CHECK FAIL DNS_ERROR_RCODE_NAME_ERROR,Microsoft.DnsClient.Commands.ResolveDnsName
2026-11-25 23:32:02 -03:00 ALERT FAIL DNS_ERROR_RCODE_NAME_ERROR,Microsoft.DnsClient.Commands.ResolveDnsName
2026-11-25 23:32:12 -03:00 CHECK FAIL DNS_ERROR_RCODE_NAME_ERROR,Microsoft.DnsClient.Commands.ResolveDnsName
2026-11-25 23:32:22 -03:00 CHECK OK A=192.168.1.17
2026-11-25 23:32:22 -03:00 RECOVERY OK A=192.168.1.17
2026-11-25 23:32:32 -03:00 CHECK OK A=192.168.1.17
```

The 20 seconds between alert and recovery are the probe's observed timestamps. The test covered resolution from Windows on the local network and notifications during an interactive run. The published evidence is from the PowerShell console.

### PowerShell probe

```powershell
$log = Join-Path $env:USERPROFILE ("pi3-dns-monitor-{0}.log" -f (Get-Date -Format 'yyyyMMdd-HHmmss'))
$anterior = $null
Write-Host "Registro: $log"

while ($true) {
    $estado = 'FAIL'
    try {
        $respuesta = Resolve-DnsName -Name 'intranet.pi-lab.test.' -Type A -Server '192.168.1.17' -DnsOnly -NoHostsFile -QuickTimeout -ErrorAction Stop
        $direcciones = @($respuesta | Where-Object Type -eq 'A' | ForEach-Object IPAddress)
        $detalle = 'A=' + ($direcciones -join ',')
        if ($direcciones.Count -eq 1 -and $direcciones[0] -eq '192.168.1.17') {
            $estado = 'OK'
        }
    } catch {
        $detalle = $_.FullyQualifiedErrorId
    }

    $hora = Get-Date -Format 'yyyy-MM-dd HH:mm:ss zzz'
    Write-Host "$hora CHECK $estado $detalle"
    if ($estado -ne $anterior) {
        $evento = if ($null -eq $anterior) { 'BASELINE' }
                  elseif ($estado -eq 'FAIL') { 'ALERT' }
                  else { 'RECOVERY' }
        "$hora $evento $estado $detalle" | Tee-Object -FilePath $log -Append
        $anterior = $estado
    }
    Start-Sleep -Seconds 10
}
```
