# mocking reference — vitest + react testing library

a searchable, copy-paste-friendly reference for every mocking pattern you'll
encounter in React/TypeScript component tests. organized so you can jump
straight to the pattern you need.

---

## how to use this reference

**start here if you're new to mocking.** read part 1 first — it covers the
three core tools (`vi.fn`, `vi.spyOn`, `vi.mock`) and how vitest cleans up
mocks between tests. everything else builds on these.

**looking for a specific recipe?** scan the scenario tables below or use
ctrl+F / cmd+F across the files.

**in a hurry?** jump to the quick-reference table at the bottom of part 4.

---

## files

| File | What's inside | Read this when you need to… |
|------|---------------|-----------------------------|
| [01-foundations.md](./01-foundations.md) | `vi.fn()`, `vi.spyOn()`, `vi.mock()`, `vi.hoisted`, `importOriginal`, mock lifecycle (`mockClear`/`mockReset`/`mockRestore`), vitest config | Understand the building blocks. Every other pattern uses these. |
| [02-component-testing.md](./02-component-testing.md) | Mocking custom hooks, mocking child components, mocking context providers, mocking third-party hooks (React Router, matchMedia) | Write component tests that isolate what you're testing. |
| [03-async-and-network.md](./03-async-and-network.md) | `vi.stubGlobal` for fetch, `mockResolvedValue`/`mockRejectedValue`, sequential responses, deferred promises, MSW setup, fake timers | Test anything that talks to a server or uses `setTimeout`. |
| [04-typescript-and-troubleshooting.md](./04-typescript-and-troubleshooting.md) | `vi.mocked` type inference, `satisfies`, `MockInstance`, common mistakes with BAD/GOOD examples, quick-reference lookup table | Fix type errors in mocks, avoid known pitfalls, or quickly look up which tool to use. |

---

## scenario index

> "I need to ___. Where do I look?"

| I need to… | File | Section |
|---|---|---|
| Create a mock callback for a component prop | 01 | vi.fn() — standalone mock functions |
| Check that an existing method was called without replacing it | 01 | vi.spyOn() — observing without replacing |
| Replace an entire module in my test file | 01 | vi.mock() — replacing modules |
| Mock one export but keep the rest real | 01 | partial module mocking with importOriginal |
| Use a variable inside a vi.mock factory | 01 | vi.hoisted |
| Understand mockClear vs mockReset vs mockRestore | 01 | mock lifecycle |
| Configure automatic mock cleanup in vitest | 01 | recommended reset strategy |
| Mock a custom hook so my component gets fake data | 02 | mocking custom hooks in component tests |
| Reduce boilerplate when mocking a hook with many properties | 02 | mock factory helper |
| Mock a heavy child component (chart, map, editor) | 02 | mocking child components |
| Set up a reusable wrapper with providers (theme, query client) | 02 | mocking context providers |
| Mock React Router's useNavigate / useParams | 02 | react router |
| Decide between mocking a router hook vs using MemoryRouter | 02 | react router |
| Mock fetch for a component that calls an API | 03 | mocking fetch with vi.stubGlobal |
| Test loading → success → error transitions | 03 | async mock patterns |
| Keep a fetch promise pending to assert on loading state | 03 | deferred promise pattern |
| Set up MSW for more realistic network mocking | 03 | MSW as an alternative to fetch mocking |
| Test a debounced input or setTimeout behavior | 03 | mocking timers |
| Get TypeScript to understand my mock's return type | 04 | vi.mocked — deep type inference |
| Type a mock factory helper correctly | 04 | typing the mock factory with satisfies |
| Figure out why my mock is missing properties | 04 | mistake: mock return value shape doesn't match |
| Stop tests from leaking state to each other | 04 | mistake: not resetting mocks between tests |
| Understand why vi.mock inside describe doesn't work | 04 | mistake: putting vi.mock in the wrong scope |