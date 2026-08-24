# Sprint 2 Regression Checklist

## Purpose

Minimum regression checklist before promoting a Sprint 2 release candidate.

## Sprint 1 Paths (must remain green)

- [ ] Google login / logout
- [ ] Manual New Application create
- [ ] AI URL prefill create (when provider configured)
- [ ] Applications list search / filter / sort / pagination
- [ ] Application detail / edit / status update / delete
- [ ] Frontend production build and backend `verify`

## Sprint 2 Paths

### Extension

- [ ] Load unpacked production or local build in Chrome
- [ ] SEEK full-page and side-panel capture
- [ ] Indeed full-page and side-panel capture
- [ ] Pairing approve from Web connect page
- [ ] Explicit Save → saved / duplicate / auth-required / failure states
- [ ] Citizenship/PR review recommended when present
- [ ] Companion appears on supported surfaces and stays quiet for normal jobs

### Web

- [ ] Home hides/shows Extension discovery appropriately
- [ ] Analytics ranges and empty/error/populated states
- [ ] Application Detail history and intelligence pending/failed/unavailable/success
- [ ] New Application remains usable as secondary path

### Async / data

- [ ] Core save succeeds when AI is slow/unavailable
- [ ] Intelligence eventually appears or shows controlled failure/unavailable
- [ ] Analytics updates without blocking create/status change
- [ ] Flyway migrations apply cleanly on PostgreSQL-shaped environment

## CI Gates

- [ ] Backend CI `verify` green on candidate SHA
- [ ] Frontend CI lint + build green on candidate SHA
- [ ] Local `scripts/precheck` recommended before push when changing Extension/frontend/backend together

## Related

- [Release Quality Gate](release-quality-gate.md)
- [Definition of Done](../definition-of-done.md)
