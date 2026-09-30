# Safe sharing checklist

This repository is public. Keep these items out of commits, issues, and screenshots:

- Router exports, backup files, database files, logs, and packet captures.
- Router, PPPoE, Hotspot, RADIUS, Pi-hole, SSH, VPN, and GitHub credentials or tokens.
- Serial numbers, software IDs, public IP addresses, private subscriber names, phone numbers, MAC addresses, and customer IP assignments.
- Internal DNS names and exact production subnets unless they have been deliberately approved for publication.

Use `export hide-sensitive` for private diagnostics, but inspect the result before sharing: identifiers and topology can remain sensitive even when passwords are hidden. Replace live values with placeholders before publishing.

If a credential has appeared in a shared transcript or file, rotate it. Removing it from a later commit does not erase it from repository history.
