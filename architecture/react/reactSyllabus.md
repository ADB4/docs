# React / TypeScript / MUI — 20-Week Mastery Syllabus (Expanded Edition)

**Self-Directed Saturday Study Program**
6–8 hours per session · 5 modules · 6 projects + capstone

*Inspired by the rigor of EECS 381 (University of Michigan)*

---

## Design Philosophy

This syllabus borrows two structural ideas from rigorous university courses:

**1. Prescribed readings with specific sources.** Each week assigns concrete chapters, documentation pages, or articles. You should read these before or during the session, not after. The readings are the minimum — go deeper where your curiosity takes you, but cover the assigned material.

**2. Projects sequenced to the material.** Six focused projects plus a capstone are threaded through the 20 weeks. Each project is designed so that everything you need has been covered by the time it is assigned. Projects build on each other: the data model from Project 2 reappears in Project 4, the theme from Project 3 carries into Project 5.

The 30-minute struggle rule still applies: when stuck on a concept, spend at least 30 minutes working through it before searching or asking for help. But do not spend hours stuck on a project requirement — ask your study companion for a targeted hint.

---

## Primary Reading Sources

All readings below reference these sources. Bookmark them before you begin.

| Abbreviation | Source | URL / Location |
|---|---|---|
| **TS-HB** | TypeScript Handbook (official) | typescriptlang.org/docs/handbook |
| **React** | React Documentation (react.dev) | react.dev/learn and react.dev/reference |
| **MUI** | MUI Documentation (v6) | mui.com/material-ui/getting-started/ |
| **TQ** | TanStack Query v5 Docs | tanstack.com/query/latest/docs |
| **Vite** | Vite Documentation | vite.dev/guide/ |
| **RTL** | React Testing Library Docs | testing-library.com/docs/react-testing-library/intro |
| **TL** | Testing Library (core) Docs | testing-library.com/docs/ |
| **UE** | user-event Docs | testing-library.com/docs/user-event/intro |
| **KCD** | Kent C. Dodds Blog | kentcdodds.com/blog |
| **RC** | React Compiler Docs | react.dev/learn/react-compiler |

*Supplementary sources are cited inline where assigned. All official docs are free.*

---

## Project Sequence Overview

Projects are cumulative. Later projects reuse code, types, and patterns from earlier ones.

| # | Project | Assigned / Due | Key Skills Exercised |
|---|---|---|---|
| P0 | TypeScript Kata Set | Wk 1 / End Wk 2 | Basic + advanced types, generics, utility types, type guards |
| P1 | Typed React Starter | Wk 3 / End Wk 4 | Prop typing, hook typing, tsconfig, as const, strict mode |
| P2 | Stateful Task Manager | Wk 5 / End Wk 7 | Rendering model, useReducer, Context, useRef, custom hooks |
| P3 | MUI Design System Shell | Wk 9 / End Wk 10 | Theme creation, sx, styled(), styleOverrides, dark mode |
| P4 | Data Dashboard | Wk 11 / End Wk 12 | DataGrid, Autocomplete, Dialog, module augmentation, a11y |
| P5 | Tested & Fetching | Wk 13 / End Wk 18 | Vitest, RTL, mocking, TanStack Query, Suspense, error boundaries, Vite |
| CAP | Capstone: Full Application | Wk 19 / End Wk 20 | All of the above, integrated |

---

## Module 1: TypeScript Foundations (Weeks 1–4)

### Week 1: TypeScript Fundamentals

**Topics:** Basic types, interfaces, type aliases, type inference, function typing, union and intersection types.

**Assigned Readings:**
- TS-HB: "The Basics" — read the entire page. Pay attention to type inference and the strictness flags.
- TS-HB: "Everyday Types" — full page. Focus on when to use interface vs. type alias.
- TS-HB: "Narrowing" — full page. Understand truthiness narrowing, typeof, and equality narrowing.
- TS-HB: "More on Functions" — read through "Function Overloads." Skim rest.
- TS-HB: "Object Types" — full page. Read the section on readonly and index signatures carefully.

