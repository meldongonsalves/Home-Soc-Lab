# 04 — Splunk & Sysmon Deployment

This was the stage that made everything else in the lab observable — without
it, the attack in the next section would happen with nothing to show for it.

## Splunk Enterprise (on `SPLUNK-SRV`)

- Set a static IP via netplan on Ubuntu Server so the endpoint forwarders
  had a stable target to send logs to.
- Installed the Splunk Enterprise `.deb` package, ran it as a dedicated
  `splunk` user (not root), and enabled it to start on boot
  (`./splunk enable boot-start -user splunk`).
- Created a custom index called `endpoint` to keep this lab's telemetry
  separate from Splunk's default indexes.
- Under **Settings → Forwarding and receiving → Configure receiving**,
  opened a receiving port on `9997` (Splunk's default forwarder port) so
  the Windows hosts had somewhere to send data.

## Sysmon + Splunk Universal Forwarder (on `AD-DC01` and `TARGET-PC`)

On each Windows host:

1. Installed **Sysmon** (Microsoft Sysinternals) using a public,
   well-maintained Sysmon configuration rather than the (very noisy)
   default config — this filters Sysmon's event volume down to genuinely
   useful signal.
2. Installed the **Splunk Universal Forwarder**, pointing it at
   `192.168.10.10:9997` (the Splunk server) during setup.
3. The forwarder's default `inputs.conf` doesn't collect anything by
   itself — it has to be told what to monitor. Rather than editing the
   shipped `inputs.conf` directly (any Splunk upgrade would overwrite
   that), I created a new one under the forwarder's `local\` directory
   specifying the Windows **Security**, **System**, and **Sysmon**
   event log channels, all tagged to the `endpoint` index.
4. Restarted the Splunk Forwarder service after every `inputs.conf` change
   — it does not pick up changes live.
5. One easy-to-miss gotcha: by default the forwarder service can run under
   a restricted `NT SERVICE` account that doesn't have permission to read
   the Security event log. Had to change the service's **Log on as** to
   **Local System account** before events actually started flowing.

## Verifying ingestion

In Splunk's **Search & Reporting** app:

```
index=endpoint
```

...over the last 24 hours confirmed both hosts (`AD-DC01` and `TARGET-PC`)
were reporting in, with `source` values correctly split across Security,
System, and Sysmon channels. This confirmation step mattered — without
verifying ingestion here, any "no results" later would be ambiguous between
"no attack happened" and "logging is broken," which are very different
problems to debug.
