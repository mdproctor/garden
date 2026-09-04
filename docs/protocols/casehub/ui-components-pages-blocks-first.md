---
id: PP-20260904-66fb70
title: "Non-domain UI components must be authored in pages or blocks-ui before app integration"
type: rule
scope: platform
applies_to: "All CaseHub repos building UI — platform visualization, dashboard widgets, reusable controls"
severity: important
refs:
  - casehubio/pages
  - casehubio/blocks-ui
violation_hint: "A UI component built directly in an app repo (claudony, aml, clinical) instead of pages/blocks-ui"
created: 2026-09-04
---

Non-domain UI components — visualization widgets, dashboard controls, data
presentation elements — must be authored in `pages` or `blocks-ui` first,
then integrated into application repos. Building directly in an app repo
prevents reuse across apps and fragments the component library.
Domain-specific views that only make sense in one app are exempt.