> **PROJECT 0 ASSIGNED: TypeScript Kata Set**
> A set of 20–30 small type challenges: write type definitions, implement functions with specific signatures, fix broken types. No React yet. Work through them over Weeks 1–2. Everything needed is covered by end of Week 2.

*By the end of this week you should be able to write interfaces, use union types, and let TypeScript infer types without annotating everything.*

---

### Week 2: Advanced TypeScript

**Topics:** Generics, utility types (Partial, Pick, Omit, Record), satisfies operator, discriminated unions, type guards, conditional types (intro).

**Assigned Readings:**
- TS-HB: "Generics" — full chapter through "Generic Constraints." Read "Using Type Parameters in Generic Constraints" carefully.
- TS-HB: "Typeof Type Operator" and "Indexed Access Types" — both pages.
- TS-HB: "Conditional Types" — read for conceptual understanding. Don't memorize the syntax; understand when you'd reach for them.
- TS-HB: "Mapped Types" — full page. This is how Partial, Pick, etc. are implemented.
- TS-HB: "Template Literal Types" — skim for awareness. We return to this in Week 4.
- TypeScript docs: "Utility Types" reference page — read entries for Partial, Required, Pick, Omit, Record, ReturnType. Skim others.
- Article: "The satisfies operator" from the TypeScript 4.9 release notes (typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html).

*PROJECT 0 is due at end of this week. Complete all kata challenges.*

---

### Week 3: TypeScript in React

**Topics:** Prop typing without React.FC, hook typing (useState, useEffect, useContext), custom hook return types, compound component type patterns.

**Assigned Readings:**
- React: "TypeScript" page (react.dev/learn/typescript) — full page. This is the official guide to typing React code.
- React: "Describing the UI" section — read "Your First Component" and "Passing Props to a Component." Focus on how you'd type everything.
- React: "Adding Interactivity" section — read "State: A Component's Memory" and "State as a Snapshot."
- TS-HB: "Type Declarations" — understand .d.ts files and @types packages. You'll need this for MUI later.
- Article: Matt Pocock, "React.FC: Past, Present, and Future" or equivalent — understand why plain function declarations are preferred.

> **PROJECT 1 ASSIGNED: Typed React Starter**
> Build a small React app (a reading list tracker or similar) with Vite + TypeScript strict mode. Must use: typed props (no React.FC), typed useState and useEffect, at least one custom hook with an explicit return type, at least one generic component. No styling required — functionality and types only. Due end of Week 4.

---

### Week 4: TypeScript Project Configuration

**Topics:** tsconfig.json for production, declaration files in depth, as const assertions, template literal types, branded types.

**Assigned Readings:**
- TS-HB: "tsconfig.json" overview page. Then open tsconfig reference and read the entries for: strict, noUncheckedIndexedAccess, exactOptionalPropertyTypes, moduleResolution, target, lib.
- TS-HB: "Modules" chapter — understand ESM vs CommonJS resolution, import type syntax.
- Article: "as const" from TypeScript 3.4 release notes. Understand const assertions for literal types.
- Article: Matt Pocock or equivalent on branded types (also called nominal types, opaque types). Understand the pattern: `type UserId = string & { readonly __brand: unique symbol }`.
- Vite: "Getting Started" and "Features" pages — understand how Vite handles TypeScript (esbuild for dev, tsc for type checking).

*PROJECT 1 is due at end of this week. Your tsconfig should use strict: true and your types should compile without errors.*

---

## Module 2: React Deep Dive (Weeks 5–8)

### Week 5: React Rendering Model and React Compiler

**Topics:** Rendering pipeline (trigger/render/commit), Rules of React, React Compiler setup and automatic memoization, manual memoization as escape hatch, React DevTools Profiler, React.lazy and code splitting.

**Assigned Readings:**
- React: "Render and Commit" (react.dev/learn/render-and-commit) — this is the single most important page in the React docs. Read it twice.
- React: "Preserving and Resetting State" — full page. Understand key-based remounting.
- RC: React Compiler documentation — full page. Understand what it memoizes and the Rules of React it enforces.
- React: "Rules of React" page — read carefully. These are the contract the Compiler relies on.
- React: "memo" API reference, "useMemo" API reference, "useCallback" API reference — skim. Understand these as escape hatches, not defaults.
- React: "Lazy" API reference and "Suspense" API reference — read for code splitting. We return to Suspense for data fetching in Week 17.

