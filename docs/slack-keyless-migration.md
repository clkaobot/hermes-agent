# Wrapper → Hermes: keyless migration prompt

```text
Replace this Slack wrapper with Hermes, using the existing keyless Slack and LLM
integrations. Follow Hermes's documentation for installation and configuration.
Keep the old wrapper available but stopped as a rollback option.

Known gaps to handle:

- Keyless Slack already supports Socket Mode: call apps.connections.open through
  the existing credential-injecting Web API endpoint, then connect directly to the
  returned ticket-based WSS URL. No local Slack tokens or WebSocket relay is needed.
  Hermes needs only an auth/API-routing adaptation. Use its custom Web API base
  URL support (see PR #73433), with SDK-compatible non-secret token placeholders
  if required by the credential checks. Verify the endpoint accepts them. Preserve
  the native adapter; do not add polling, watermarks, thread discovery, or
  text-approval replacements.

- Stop and disable the wrapper BEFORE Hermes connects to the same Slack app.
  Slack distributes events across Socket Mode connections; it does not broadcast
  every event to both. Different channel allowlists do not prevent contention.
  Keep Shelley itself running, but not its Slack consumer. Reverse that order on
  rollback. An agent reached through the wrapper must arrange cutover/reporting
  outside the relay it is stopping.

- Pin the existing LLM endpoint as an explicit custom provider and preserve the
  requested model/API mode. A provider-prefixed model name alone can route Hermes
  to the wrong provider. Diagnose model errors separately from Slack delivery.

Preserve owner/channel restrictions and allow the old thread's channel if retaining
it. Slack thread history is not Shelley's private agent history. Verify a fresh
owner message reaches Hermes and produces one same-thread reply; a connected socket
or successful outbound post alone does not prove the migration works.
```

Custom endpoint support is proposed in the overlapping open
[PR #73433](https://github.com/NousResearch/hermes-agent/pull/73433) (`base_url`),
not assumed to be upstream already. Build on that work rather than submitting a
duplicate. The current local stopgap instead uses
`platforms.slack.extra.keyless_api_base_url` and supplies placeholders internally.
Protected Slack file downloads and app permissions still need separate verification;
keyless Web API access is not a blanket feature-parity guarantee.

Reference: [Slack — using multiple connections](https://docs.slack.dev/apis/events-api/using-socket-mode/#using-multiple-connections).
