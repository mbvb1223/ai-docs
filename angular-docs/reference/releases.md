# Angular Versioning and Releases
> Source: https://angular.dev/reference/releases

## Overview

Angular maintains stability while continuously evolving. The framework balances the need for reusable components and established practices with ongoing improvements to keep pace with web ecosystem changes.

### Core Commitments

The Angular team ensures:
- Minimal breaking changes with migration tools when possible
- Clear deprecation policies allowing adequate update time

**Note:** These practices apply to Angular 2.0 and later. For AngularJS (v1.x), see the upgrade guide.

---

## Angular Versioning

Version numbers follow semantic versioning in the format `major.minor.patch` (e.g., 7.2.11).

### Release Types by Version Component

| Level | Description |
|-------|-------------|
| **Major** | Significant new features; update scripts, code refactoring, and testing may be required |
| **Minor** | Smaller features; fully backward-compatible with optional API adoption |
| **Patch** | Bug fixes; low-risk updates requiring no developer intervention |

**Important:** Starting with Angular 7, `@angular/core` and CLI versions are aligned.

### Preview Releases

- **Next** (`-next` identifier): Under active development and testing
- **Release Candidate** (`-rc` identifier): Feature-complete, in final testing

Latest pre-release documentation: [next.angular.dev](https://next.angular.dev)

---

## Release Frequency

Angular follows a predictable release schedule:

- Major release every 6 months
- 1-3 minor releases per major version
- Patch and pre-release builds nearly weekly

This cadence allows early access to features while maintaining stability for production users.

---

## Support Policy and Schedule

### Release Schedule

| Version | Date |
|---------|------|
| v21.1 | Week of 2026-01-12 |
| v21.2 | Week of 2026-02-23 |
| v22.0 | Week of 2026-05-18 |

### Support Window

All major releases receive 18 months of support:

| Stage | Duration | Details |
|-------|----------|---------|
| **Active** | 6 months | Regular updates and patches released |
| **LTS** | 12 months | Critical and security fixes only |

### Currently Supported Versions

| Version | Status | Released | Active Ends | LTS Ends |
|---------|--------|----------|-------------|----------|
| ^21.0.0 | Active | 2025-11-19 | 2026-05-19 | 2027-05-19 |
| ^20.0.0 | LTS | 2025-05-28 | 2025-11-19 | 2026-11-28 |
| ^19.0.0 | LTS | 2024-11-19 | 2025-05-28 | 2026-05-19 |

Angular v2-v18 are no longer supported.

### LTS Fixes

Fixes qualify for LTS versions when addressing:
- Newly identified security vulnerabilities
- Regressions caused by third-party changes (e.g., browser updates)

---

## Deprecation Policy

Deprecated APIs remain available for at least two major releases (approximately 12 months).

### Deprecation Stages

| Stage | Details |
|-------|---------|
| **Announcement** | Posted in changelog; marked with strikethrough in docs; annotated with `@deprecated` for IDE hints |
| **Deprecation Period** | Available for minimum two major releases; removal occurs only in major versions; maintained per LTS policy |
| **npm Dependencies** | Major releases may require updates; minor releases expand supported versions optionally |

---

## Compatibility Policy

Angular is composed of packages, subprojects, and tools. The public API surface is explicitly documented to prevent accidental private API use.

### Backward Compatibility Checks

Before merging changes:
- Unit and integration tests run
- Public API type definitions are compared pre/post-change
- Tests from Google applications depending on Angular are executed

Breaking changes in exceptional cases (critical security patches) include explicit framework communication.

---

## Breaking Change Policy and Update Paths

Breaking changes require modifications because the new state isn't backward-compatible. Examples include API removal, type definition changes, or dependency updates.

### Support for Breaking Changes

- Deprecation policy applied before removal
- `ng update` automation with code transformations tested on thousands of projects
- Step-by-step instructions via "Angular Update Guide"

### ng update Requirements

Update to any supported version if:
1. Target version is currently supported
2. Source version is within one major version of the target

**Example:** Update v10 -> v11 -> v12 for multi-version jumps (sequential updates only).

---

## Developer Preview

New APIs occasionally launch as "Developer Preview" -- fully functional but not yet stabilized under standard deprecation policies.

Rationale: Gather feedback before stabilization or complete documentation/tooling.

**Key Point:** These APIs may change anytime, even in patch versions. Teams must weigh benefits against stability risks.

---

## Experimental APIs

Experimental APIs may never stabilize or undergo significant changes.

**Key Point:** Standard framework policies don't apply. Change is possible at any version level.

---

## Additional Resources

- [Angular Blog](https://blog.angular.dev)
- [GitHub Repository](https://github.com/angular/angular)
- [Contribute](https://github.com/angular/angular/blob/main/CONTRIBUTING.md)
- [Public API Surface Documentation](https://github.com/angular/angular/blob/main/contributing-docs/public-api-surface.md)
- [Update Guide](https://angular.dev/update-guide)