> **PROJECT 2 ASSIGNED: Stateful Task Manager**
> A task management app with: multiple task lists, drag-to-reorder (optional), filters (active/completed/all), and a dark mode toggle. Must use: useReducer for task state, Context for theme/preferences (not for task data), at least one useRef, at least one custom hook. The React Compiler should be enabled. Everything needed is covered by end of Week 7. Due end of Week 7.

---

### Week 6: State Management

**Topics:** useReducer patterns, Context API (proper use and limitations), client vs. server vs. URL state, when external state libraries are warranted.

**Assigned Readings:**
- React: "Extracting State Logic into a Reducer" — full page. Study the reducer + dispatch pattern.
- React: "Scaling Up with Reducer and Context" — full page. Understand the composition, but note the performance caveat.
- React: "Passing Data Deeply with Context" — full page.
- React: "Choosing the State Structure" — full page. Extremely practical advice.
- React: "Sharing State Between Components" — read for lifting state concepts.
- Article: Kent C. Dodds, "Application State Management with React" (or equivalent). The argument for why most apps don't need external state libraries.

---

### Week 7: Hooks in Depth

**Topics:** useRef (DOM refs and mutable values), useImperativeHandle, useLayoutEffect, custom hook design, race conditions and stale closures.

**Assigned Readings:**
- React: "Referencing Values with Refs" and "Manipulating the DOM with Refs" — both full pages.
- React: "useRef" API reference — read the caveats section.
- React: "useImperativeHandle" API reference — understand when this is appropriate (almost never, but know it).
- React: "useLayoutEffect" API reference — understand the difference from useEffect. When layout measurement is needed.
- React: "Reusing Logic with Custom Hooks" — full page. Study the naming convention and return patterns.
- React: "You Might Not Need an Effect" — full page. One of the most important pages for avoiding bugs.
- React: "Synchronizing with Effects" and "Lifecycle of Reactive Effects" — both full pages. Study the dependency array rules.
- Article: Dan Abramov, "A Complete Guide to useEffect" (overreacted.io) — the definitive explanation of stale closures.

*PROJECT 2 is due at end of this week.*

---

### Week 8: Component Patterns and Architecture

**Topics:** Compound components, render props, HOCs (historical context only), controlled vs. uncontrolled components, folder structure conventions, React Server Components conceptual overview.

**Assigned Readings:**
- React: "Thinking in React" — re-read now with more experience. Focus on component decomposition.
- React: "Sharing State Between Components" — re-read. Controlled component pattern.
- Article or talk: Kent C. Dodds on compound component pattern (or equivalent). Understand the Context + children pattern.
- React: "React Server Components" page (if available on react.dev). Read for conceptual understanding only — we are not implementing RSC in this syllabus, but you should know what they are.
- Review: skim the React docs "Escape Hatches" section index. Make sure every hook is at least familiar.

*This is a consolidation week. No new project. Use extra time to refactor Project 2 using patterns learned this week. Self-review your code: would another developer understand it?*

---

## Module 3: MUI Mastery (Weeks 9–12)

### Week 9: MUI Fundamentals and Theming

**Topics:** Theme structure, palette/typography/spacing/breakpoints, CSS variables mode, Grid v2 and Stack, dark mode toggle.

**Assigned Readings:**
- MUI: "Material UI — Overview" and "Installation" — follow setup with Vite.
- MUI: "Theming" section — read "Theme" overview, "Palette," "Typography," "Spacing," "Breakpoints," "CSS theme variables."
- MUI: "Dark mode" page — understand CSS variables approach vs. dual theme approach.
- MUI: "Grid" (Grid v2) page — full page. Note: this is the responsive grid, not the old Grid v1.
- MUI: "Stack" page — understand when Stack vs Grid.
- MUI: "Box" page — understand Box as the fundamental layout primitive.

