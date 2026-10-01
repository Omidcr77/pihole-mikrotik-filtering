# Pi-hole with MikroTik Hotspots

Runbook for the Pi-hole DNS filtering deployment on a MikroTik network. It records the tested design, current rollout state, and recovery steps without storing router exports or credentials.

## Current status

- Pi-hole v6 is the filtering resolver for authenticated Hotspot clients on the central Hotspot router.
- The router DHCP scopes advertise their own gateway as DNS. The Hotspot router's built-in DNS interception still catches ordinary DNS traffic.
- Two static `pre-hotspot` NAT rules redirect authenticated Hotspot DNS (UDP and TCP port 53) to Pi-hole. They are health-controlled by a 30-second scheduler: three successful DNS checks enable them; two failures disable them so the Hotspot router can answer DNS using its configured upstream resolvers.
- Office-specific rules remain active. PPPoE redirects remain paused.
- Port 853 blocking is configured for the Office Hotspot only. DNS over HTTPS on port 443 is not blocked network-wide.
- The deployment uses one Pi-hole host. The fallback path exists, but a full Pi-hole outage/failover drill and a synthetic or peak-concurrency capacity test have not been recorded.

## Capacity snapshot (2026-10-01)

The server was observed during normal operation, not under a controlled 3,000-client load test:

- Hardware: Intel Core i5-6500 (4 cores), 7.7 GiB RAM, and a 931.5 GB rotational disk with about 811 GB free.
- Pi-hole: Core 6.4.3, FTL 6.7.1, Web 6.6; Kali GNU/Linux 2026.3.
- At the observation: about 43 DNS queries/second, load average below 1, 29.4% memory use in the dashboard, and FTL using about 386 MiB RSS.
- A 60-second `vmstat` sample showed CPU idle mostly 94–98%, negligible I/O wait, and almost no ongoing swap activity. About 1.4 GiB of swap was allocated/in use, while roughly 5 GiB RAM remained available.

These measurements indicate ample headroom for the observed traffic and make service to 3,000 connected hotspot clients a reasonable expectation. They do not guarantee performance during peak bursts or establish a maximum query rate. Recheck Pi-hole query rate, CPU, memory, swap activity, and I/O wait during the busiest period. The rotational disk has plenty of free space but is the main hardware limitation for database I/O. Pi-hole lists Debian and Ubuntu among supported distributions; Kali works in this deployment but is not listed as an officially supported OS. Consider a supported base OS for a future production rebuild.

## Documents

- [Deployment history and decisions](docs/DECISIONS.md)
- [Deployment and recovery runbook](docs/DEPLOYMENT.md)
- [Troubleshooting notes](docs/TROUBLESHOOTING.md)
- [Safe sharing checklist](docs/SAFE-SHARING.md)

All addresses and command values in the runbook are placeholders. Replace them only from a private, current network plan. Do not put raw exports, backups, credentials, serial numbers, public IPs, customer names, or phone numbers in this public repository.

## Scope and limits

DNS filtering blocks names resolved through Pi-hole. It does not inspect page content or block every form of encrypted DNS. Chrome Secure DNS and other DNS-over-HTTPS clients can bypass ordinary port 53 redirects. Filtering lists also need review and updates because domains may be absent or change over time.
