# Deployment and recovery runbook

This runbook describes the working pattern used for the deployment. It is not a router export. Commands use placeholders and must be adapted to a router's verified interfaces, routes, and address lists.

## Topology

The Pi-hole host sits on a small routed server segment behind a central MikroTik. A separate MikroTik serves Hotspot clients across multiple routed or tunneled Hotspot networks.

```text
Hotspot clients
      |
Hotspot MikroTik (Hotspot DNS interception and health-controlled redirect)
      |
Main MikroTik (routes and source NAT toward Pi-hole)
      |
Pi-hole (DNS filtering; upstream resolvers configured here)
```

Before rollout, confirm that the Hotspot router can query Pi-hole, that replies return through the expected NAT path, and that Pi-hole accepts requests from the translated source address. Do not expose Pi-hole DNS to the public Internet.

## Pi-hole preparation

1. Give the Pi-hole host a stable address and confirm its default gateway and route to the MikroTik network.
2. Confirm DNS service on UDP and TCP port 53 from the router and one test client.
3. Configure at least one working upstream resolver in Pi-hole.
4. Enable and review the desired blocklists, then update gravity after changing list subscriptions.
5. Add local DNS records only where needed for internal services or Hotspot portal names.
6. Restrict the Pi-hole admin page and SSH to trusted management networks. Keep the admin password unique.

## Router prerequisites

On the Main MikroTik, verify the route to Pi-hole and the return path. If Pi-hole uses a restrictive listening mode, source-NAT routed client DNS so Pi-hole sees an allowed local source. Keep the NAT rule limited to DNS destined for the Pi-hole address; do not NAT general client traffic for this purpose.

On the Hotspot MikroTik:

- Set each Hotspot DHCP network's DNS server to that subnet's gateway address. This lets clients discover the router as their resolver.
- Keep the router's own DNS resolver enabled for LAN requests and set known-good upstream resolvers. This is the fallback used when the Pi-hole redirect is disabled.
- Identify the existing Hotspot client address list and verify it contains every intended Hotspot range. Do not assume it includes unrelated networks.
- Use the Hotspot `pre-hotspot` chain for authenticated-client DNS redirects so the rule runs before the dynamic Hotspot DNS redirect.

## DNS redirect pattern

The deployed rules use these properties:

```routeros
/ip firewall nat add chain=pre-hotspot action=dst-nat \
    to-addresses=<PIHOLE_IP> to-ports=53 protocol=udp \
    src-address-list=<AUTHENTICATED_HOTSPOT_RANGES> hotspot=auth \
    dst-port=53 disabled=yes comment="PIHOLE-ALL-HOTSPOTS UDP"

/ip firewall nat add chain=pre-hotspot action=dst-nat \
    to-addresses=<PIHOLE_IP> to-ports=53 protocol=tcp \
    src-address-list=<AUTHENTICATED_HOTSPOT_RANGES> hotspot=auth \
    dst-port=53 disabled=yes comment="PIHOLE-ALL-HOTSPOTS TCP"
```

Start disabled. Verify Pi-hole availability and the health script before enabling. Check rule packet counters while a test client performs a fresh DNS lookup. An authenticated Hotspot test client should resolve an allowed domain normally and a denied domain to the configured Pi-hole blocking response.

The address list and source-NAT rules are site-specific. Avoid copying internal ranges or interface names from another deployment.

## Health-controlled fallback

The RouterOS script checks a DNS name directly against Pi-hole. After three consecutive successful checks it enables rules whose comments start with `PIHOLE-ALL-HOTSPOTS `. After two failed checks it disables those rules. The scheduler runs the check every 30 seconds. With redirects disabled, RouterOS's dynamic Hotspot rule sends standard client DNS to the MikroTik resolver, which uses the configured upstream resolvers.

Verify the live state with:

```routeros
/system scheduler print detail where name="<HOTSPOT_PIHOLE_WATCHER>"
/system script environment print where name~"<HEALTH_COUNTER_PREFIX>"
/ip firewall nat print stats where comment~"^PIHOLE-ALL-HOTSPOTS "
```

In RouterOS output, `X` beside a specific scheduler or NAT row means that row is disabled. A flag legend by itself does not mean every row is disabled.

### Manual fallback

If Pi-hole becomes unavailable and the health watcher does not disable the redirect promptly, disable only the all-Hotspot redirect pair:

```routeros
/ip firewall nat disable [find where comment~"^PIHOLE-ALL-HOTSPOTS "]
```

Confirm ordinary browsing and DNS through the router. When Pi-hole is healthy again, the watcher can re-enable the redirects after the configured success threshold. To manually restore, first test Pi-hole from the router, then enable only these rules:

```routeros
/ip firewall nat enable [find where comment~"^PIHOLE-ALL-HOTSPOTS "]
```

## Secure DNS limits

Redirecting UDP/TCP port 53 does not intercept DNS over HTTPS (DoH), which uses HTTPS on port 443. DNS over TLS and DNS over QUIC commonly use port 853; the current deployment blocks that port only for the Office Hotspot. Extending that filter needs a separate review because encrypted DNS may be in use by clients. Chrome's automatic Secure DNS behavior can vary by DHCP-provided resolver and browser configuration.

## Rollout and monitoring

Use one representative authenticated client on a non-Office Hotspot first. Confirm its DHCP DNS, DNS redirect counters, Pi-hole query log, and ordinary web/app behavior. Then observe Pi-hole query rate, response latency, CPU, memory, and logs during normal and peak periods before treating a single Pi-hole host as suitable for all users. The network has been reported to have roughly 3,000 concurrent users; that is not a measured DNS capacity result.

Do not include PPPoE in this rollout without a separate test. The PPPoE DNS redirects are currently paused.