> **PROJECT 3 ASSIGNED: MUI Design System Shell**
> Create a reusable theme and layout shell: custom palette, typography scale, spacing tokens, responsive Grid layout, dark/light mode toggle persisted to localStorage. Must use: createTheme with full customization, CSS variables mode, at least one styled() component, sx prop for one-off styles. No business logic — focus on the design system. Due end of Week 10.

---

### Week 10: MUI Styling

**Topics:** sx prop in depth, styled() API, theme styleOverrides and variants, slots/slotProps API. Pigment CSS awareness.

**Assigned Readings:**
- MUI: "The sx prop" — full page. Understand responsive values, theme callbacks, and pseudo-selectors.
- MUI: "Styled engine" and "styled()" API reference — understand when styled() is better than sx.
- MUI: "Theme components" page (styleOverrides + variants) — this is how you customize components globally.
- MUI: "Overriding component structure" (slots/slotProps) — read the explanation. This is the modern customization API.
- MUI blog: search for the Pigment CSS announcement — skim for awareness. This is MUI's zero-runtime CSS direction.
- Do NOT read about or use makeStyles. It is deprecated and removed from MUI v6.

*PROJECT 3 is due at end of this week.*

---

### Week 11: MUI Complex Components

**Topics:** MUI X DataGrid (v8), Autocomplete, Dialog, Drawer, Snackbar, multi-step forms with validation.

**Assigned Readings:**
- MUI: "Data Grid" overview page (MUI X) — read "Getting Started," "Columns," "Rows," "Sorting," "Filtering." Skim "Selection" and "Pagination."
- MUI: "Autocomplete" page — full page. Study the controlled and async patterns.
- MUI: "Dialog" page — read the full-screen and responsive dialog patterns.
- MUI: "Drawer" page — understand persistent vs. temporary.
- MUI: "Snackbar" page — understand the auto-hide and action patterns.
- MUI: "Stepper" page — relevant for multi-step form UX.

> **PROJECT 4 ASSIGNED: Data Dashboard**
> Build a data dashboard that reuses your theme from Project 3. Must include: a DataGrid with sorting and filtering, an Autocomplete for search/filter, a Dialog for detail views, a Drawer for navigation, Snackbar for user feedback. Use the task data model from Project 2 or create a new dataset. Due end of Week 12.

---

### Week 12: MUI with TypeScript and Accessibility

**Topics:** Theme module augmentation, custom typed tokens, WAI-ARIA patterns, axe-core testing.

**Assigned Readings:**
- MUI: "TypeScript" page — read the module augmentation section. Understand how to add custom palette colors and spacing tokens that TypeScript recognizes.
- MUI: "How to customize" overview — re-read with fresh eyes now that you know the styling APIs.
- WAI-ARIA: Read "WAI-ARIA Authoring Practices" for Dialog, Menu, and Listbox patterns (w3.org/WAI/ARIA/apg/).
- Article: Marcy Sutton or Deque, introduction to axe-core for automated accessibility testing.
- MUI: review any component API page and notice the Accessibility section at the bottom — MUI documents ARIA requirements per component.

*PROJECT 4 is due at end of this week. Run axe-core on your dashboard before calling it done.*

---

## Module 4: Testing Mastery (Weeks 13–16)

*Testing is too important — and too often done badly — to compress into a single session. This module gives each dimension of frontend testing the dedicated attention it deserves.*

### Week 13: Testing Philosophy and Vitest Fundamentals

**Topics:** Why we test (and what not to test), the Testing Trophy, Vitest setup and configuration, test lifecycle hooks, basic assertions, the `describe`/`it`/`expect` API, watch mode, test environments (`jsdom`, `happy-dom`).

