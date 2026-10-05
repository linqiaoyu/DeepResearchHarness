# R170 — Remove the finance-default documentation gate

The latest main CI failed after a README-only commit removed the finance-default
statement. The failing step was `finance_default_capabilities`; Postgres storage
and Qdrant vector index jobs passed. The failure was a documentation marker
requirement, not evidence of an Agent runtime failure.

Original CI error:

```text
finance_default_self_test=FAIL current registry is dirty
failed_step=finance_default_capabilities returncode=1
Process completed with exit code 1.
```

Evaluating the actual upstream README isolated the failure:

```text
README finance-default statement is missing or stale
```

The user explicitly requested deletion of this check, preservation of the current
README, and a push to main. Remove `scripts/check_finance_default_capabilities.py`
and its invocation from `scripts/gate.py`. The finance-default registry and the
other capability checks remain. This intentionally retires that check and its
README marker requirement; no evaluation metric, threshold, fixture, or test is
changed. No new reader-visible product capability is claimed.

Validation: the complete offline gate must pass before pushing. The existing
bidirectional guard-wiring check must find no dangling reference to the deleted
script. README must have an empty diff against the upstream failure commit.
Remote CI results are recorded in the local round report after the push.

Local environment failures were separate from the CI bug: Git object reads
reported `mmap failed: Operation timed out`, the old editable environment could
not import the package without PYTHONPATH, and rebuilding initially failed on a
missing Python CA certificate. The old environment was preserved; installation
succeeded using the system CA bundle. No paid provider calls were made. No
previously unreachable product downstream path was enabled by this change.
