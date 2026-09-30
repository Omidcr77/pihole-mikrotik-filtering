# Troubleshooting notes

## Pi-hole admin page shows another service

Check the actual listening ports and web server configuration on the Pi-hole host. In this deployment, plain HTTP port 80 served a separate media web server. Do not assume the Pi-hole admin UI is at the bare host address; Pi-hole v6 commonly serves its admin interface under `/admin/` and may use HTTPS depending on local configuration.

## Pi-hole does not answer routed client queries

Check, in order:

1. Pi-hole FTL is running and listens on UDP and TCP port 53.
2. Pi-hole's listening mode allows the source address it actually sees.
3. The Main MikroTik has a connected route to Pi-hole and a narrowly scoped DNS source-NAT rule where required.
4. Return packets follow the same stateful NAT path.
5. A packet capture on the Pi-hole interface shows both query and response.

Checksum warnings in a host packet capture can be caused by NIC checksum offload. Confirm with a second capture point or by disabling offload temporarily before treating the warning as a damaged packet.

## A site is not blocked even though it was expected on a list

Test the exact hostname and its `www` form directly against Pi-hole:

```text
nslookup example-blocked-domain.test <PIHOLE_IP>
nslookup www.example-blocked-domain.test <PIHOLE_IP>
```

If Pi-hole returns public IP addresses, inspect the Query Log and search the active denylist, subscribed lists, allowlists, and group assignments. A domain mentioned in a source list is not proof it is present in the current compiled gravity database. A manual wildcard deny entry can cover the apex domain and subdomains; verify the result with fresh queries.

## Hotspot portal URL fails

Use the exact profile `dns-name` and hotspot address. Add the portal hostname to Pi-hole's local DNS when clients use Pi-hole. Captive portals generally require HTTP for the initial redirect; HTTPS cannot be transparently redirected without certificate errors. Do not rename unrelated Hotspot profiles when changing one service name.

## Chrome works only when Secure DNS is off

Check the DHCP-advertised DNS address and Chrome's Secure DNS mode. Port 53 NAT rules cannot force Chrome's DoH traffic through Pi-hole. Do not claim universal enforcement from a successful `nslookup` alone; compare the browser's behavior with Pi-hole's Query Log.

## WhatsApp media or status stops loading

Pause the most recent DNS redirect scope and compare behavior. Check Pi-hole Query Log for the exact failing hostnames and response codes before allowlisting anything. Do not use a generic allowlist based only on an app symptom: WhatsApp uses multiple CDN and service domains, and failures may be unrelated to DNS filtering.

## RouterOS command paste errors

RouterOS CLI context matters. Enter `/` to return to the root prompt before using an absolute command. For a multiline script, use WinBox/WebFig System → Scripts and paste into the Source field, apply, then inspect the saved source. Do not continue past a parse error; confirm the item and counters before scheduling it.