**Assigned Readings:**
- KCD: "Write tests. Not too many. Mostly integration." — one page. This sets the philosophy for everything that follows. Understand the Testing Trophy (static analysis → unit → integration → e2e) and why integration tests provide the best confidence-to-cost ratio.
- KCD: "Testing Implementation Details" — understand why testing internal state, method calls, and component internals leads to brittle tests.
- Vitest: "Getting Started" — follow the installation and first test walkthrough end-to-end. Set up Vitest in a Vite + React + TypeScript project.
- Vitest: "Features" — read the full page. Pay attention to: Vite-native transformation, watch mode, in-source testing, TypeScript support out of the box, and Jest compatibility.
- Vitest: "Configuring Vitest" — understand how Vitest reads `vite.config.ts` (the `test` key) or `vitest.config.ts`. Read the entries for `environment`, `globals`, `setupFiles`, `include`, `coverage`.
- Vitest: "Test API Reference" — read the entries for `describe`, `it` / `test`, `expect`, `beforeEach`, `afterEach`, `beforeAll`, `afterAll`. Skim the full matcher list on `expect`.
- Vitest: "Test Environment" — understand the difference between `node`, `jsdom`, and `happy-dom`. For React component testing, you need a DOM environment.

> **PROJECT 5 ASSIGNED: Tested & Fetching**
> Add a comprehensive test suite to your existing projects (primarily Project 2 or 4). Build incrementally across Weeks 13–16: start with basic component tests, add queries and user interaction, layer in mocking and async patterns, and finish with integration tests and accessibility checks. By the end of Week 16: at least 15 component tests, at least 3 custom hook tests, at least 2 mocked API tests, at least 1 integration test spanning multiple components. Then in Weeks 17–18, add TanStack Query for data fetching with corresponding test coverage. Due end of Week 18.

*By the end of this week you should have Vitest running in your project, be able to write and run basic assertion tests, and understand the testing philosophy that guides every decision in Weeks 14–16.*

---

### Week 14: React Testing Library — Rendering, Queries, and user-event

**Topics:** RTL philosophy (test behavior, not implementation), `render` and `screen`, query variants (`getBy`, `queryBy`, `findBy`, `getAllBy`), query priority (`getByRole` > `getByLabelText` > `getByText` > `getByTestId`), accessible queries and ARIA roles, `user-event` vs. `fireEvent`, `waitFor` and async utilities, `within`, `cleanup`, debugging with `screen.debug()` and `logRoles`.

**Assigned Readings:**
- RTL: "Introduction" — full page. Internalize the guiding principle: "The more your tests resemble the way your software is used, the more confidence they can give you."
- RTL: "Guiding Principles" — full page. Understand why RTL does not expose component instances or internal state.
- TL: "About Queries" — full page. This is the master reference for all query types. Study the priority table carefully:
  1. **Queries Accessible to Everyone:** `getByRole`, `getByLabelText`, `getByPlaceholderText`, `getByText`, `getByDisplayValue`
  2. **Semantic Queries:** `getByAltText`, `getByTitle`
  3. **Test IDs:** `getByTestId` — last resort only
- TL: "ByRole" query page — read in depth. `getByRole` with the `name` option (`getByRole('button', { name: /submit/i })`) should be your default query for nearly everything. Understand implicit ARIA roles for HTML elements (e.g., `<button>` has role `button`, `<a href>` has role `link`, `<input type="text">` has role `textbox`).
- UE: "Introduction" — full page. Understand why `user-event` simulates real user interactions (focus, keyboard, pointer) while `fireEvent` dispatches synthetic DOM events. `user-event` is always preferred.
- UE: "Setup" — understand the `userEvent.setup()` pattern that returns a user instance for the test.
- UE: API pages for `click`, `type`, `clear`, `selectOptions`, `keyboard`, `tab` — skim all. Read `click` and `type` carefully.
- KCD: "Common Mistakes with React Testing Library" — full article. Essential reading. Key takeaways: use `screen` instead of destructuring `render`; use `getByRole` not `getByTestId`; don't wrap things in `act()` unnecessarily (`render` and `fireEvent` already do this); use `findBy` for async elements instead of `waitFor` + `getBy`; prefer `user-event` over `fireEvent`.
- RTL: "API" reference — read the `render` function signature. Understand the `wrapper` option (for providing Context, theme, router wrappers). Read the `cleanup` section — Vitest with RTL auto-cleans after each test.

