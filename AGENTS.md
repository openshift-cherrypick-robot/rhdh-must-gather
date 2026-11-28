# AGENTS.md

This file records only what you cannot work out from the code: intent behind
deliberate choices, and conventions that live outside the repository. Anything
visible in the source, the workflows, or the existing docs belongs there — extend
the file that owns it instead of adding it here.

`README.md`, `CONTRIBUTING.md`, `docs/`, and the `Makefile` are authoritative for
what the tool does, how it is run and tested, and what it collects.

## Intent that the code cannot express

- **No subprocesses for cluster access.** The binary talks to the Kubernetes API
  through client libraries and must never shell out to `kubectl`, `oc`, `helm`,
  `jq`, `yq`, or similar. This is not a style preference: the image runs on
  disconnected clusters and in minimal containers where those tools are absent.
- **Sanitization is a boundary, not a step.** It runs on every exit path, and
  every collector's output must pass through it. Do not add a code path that
  writes raw cluster data around it, and do not collect secrets unless
  `--with-secrets` is set.
- **A collector failure must not abort the run.** The value of a must-gather is
  the union of everything collected, so a collector that hits an error should
  record whatever partial data it already has, log the failure, and return so the
  remaining collectors still run. Only an explicit user interrupt may stop the
  collection early.
- **Collection runs sequentially on purpose.** Keeping the load on the API server
  predictable and the output ordering stable matters more than speed. Do not
  parallelize collectors without discussing it first.

## CI behaviour that is not written down

- A `pull_request` workflow must not use `on.pull_request.paths`. When the filter
  does not match, the workflow never runs, required status checks stay pending,
  and the pull request is blocked. Trigger on every pull request and gate steps
  with `tj-actions/changed-files` instead.

## Pull requests

- Every pull request follows `.github/PULL_REQUEST_TEMPLATE.md`: fill in the
  description, the linked Jira issue, if any, the acceptance-criteria checkboxes,
  and the reviewer test notes rather than deleting those sections.

## Changes to dependencies or the image

- Run `go mod tidy` whenever `go.mod` changes, and commit `go.mod` and `go.sum`
  together.
- Run `make regen-rpm-lockfile` whenever a change could alter the RPM set
  (`rpms.in.yaml`, a Containerfile, a base image bump), and commit the
  regenerated `rpms.lock.yaml`.

## Conventions

- **Issues are tracked in Jira in the `RHDHBUGS` project**, with component
  `Mustgather` — not in GitHub issues. The team reviews them there.
- **Commits follow Conventional Commits.** The body is required and explains why
  the change was made. Include an `Assisted-by:` trailer naming the AI tool used
  for agent-assisted work.
