# Deployment decisions and tested history

This note records the troubleshooting path and resulting choices, with identifying network values removed.

## Pi-hole host and admin access

- Pi-hole v6 is installed on a Linux host. The `pihole` command was not on the shell's default PATH when invoked under `sudo`; the installed command was available at `/usr/local/bin/pihole`.
- The host's plain HTTP port 80 was serving a separate media web server. Pi-hole's admin interface was reached under its admin path/HTTPS configuration, not by assuming the root page belonged to Pi-hole.
- Pi-hole's upstream DNS was set to external resolvers. DNS rate limiting was set to zero during this work after checking the setting; any production rate-limit policy should be selected based on measurements.

## Validation before wider rollout

- A single authenticated Hotspot test client was pointed at Pi-hole. Allowed names resolved normally; a domain on the enabled adult blocklist returned Pi-hole's configured blocked response.
- RouterOS counters showed the pilot redirect/source-NAT rules matching test DNS, and a packet capture on the Pi-hole host showed both requests and replies.
- On the Office Hotspot, an Android Chrome client initially behaved differently while Chrome Secure DNS was enabled. The Office DHCP scope was changed to advertise its gateway DNS; after reconnecting and testing, normal browsing and blocked-domain lookups worked.
- The Office Hotspot portal name was changed to a private `home.arpa` name and a local DNS record was added. The plain HTTP portal status page worked; HTTPS was not configured with a matching certificate.

## All-Hotspot rollout

- All 20 configured DHCP network scopes on the Hotspot router now advertise their gateway as DNS.
- Two disabled-then-health-controlled NAT rules were added in `pre-hotspot` for authenticated Hotspot clients whose source addresses are in the router's existing `Network` address list. The router already had a source-NAT path toward the Pi-hole host.
- The health checker queries Pi-hole directly every 30 seconds. In observed state, its good counter reached three, the bad counter remained zero, and the two all-Hotspot redirect rules became enabled. The scheduler run counter advanced and its entry had no disabled (`X`) flag.
- PPPoE redirect rules remain paused. A prior PPPoE test coincided with WhatsApp status failures, but the affected Office router was later confirmed to use a static address, and the root cause was not established. Do not resume PPPoE redirects as part of the Hotspot rollout.
- A manually tested domain expected from a subscribed list was not found in Pi-hole's current list database. A wildcard deny entry was added; fresh queries for the apex and `www` hostname returned blocked addresses.

## Known gaps

- No second Pi-hole host is available; resolver redundancy is provided by RouterOS fallback, not a second filtering server.
- The broad rollout's failure transition has not been tested by taking Pi-hole offline while all Hotspot clients are active.
- Peak DNS query capacity with roughly 3,000 concurrent users has not been measured.
- Only Office Hotspot has explicit TCP/UDP port 853 filtering. DoH over HTTPS remains outside the scope of the current rules.