*By the end of this week you should be able to render any component, find elements using accessible queries (primarily `getByRole`), simulate user interactions with `user-event`, and assert on DOM state. You should also know when to use `queryBy` (asserting absence), `findBy` (async appearance), and `getAllBy` (multiple elements).*

---

### Week 15: Mocking, Async Testing, and Hook Testing

**Topics:** `vi.fn()`, `vi.spyOn()`, `vi.mock()`, mock lifecycle (`mockClear`/`mockReset`/`mockRestore`), mocking modules and API calls, fake timers (`vi.useFakeTimers`), testing async behavior (`findBy`, `waitFor`, `waitForElementToBeRemoved`), testing custom hooks with `renderHook`, mocking Context providers, mocking `fetch` or API layers.

**Assigned Readings:**
- Vitest: "Mocking" guide — full page. Understand the three core mocking tools:
  - `vi.fn()` — creates a standalone mock function for tracking calls and controlling return values
  - `vi.spyOn(object, 'method')` — wraps an existing method to track calls while preserving the original implementation by default
  - `vi.mock('./module')` — replaces an entire module; the call is hoisted to the top of the file before imports
- Vitest: "Mock Functions" API reference — read the entries for `mockReturnValue`, `mockReturnValueOnce`, `mockResolvedValue`, `mockResolvedValueOnce`, `mockImplementation`, `mockClear`, `mockReset`, `mockRestore`.
- Vitest: "Vi" API reference — read the entries for `vi.fn`, `vi.spyOn`, `vi.mock`, `vi.mocked`, `vi.hoisted`, `vi.useFakeTimers`, `vi.useRealTimers`, `vi.setSystemTime`, `vi.restoreAllMocks`.
- Vitest: "Mocking Modules" guide — read in full. Understand: `vi.mock` hoisting behavior (it always executes before imports); the factory function pattern; automocking (no factory); `importOriginal` for partial mocking; `vi.mocked()` as a type helper.
- RTL: "renderHook" API reference — understand how to test custom hooks in isolation. Study the `wrapper` option for providing Context.
- TL: "Async Utilities" — read `waitFor`, `waitForElementToBeRemoved`, and the `findBy` query variant. Understand that `findBy` is `waitFor` + `getBy` combined.
- Vitest: "Fake Timers" guide — understand `vi.useFakeTimers()`, `vi.advanceTimersByTime()`, `vi.runAllTimers()`, and `vi.useRealTimers()`. Know when to use fake timers (debounce, setTimeout, setInterval) vs. when to use `findBy` (data fetching, state transitions).
- KCD: "How to Test Custom React Hooks" (or equivalent) — understand when to use `renderHook` vs. when to test hooks indirectly through the components that use them. The general guidance: prefer testing through a component; use `renderHook` only when the hook is used by many components or has complex logic worth isolating.

*By the end of this week you should be able to mock API calls and module dependencies, test loading/error/success states, test custom hooks with `renderHook`, control time in tests, and understand the `mockClear`/`mockReset`/`mockRestore` lifecycle.*

---

### Week 16: Test Architecture, Patterns, and Coverage

**Topics:** Integration testing patterns (testing multiple components together), custom render functions with providers, testing forms (validation, submission, error display), testing accessibility with `jest-axe` / `vitest-axe`, test organization and file structure, code coverage configuration and interpretation, the Testing Trophy revisited, when NOT to test, refactoring tests for maintainability.

