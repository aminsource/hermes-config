# Next.js App Router + Playwright Route Smoke Coverage

This reference captures a proven pattern for adding all-page smoke coverage to a Next.js App Router frontend.

## Recipe

1. List every `src/app/**/page.tsx` route.
2. Normalize to URLs:
   - Remove route groups like `(header)`, `(home)`, `(actionMenu)`.
   - Keep normal segments.
   - Substitute dynamic segments with deterministic examples: `[slug] -> 1`, `[groupId] -> e2e-group`.
3. Build a Playwright route table:
   - `authenticatedRoutes` for routes that need seeded auth cookies.
   - `publicRoutes` for login/OTP pages or other genuinely unauthenticated pages.
4. For each route:
   - `goto(route.path)`.
   - Assert `body` is visible.
   - Assert stable text when the page has stable copy.
5. Add or extend `mockApi(page)` to cover every route's layout and page-level network calls.
6. Verify coverage with a script/check comparing normalized `page.tsx` routes to spec `path:` entries; expect no missing routes.
7. Run targeted E2E, then full E2E, unit tests, lint, and build.

## Mocking lessons

- Layout calls matter: profile, terminals, unread count, notifications, and navigation data can break pages unrelated to the test target.
- Data shape must match the app's hook layer after unwrapping:
  - If component code does `data.map(...)`, the mocked unwrapped value must be an array.
  - If component code expects paginated `data.content`, return a page object at that exact layer.
- Add specific mocks for known endpoints before any generic fallback. A generic `{ data: pageContent([]) }` can cause `data.map/filter is not a function` if the hook expected a plain array.
- Redirect/callback routes should assert final destination copy instead of transient callback copy.
- Some routes that look public may be redirected by middleware; use authenticated fixtures when the product path is authenticated.

## Example route-table shape

```ts
const authenticatedRoutes = [
  { path: "/", text: /Dashboard|Home/ },
  { path: "/shop/1", text: /Shop|Terminal/ },
  { path: "/associated/groups/e2e-group", text: /Group|Access/ },
  { path: "/contact" }, // placeholder page: visible body only
];

const publicRoutes = [
  { path: "/login", text: /Login|Phone/ },
  { path: "/login/verify-otp", text: /OTP|Code/ },
];
```

## Post-verification hygiene

Next/PWA builds may regenerate service workers or other generated assets. Always check `git status` after build/test runs and revert generated diffs unless they are intentionally part of the change.
