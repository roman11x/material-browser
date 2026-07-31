# Experiments

Throwaway experiment code from source spikes lives here, one directory per issue:
`experiments/<issue-number>/`, each with its own README stating that it is throwaway.

Rules, from [`AGENTS.md`](../AGENTS.md) §7 and
[`docs/agents/implementer.md`](../docs/agents/implementer.md):

- spike issues produce documents and throwaway experiments, never permanent code;
- experiment code is clearly marked and excluded from release paths;
- nothing under `experiments/` ships or is imported by product code;
- experiments are labelled and time-boxed, and are either promoted through an ADR amendment or
  deleted — they never silently become architecture
  ([`docs/agents/escalation.md`](../docs/agents/escalation.md)).

The directory is empty today: no spike has been run.