**Assigned Readings:**
- KCD: "AHA Testing" — understand the balance between DRY and WET in test code. Tests should be readable standalone. Prefer "Avoid Hasty Abstractions" — duplication in tests is often better than premature shared utilities.
- KCD: "Avoid Nesting When You're Testing" — understand the argument against deeply nested `describe` blocks. Flat test structure with descriptive names is often clearer.
- KCD: "Making Your UI Tests Resilient to Change" — understand the relationship between query choice and test brittleness. Tests that query by role or label survive refactors; tests that query by class name or test ID break.
- RTL: "Setup" — re-read the `wrapper` option. Study the pattern of creating a custom `renderWithProviders` function that wraps components in all necessary Context providers (theme, router, query client, etc.).
- Vitest: "Coverage" — read the configuration page. Understand the `coverage.provider` option (`v8` or `istanbul`), `coverage.include`, `coverage.exclude`, `coverage.thresholds`. Understand what coverage numbers mean and why 100% coverage is not a goal — coverage measures execution, not correctness.
- RTL: FAQ page — skim for common patterns: testing modals/portals, testing router-dependent components, testing components that use `useEffect` for data fetching.
- Article: Deque, "axe-core for Automated Accessibility Testing" or the `vitest-axe` README — understand how to add automated accessibility checks to your component tests with `expect(container).toHaveNoViolations()`.
- Review: re-read KCD "Common Mistakes with React Testing Library" — you will catch things you missed the first time now that you have hands-on experience.

> **End-of-module self-assessment:** Run your full test suite. Check coverage for critical user flows (not for a number). Review each test and ask: "If the implementation changed but the behavior stayed the same, would this test still pass?" If not, refactor it. Run `vitest-axe` on at least 3 components.

*By the end of this week you should have a maintainable, well-organized test suite that tests behavior over implementation, uses accessible queries, mocks external dependencies cleanly, and includes at least basic accessibility checks. You should be able to articulate why you test what you test — and why you leave some things untested.*

---

## Module 5: Integration and Production (Weeks 17–20)

### Week 17: Server State and Data Fetching

**Topics:** TanStack Query v5: useQuery, useSuspenseQuery, mutations, optimistic updates, Suspense and error boundaries, infinite scroll.

**Assigned Readings:**
- TQ: "Quick Start" — follow the tutorial end-to-end.
- TQ: "Queries" — understand query keys, staleTime, gcTime, enabled.
- TQ: "Mutations" — understand onMutate for optimistic updates, invalidateQueries.
- TQ: "Suspense" page — understand useSuspenseQuery and how it integrates with React Suspense.
- React: "Suspense" API reference — re-read now in the context of data fetching.
- React: "Error Boundaries" (class-based, or react-error-boundary library) — understand the pattern.
- TQ: "Infinite Queries" — read for the useInfiniteQuery pattern.

---

### Week 18: Build Tooling

**Topics:** Vite configuration in depth, React Compiler as Vite plugin, ESLint with typescript-eslint, Prettier, bundle analysis.

**Assigned Readings:**
- Vite: "Configuring Vite" — understand vite.config.ts, plugins, build options.
- Vite: "Building for Production" — understand chunking, minification, target.
- RC: React Compiler installation guide — follow the Vite plugin setup (babel-plugin-react-compiler via vite-plugin-react).
- typescript-eslint: "Getting Started" — set up the recommended config.
- Prettier: "Install" page — integrate with your editor and with ESLint (eslint-config-prettier).
- Article: search for "rollup-plugin-visualizer" or "vite-bundle-analyzer" — understand how to read a bundle treemap.

*PROJECT 5 is due at end of this week. Your project should pass all tests, fetch data via TanStack Query, and build cleanly with Vite.*

---

### Week 19: Capstone — Build

**Topics:** Integrate everything into a single, polished application.

**Assigned Readings:**
- No new assigned readings. This week is for building.
- Re-read any documentation for APIs you're using in the capstone. The docs are your primary reference now, not tutorials.

> **CAPSTONE PROJECT**
> Build a complete application that demonstrates mastery of all five modules. Requirements: TypeScript strict mode throughout, custom MUI theme with dark mode, at least one DataGrid or complex MUI component, TanStack Query for data fetching, full test suite (Vitest + RTL) with accessible queries and mocked API layer, React Compiler enabled, clean ESLint + Prettier configuration, proper code splitting. Scope it to be completable in 12–16 hours across two sessions. A portfolio-quality project tracker, admin dashboard, or content management tool are all good choices.

*Focus on building. Get the application working end-to-end. Do not polish prematurely — Week 20 is for refinement.*

---

### Week 20: Capstone — Polish, Test, and Review

**Topics:** Code review, test coverage gaps, accessibility audit, bundle analysis, documentation.

