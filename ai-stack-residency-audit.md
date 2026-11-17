AI stack residency and sovereignty audit

A storage-region commitment answers one question: where data rests. It does not answer who can reach that data, or what the tool sends home on its own. On November 16, 2026, a researcher published a teardown showing that a Windows 11 system component records which AI coding agents a user runs and uploads the record under default-on optional diagnostic data (xusheng.dev, verified by the author on a Windows 11 ARM64 build 26200 VM; per the author, a usage bitmap, with no prompts or code observed). The lesson for an audit is general: the flows you signed for are rarely the only flows.

This checklist produces one record per AI tool in use. Each item has a verifiable answer. Budget roughly 30 to 60 minutes per tool for the first pass (assumption: documentation is in English and the tool is already installed).

The three questions per tool

1. Where is the vendor incorporated?

Read the data processing agreement or terms of service and find the contracting legal entity and its governing law clause. A subsidiary name is not enough; note the parent if the DPA names one. Incorporation decides which governments can compel the vendor directly, independent of where your data is stored.

2. Where does inference run?

Look for a commitment about inference endpoint geography that is separate from the storage commitment. Many policies specify where data is stored and say nothing about where prompts are processed. The absence of a stated inference region is itself an answer: record it as none stated.

3. What leaves the machine?

List what the tool transmits: documents, queries, outputs, metadata, and usage telemetry. The documentation gives you the intended flows. The capture below gives you the observed ones. Record both; disagreements between the two are findings.

Observing outbound traffic

macOS / Linux. Watch live connections for a running tool:

lsof -i -P -n | grep -i <process-name>
Capture DNS queries while the tool starts (stop with Ctrl-C after startup settles):

sudo tcpdump -i any -n udp port 53
Record a full capture for later review:

sudo tcpdump -i any -w ai-tool-startup.pcap
Windows (PowerShell, elevated):

Get-NetTCPConnection -State Established |
  Where-Object OwningProcess -eq (Get-Process <process-name>).Id
Resolve an endpoint and check who operates it:

dig +short api.vendor.example
whois <ip-address> | grep -iE 'orgname|org-name|country'
Run the capture in three states: during startup, during normal use, and while the tool is idle for 30 minutes. Idle traffic is where periodic telemetry shows up.

Scope honestly. A capture shows destinations, timing, and volume. TLS hides payload contents, so the capture alone cannot tell you whether a telemetry ping carries a counter or a query. Where contents matter, pair the capture with the vendor's telemetry documentation and any opt-out setting, and record what you could not verify.

Dependencies count. The runtime, the installer framework, and the operating system may each phone home separately from the tool. If a destination in the capture does not match the tool's documented endpoints, attribute it before you file it.

Record template

One row per tool:

Tool	Vendor legal entity (jurisdiction)	Storage commitment	Inference commitment	Observed outbound hosts	Telemetry opt-out	Risk tier	Reviewed
example-assistant	Example Corp (US, Delaware)	EU (Frankfurt), DPA §4.2	none stated	api.example.com (US); telemetry.example.com (US)	settings, off by default: no	B	2026-11-17
The same record as YAML, for keeping the audit in version control:

- tool: example-assistant
  vendor_entity: "Example Corp (US, Delaware)"
  storage_commitment: "EU (Frankfurt), DPA §4.2"
  inference_commitment: none stated
  observed_hosts:
    - host: api.example.com
      region: US
      state: [startup, active]
    - host: telemetry.example.com
      region: US
      state: [idle]
  telemetry_opt_out:
    exists: true
    default_off: false
    location: "settings > diagnostics"
  unverified:
    - payload contents (TLS)
  risk_tier: B
  reviewed: 2026-11-17
Tiering and cadence

Group tools by what they can reach, since the same telemetry behavior carries different weight at different access levels:

Tier A: tools with access to customer records, email, calendars, or credentials.
Tier B: tools that process internal documents or code.
Tier C: tools that touch only public information and connect to nothing internal.
Audit Tier A first. Re-run the audit at vendor renewal, after a major version update of the tool, and when a vendor changes its legal entity or is acquired. A dated reviewed field per row keeps the table honest about staleness.
