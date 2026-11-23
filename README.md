https://github.com/Cytificial/AI-email-protection-alert
# Hercules gateway discovery

A small, standard-library-only Python 3.9+ collector to run inside the existing
Hermes backend environment on the phone. It does not require Android SDK tools.

## Purpose

The supplied audit lists a candidate listener and endpoints but contains no
request/response schemas, SSE frames, gateway/plugin source version, or successful
authenticated exchange. It also asserts one-time pairing and a tunnel requirement
without evidence. Those claims remain unverified.

This collector finds evidence to identify the installed implementation. It does
not claim to connect Hercules, authenticate a request, or prove a pairing flow.

## Run through Hermes

Extract this package to `/root/hercules/tools/gateway-discovery`, then run:

```sh
python3 /root/hercules/tools/gateway-discovery/hercules_gateway_discovery.py \
  --port 47930 \
  --output /sdcard/Download/Hercules_Gateway_Evidence.json
```

If Hermes source is elsewhere, append `--root /the/specific/hermes/source/folder`.
Default roots are `/opt/hermes-src`, `$HERMES_HOME/plugins` and
`$HERMES_HOME/installs` (or `~/.hermes` when HERMES_HOME is unset). Known mobile
plugin symlink roots are also inspected. Broad system/home roots are rejected.

## What the collector does

- Sends four unauthenticated GET probes to 127.0.0.1 only, including an intentionally
  nonexistent route to show when authentication may hide route existence.
- Disables proxy routing and redirects for these local probes.
- Reports status, recognized content type and authentication challenge scheme.
- If OpenAPI is available, exports only field/type/required schema structure.
- Parses Python source with AST; it never imports or executes that source.
- Records route/field/event candidates, file paths and SHA-256 hashes.
- Reports local Git revisions and sanitized recognized-host origin URLs when found.

No config.yaml, auth.json, .env, secrets.py, credentials.py, or config.py is read.
No bearer token is requested or sent. No raw HTTP response bodies, source code,
configuration values, exception messages, or process environments are exported.
It writes the JSON report only; no gateway, profile, package, or service changes
are made. The scan is bounded to 1,500 Python files / 24 MiB.

All source results are static candidates until tied to the running server. The
collector does not identify which process owns the candidate port. Repositories
may have local modifications; compare the source file hashes before assuming an
upstream commit exactly matches the installed files.

## Finish the protocol contract

Hermes should inspect the relevant handler source identified by the report and
record the actual method, JSON request, response/SSE shapes, completion/error
signals, authentication setup, session identity and cancellation/recovery behavior.
Use source paths and line references. Keep missing details marked unverified.

Only after the request format is established, test one harmless authenticated
request locally (for example an exact short reply), keeping credentials inside
the phone runtime. Export reviewed schemas and safe observed output, never tokens
or authentication files. Preserve the existing working gateway and profiles.

The final handoff should include Hercules_Gateway_Evidence.json plus an evidence-
backed Hercules_Gateway_Contract.md. An Android-app-side reachability check remains
necessary; no tunnel or network binding change is inferred by this package.

## Validation

Seven included unit tests passed on Python 3.12. They exercise source parsing,
credential-file exclusions, secret-value omissions, URL credential removal,
schema sanitization, 401 reporting and redirect refusal against local test servers.
No test was run against the actual phone gateway in the build environment.

```sh
python3 -m unittest -v test_gateway_discovery
```
