# WildIp App

A DNS App for Technitium DNS Server that returns an IP address embedded in the queried name.

## Overview

- **APP-record driven** – no root-level `dnsApp.config`
- **Subdomain parsing** – extracts IPv4 or IPv6 values from the query name
- **IPv4 formats** – supports dotted decimal, dashed decimal, and compact 8-character hexadecimal notation
- **NODATA fallback** – returns an SOA-based NODATA response when parsing fails

## Integration / extension points

- Implements: `IDnsApplication`, `IDnsAppRecordRequestHandler`
- Runs as an APP-record request handler.

## Configuration

`dnsApp.config` is not used by this app.

Create an APP record for the base name you want to use, for example `ip.example.com`. The APP record data may restrict the addresses the app is allowed to synthesize:

```json
{
  "allowedNetworks": [
    "10.0.0.0/8",
    "172.16.0.0/12",
    "192.168.0.0/16",
    "::1/128"
  ]
}
```

## Runtime behavior

### A queries

The app recognizes IPv4 addresses in the following forms:

- Dotted decimal: `192.168.1.10.ip.example.com`
- Dashed decimal: `192-168-1-10.ip.example.com`
- Compact hexadecimal: `c0a8010a.ip.example.com`
- Prefixed hexadecimal: `app-c0a8010a.ip.example.com`

Each example above returns `192.168.1.10`. Hexadecimal parsing is case-insensitive.

### AAAA queries

- Accepts either dashed IPv6 text converted to `:` or a 32-character hex string.
- If parsing succeeds, returns an AAAA record.
- If parsing fails, returns NODATA with the zone SOA in the authority section.

## Risks / operational notes

- Parsing is permissive and can match unintended labels.
- Use `allowedNetworks` to constrain the addresses that may be returned.
- Be careful exposing this functionality publicly.

## Troubleshooting

- Confirm the APP record name is correct.
- Confirm the subdomain format is valid for the requested record type.
- Confirm the decoded address is included in `allowedNetworks`.