**Assigned Readings:**
- No new readings. This week is for refinement and self-review.

**Session plan:**
1. Run the Self-Review Protocol (see Appendix) on your capstone.
2. Run your full test suite. Identify the 2–3 most critical user flows that are untested or undertested. Write those tests.
3. Run `vitest-axe` on your key pages/components. Fix any accessibility violations.
4. Run the bundle analyzer. Look for unexpectedly large chunks. Apply code splitting if needed.
5. Write a README that explains the project, the architecture, and how to run it. This is portfolio-quality work — present it that way.
6. Ask your study companion to review one component, one test file, and your theme configuration.

*Self-review checklist: Does every component have typed props? Is your theme augmented with custom tokens? Do your tests cover the critical paths? Does the bundle look reasonable in the analyzer? Would you put this on your GitHub?*

---

## Appendix: Self-Review Protocol

At the end of each project, before moving on, run through this checklist. The EECS 381 course scheduled formal code reviews; since you are studying solo, this protocol is your substitute.

1. Read your own code as if reviewing someone else's PR. Would you approve it?
2. Check for any `any` types. Each one should be intentional and documented with a comment explaining why.
3. Run the TypeScript compiler with `--noEmit`. Fix all errors.
4. Open React DevTools Profiler. Record a typical interaction. Look for unnecessary re-renders.
5. Run your test suite. Check coverage for critical paths (not for 100% coverage).
6. Ask your study companion to review one component or hook you're unsure about.
7. Write a brief note (2–3 sentences) about what you learned and what was hardest. Keep a running log.

---

## Appendix: Key Technical Positions (2026)

These reflect the syllabus's perspective and should be treated as your baseline. They are not arbitrary opinions — they reflect where the ecosystem has settled as of 2026.

**React Compiler:** Stable (v1.0, October 2025). It is the primary performance optimization tool. Manual memoization (React.memo, useMemo, useCallback) is an escape hatch, not a default practice.

**Component typing:** Plain function declarations are preferred over React.FC.

**MUI styling:** sx for one-off styles, styled() for reusable styled components, theme styleOverrides for global defaults. makeStyles is dead.

**Test runner:** Vitest is the default for Vite-based projects. Jest is fine for legacy codebases.

**Testing philosophy:** Test behavior, not implementation. Use accessible queries (`getByRole` first). Prefer `user-event` over `fireEvent`. The Testing Trophy favors integration tests for the best confidence-to-cost ratio.

**Server state:** TanStack Query v5 with object-based API and stable Suspense support via useSuspenseQuery.

**Client state:** Most apps do not need an external state library. useState, useReducer, and Context cover the majority of needs. URL search params for shareable UI state.

**TypeScript strict mode:** Non-negotiable for new projects.

---

## Appendix: Module-by-Module Skill Progression

| Module | Weeks | Core Skills |
|---|---|---|
| 1 — TypeScript Foundations | 1–4 | Types, generics, utility types, tsconfig, strict mode |
| 2 — React Deep Dive | 5–8 | Rendering, state, hooks, component patterns |
| 3 — MUI Mastery | 9–12 | Theming, styling APIs, complex components, a11y |
| 4 — Testing Mastery | 13–16 | Vitest, RTL, mocking, async testing, coverage |
| 5 — Integration and Production | 17–20 | TanStack Query, Vite, build tooling, capstone |

---

## Appendix: Testing Skill Progression (Module 4 Detail)

| Week | You can write... | Key new tool/concept |
|---|---|---|
| 13 | Basic Vitest assertions, simple pure-function tests, first component `render` + `expect` | Vitest setup, `describe`/`it`/`expect`, test environments |
| 14 | Component tests with accessible queries and simulated user interaction | `screen`, `getByRole`, `user-event`, `findBy` for async |
| 15 | Tests with mocked APIs, mocked modules, fake timers, and isolated hook tests | `vi.fn`, `vi.mock`, `vi.spyOn`, `renderHook`, `waitFor` |
| 16 | Integration tests, provider-wrapped renders, a11y checks, coverage-guided refactoring | Custom render, `vitest-axe`, coverage thresholds, test architecture |