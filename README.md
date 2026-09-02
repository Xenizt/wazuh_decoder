# Wazuh Decoder Builder

**[Live demo](https://xenizt.github.io/wazuh_decoder/)**

A single-file, offline tool for writing Wazuh decoders by pointing at a log line instead of hand-writing regex. Drag a field name onto part of the log and the tool generates the `<regex>`, the `<order>`, the parent/child XML, and a matching rule skeleton.

No build step, no dependencies, no server. Open `wazuh-decoder-builder.html` (or `index.html`, same file) in a browser.

## Why

Hand-writing Wazuh decoder regexes is error-prone: it's easy to get a decoder that matches the sample log in front of you but breaks on the next real variant, or to have an earlier child decoder silently swallow a later one. This tool turns that into pointing at the parts of a log line you want captured, and checks the whole decoder chain against every sample as you go.

## What it does

- **Parent and child decoders.** The parent captures the fields every log of a source shares; each child handles one variant. Switch between them with the tabs at the top.
- **Drag and drop capture.** Drop a field chip onto a token, or select any substring with the mouse and click a chip. Each capture gets a colour that matches its position in `<order>`.
- **Skip marker.** The `⋯ skip` chip emits `.*?` instead of baking a varying value into the regex as a literal. This is the most common reason a hand-written decoder matches the sample log but fails on real traffic.
- **Pattern control.** Every capture's regex fragment can be changed (IPv4, number, word, inside quotes, inside parens, timestamp, custom).
- **Chain check.** Continuously verifies that the parent's prematch and regex match every child's line, that each child decodes its own line, and — the one that is hardest to spot by hand — that no earlier child also matches a later child's line. Wazuh stops at the first matching child, so an over-broad child silently swallows the ones below it.
- **Log library.** 37 sample lines across sshd, sudo, auditd, iptables, nginx, Apache, HAProxy, Squid, PostgreSQL, MySQL, Postfix, BIND, dhcpd, vsftpd, ESXi, Cisco ASA, FortiGate, Palo Alto, MikroTik, pfSense, Windows Security, fail2ban, ClamAV and ModSecurity. Searchable, and you can paste your own lines in.
- **Output tabs.** Decoder set, rule skeleton, an alert JSON preview, and the `wazuh-logtest` commands for each variant.

## Notes and limits

- Generated regexes use `type="pcre2"`. They will not work with Wazuh's legacy OS_Regex engine.
- Anything between two captures becomes a literal in the regex. If that text varies between logs, either capture it or mark it with `⋯ skip`.
- The `YYYY Mon DD HH:MM:SS agent->location` prefix that Wazuh writes into `archives.log` is stripped automatically. That header is added at write time and never reaches the decoder.
- Custom log lines you paste into the library are not persisted; they are gone on reload. To keep a set permanently, edit the `LIBRARY` array near the top of the `<script>` block: `{g:"group", n:"name", d:"decoder-name", l:\`the log line\`}`.
- Always confirm with `/var/ossec/bin/wazuh-logtest` before restarting the manager. The in-browser test uses JavaScript's regex engine, which is close to PCRE2 but not identical.

## Sample data

All bundled log lines are synthetic. Addresses come from the documentation ranges reserved by RFC 5737 (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`), RFC 1918 private space, and `example.com`. No hostnames, usernames, keys or device identifiers belong to a real system.

## Typical workflow

1. Load a line from the library, or paste your own.
2. On the **parent** tab, capture only what every variant of that source shares — usually a timestamp, an address, and a subsystem or program name.
3. Add a child per variant, load that variant's line, and capture what is specific to it.
4. Read the chain check. Green means each child decodes its own line and nothing upstream blocks it.
5. Copy the decoder set into `/var/ossec/etc/decoders/local_decoder.xml` and the rules into `/var/ossec/etc/rules/local_rules.xml`.
6. Test each variant with `wazuh-logtest`, then `systemctl restart wazuh-manager`.
