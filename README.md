# Pi-hole with MikroTik Hotspots

Runbook for the Pi-hole DNS filtering deployment on a MikroTik network. It records the tested design, current rollout state, and recovery steps without storing router exports or credentials.

## Current status

- Pi-hole v6 is the filtering resolver for authenticated Hotspot clients on the central Hotspot router.
- The router DHCP scopes advertise their own gateway as DNS. The Hotspot router's built-in DNS interception still catches ordinary DNS traffic.
- Two static `pre-hotspot` NAT rules redirect authenticated Hotspot DNS (UDP and TCP port 53) to Pi-hole. They are health-controlled by a 30-second scheduler: three successful DNS checks enable them; two failures disable them so the Hotspot router can answer DNS using its configured upstream resolvers.
- Office-specific rules remain active. PPPoE redirects remain paused.
- Port 853 blocking is configured for the Office Hotspot only. DNS over HTTPS on port 443 is not blocked network-wide.
- The deployment uses one Pi-hole host. The fallback path exists, but a full Pi-hole outage/failover drill and capacity test at peak load have not been recorded.

## Documents

- [Deployment history and decisions](docs/DECISIONS.md)
- [Deployment and recovery runbook](docs/DEPLOYMENT.md)
- [Troubleshooting notes](docs/TROUBLESHOOTING.md)
- [Safe sharing checklist](docs/SAFE-SHARING.md)

All addresses and command values in the runbook are placeholders. Replace them only from a private, current network plan. Do not put raw exports, backups, credentials, serial numbers, public IPs, customer names, or phone numbers in this public repository.

## Scope and limits

DNS filtering blocks names resolved through Pi-hole. It does not inspect page content or block every form of encrypted DNS. Chrome Secure DNS and other DNS-over-HTTPS clients can bypass ordinary port 53 redirects. Filtering lists also need review and updates because domains may be absent or change over time.
