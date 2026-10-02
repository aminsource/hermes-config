---
name: frontend-test-coverage
description: "Frontend test coverage for Next.js/React apps: unit/integration/E2E layering, app-route smoke suites, Playwright fixtures, and verification."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [frontend, testing, nextjs, react, playwright, vitest, e2e]
    related_skills: [test-driven-development, requesting-code-review, dogfood]
---

# Frontend Test Coverage

Use this skill when the user asks to complete frontend tests, add coverage across an app, test all pages/routes, add Playwright/Vitest coverage, or harden a React/Next.js test suite.

## Core approach

Prefer layered coverage:

1. Unit tests for pure helpers, mappers, payload builders, and utility functions.
2. Integration tests for route handlers, hooks/services with mocked dependencies, and screen-level components.
3. E2E tests for critical browser journeys and app-route smoke coverage with deterministic network mocks.

Do not stop at writing tests. Run the targeted test first, then the relevant full suite, lint, and build when feasible.

## Next.js App Router route smoke coverage

When asked to test "all apps" or all pages in a Next.js App Router project:

1. Enumerate `src/app/**/page.tsx`.
2. Convert file paths to URLs:
   - Drop route groups such as `(header)` and `(actionMenu)`.
   - Keep regular segments.
   - Replace dynamic segments with deterministic examples, e.g. `[slug] -> 1`, `[groupId] -> e2e-group`.
3. Add a Playwright spec that iterates a route table and visits each route.
4. Split route tables by authentication behavior:
   - Authenticated routes use the auth fixture/cookies.
   - Truly public routes use the public fixture.
   - Verify with actual output; middleware may redirect pages that look public.
5. Assert stable visible copy for normal pages. For intentionally empty placeholder pages, assert `body` is visible and no crash occurred. For redirect/callback pages, assert stable destination copy after redirect.
6. Verify route coverage by comparing the normalized `src/app/**/page.tsx` list to `path:` entries in the spec. The target is zero missing routes.

See `references/next-playwright-route-smoke.md` for a condensed recipe and pitfalls.

## Playwright fixture/mocking guidance

- Mock layout-wide calls, not just the current page's obvious calls: profile, unread notifications, terminals, enum values, merchant status, guilds, and other global/navigation queries can crash otherwise unrelated pages.
- Match mocked response shapes to the app's unwrapped hook expectations. If a hook expects an array, return an array at the exact unwrapped location; avoid accidentally returning `{ content: [] }` where code calls `.map()` or `.filter()` on the value.
- Seed auth cookies through the browser context for authenticated route smoke tests.
- Keep mocked tokens obviously fake and deterministic.

## Verification checklist

- Targeted test passes, e.g. `yarn run test:e2e -- app-routes.spec.ts`.
- Full unit/integration suite passes, e.g. `yarn run test`.
- Full E2E suite passes, e.g. `yarn run test:e2e`.
- Lint passes or only known existing warnings remain.
- Build passes.
- `git status` is checked after build/test runs; generated assets such as service workers are reverted unless intentionally part of the change.

## Pitfalls

- Middleware redirects can make a route appear to pass while actually rendering login; assert route-specific or destination-specific text where possible.
- Empty placeholder pages need a weaker smoke assertion, but call that out in the test table rather than pretending they have stable content.
- Next.js builds may rewrite generated PWA/service-worker files. Revert unintended generated diffs after verification.
- Avoid broad network fallbacks that mask data-shape problems. If a page crashes with `data.map/filter is not a function`, add a specific mock with the correct shape instead of weakening assertions.
