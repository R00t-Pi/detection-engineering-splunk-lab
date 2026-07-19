# hardening

The broad technique-coverage sweep, separate from the narrative campaigns — running
additional ATT&CK techniques beyond the ones used in the realistic attack chain, hardening
the environment based on whatever gaps that turns up, and rolling out alerting for what's
covered so far.

- `ad-hardening-notes.md` — AD/GPO/audit-policy changes made in response to gaps found
- `alerting-baseline.md` — which Sigma-derived detections are live, alerting Splunk searches, and who owns them

Same rule as `campaigns/`: this folder tracks the *hardening and alerting* side effects of
running a technique, not a second copy of the technique itself. The actual rule, SPL, and
evidence stay in `detections/` either way — a technique triggered here that already exists
in `detections/` gets updated in place, not duplicated.