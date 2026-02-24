# part 3 — async and network mocking

patterns for testing components that talk to servers, return promises, or
use timers. covers both direct fetch mocking and MSW for more realistic
network interception.

---

## mocking fetch with vi.stubGlobal

> **when would I need this?** when your component (or a hook it uses) calls
> `fetch` directly and you want to control what the server "returns" in
> your test.
>
> `vi.stubGlobal` is preferred over `global.fetch = vi.fn()` because it
> integrates with vitest's `unstubAllGlobals` config option for automatic
> cleanup.

### basic fetch mock

```typescript
vi.stubGlobal('fetch', vi.fn());

it('fetches and displays data', async () => {
  vi.mocked(fetch).mockResolvedValueOnce({
    ok: true,
    json: () => Promise.resolve([{ id: '1', title: 'Test' }]),
  } as Response);

  render(<TodoList />);

  expect(await screen.findByText('Test')).toBeInTheDocument();
  expect(fetch).toHaveBeenCalledWith('/api/todos');
});
```

### sequential responses (GET then POST)

> **when would I need this?** when a component makes multiple fetch calls
> in sequence — e.g. loads data on mount, then sends a POST when the user
> submits a form.

```typescript
it('handles initial load then a POST', async () => {
  const user = userEvent.setup();

  vi.mocked(fetch)
    .mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve([]), // GET — empty list
    } as Response)
    .mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve({ id: '1', title: 'New' }),
    } as Response); // POST response

  render(<TodoApp />);
  await screen.findByRole('textbox', { name: /task/i });

  await user.type(screen.getByRole('textbox', { name: /task/i }), 'New');
  await user.click(screen.getByRole('button', { name: /add/i }));

  expect(fetch).toHaveBeenNthCalledWith(1, '/api/todos');
  expect(fetch).toHaveBeenNthCalledWith(2, '/api/todos', expect.objectContaining({
    method: 'POST',
  }));
});
```

### mocking other globals (location, scrollTo)

the same `vi.stubGlobal` pattern works for any global:

```typescript
vi.stubGlobal('location', {
  ...window.location,
  href: 'https://example.com/todos',
  assign: vi.fn(),
  reload: vi.fn(),
});
```

### automatic cleanup

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    unstubAllGlobals: true, // restores all vi.stubGlobal stubs after each test
  },
});
```

---

## async mock patterns

> **when would I need this?** when you need to simulate promises that
> resolve, reject, or stay pending to test loading/success/error states.

### mockResolvedValue / mockRejectedValue

```typescript
const fetchUser = vi.fn();

// success path — every call resolves with this value
fetchUser.mockResolvedValue({ id: '1', name: 'Alice' });

// failure path — every call rejects with this error
fetchUser.mockRejectedValue(new Error('Network error'));

// sequenced: first call succeeds, second fails
fetchUser
  .mockResolvedValueOnce({ id: '1', name: 'Alice' })
  .mockRejectedValueOnce(new Error('Server error'));
```

### testing loading → success transitions

```typescript
it('shows loading spinner, then displays data', async () => {
  vi.mocked(fetch)
    .mockResolvedValueOnce({
      ok: true,
      json: () => Promise.resolve([{ id: '1', title: 'Todo' }]),
    } as Response);

  render(<TodoList />);

  // loading state — fetch hasn't resolved yet
  expect(screen.getByRole('progressbar')).toBeInTheDocument();

  // data loaded — findBy waits for the element to appear
  expect(await screen.findByText('Todo')).toBeInTheDocument();
  expect(screen.queryByRole('progressbar')).not.toBeInTheDocument();
});
```

### deferred promise pattern — holding a fetch open

> **when would I need this?** when you need to assert on the loading state
> *before* the promise resolves. `mockResolvedValueOnce` resolves on the
> next microtask, which can be too fast to catch the loading UI.

```typescript
it('shows loading state while fetch is pending', async () => {
  let resolveFetch!: (value: Response) => void;
  const pendingFetch = new Promise<Response>((resolve) => {
    resolveFetch = resolve;
  });

  vi.mocked(fetch).mockReturnValueOnce(pendingFetch);

  render(<TodoList />);

  // fetch hasn't resolved — loading should be visible
  expect(screen.getByRole('progressbar')).toBeInTheDocument();

  // now resolve it manually
  await act(async () => {
    resolveFetch({
      ok: true,
      json: () => Promise.resolve([{ id: '1', title: 'Done' }]),
    } as Response);
  });

  expect(await screen.findByText('Done')).toBeInTheDocument();
});
```

---

## MSW as an alternative to fetch mocking

> **when would I need this?** when you want network mocks that work
> regardless of which HTTP client your code uses (fetch, axios, ky, etc.),
> or when you're testing components that go through TanStack Query and you
> want caching, retries, and deduplication to work naturally.

Mock Service Worker intercepts requests at the network level. your
components, hooks, and API clients all exercise their real code paths — only
the server is fake.

### when to use MSW vs vi.stubGlobal('fetch', ...)

| Scenario | Recommendation |
|----------|---------------|
| Component directly calls `fetch` | Either approach works |
| Abstraction layer wraps `fetch` (Axios, ky, API client class) | MSW — avoids mocking the wrapper |
| Testing TanStack Query integration | MSW — lets query caching, retries, and deduplication work naturally |
| Simple unit test of one component | `vi.stubGlobal` is fine and lighter weight |

### setup

```typescript
// test-helpers/msw/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/todos', () => {
    return HttpResponse.json([
      { id: '1', title: 'Buy milk', completed: false },
    ]);
  }),

  http.post('/api/todos', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: '2', ...body }, { status: 201 });
  }),
];
```

```typescript
// test-helpers/msw/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```typescript
// vitest.setup.ts (referenced in vitest.config.ts setupFiles)
import { server } from './test-helpers/msw/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### overriding handlers per test

> **when would I need this?** when a specific test needs the API to behave
> differently than the default handlers — e.g. returning a 500 error.

```typescript
import { server } from '../test-helpers/msw/server';
import { http, HttpResponse } from 'msw';

it('shows error message when API fails', async () => {
  server.use(
    http.get('/api/todos', () => {
      return HttpResponse.json(
        { message: 'Internal Server Error' },
        { status: 500 }
      );
    })
  );

  render(<TodoList />);

  expect(await screen.findByText(/failed to load/i)).toBeInTheDocument();
});
```

---

## mocking timers

> **when would I need this?** when your component uses `setTimeout`,
> `setInterval`, `debounce`, or `Date.now` / `new Date()`. do **not** use
> fake timers for testing async data fetching — use `findBy` queries
> instead.

### basic fake timers

```typescript
import { vi, describe, it, expect, beforeEach, afterEach } from 'vitest';

describe('Notification auto-dismiss', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('disappears after 5 seconds', async () => {
    render(<Notification message="Saved!" />);

    expect(screen.getByText('Saved!')).toBeInTheDocument();

    await vi.advanceTimersByTimeAsync(5000);

    expect(screen.queryByText('Saved!')).not.toBeInTheDocument();
  });
});
```

### controlling Date.now / new Date()

```typescript
beforeEach(() => {
  vi.useFakeTimers();
  vi.setSystemTime(new Date('2025-06-15T10:00:00Z'));
});

afterEach(() => {
  vi.useRealTimers();
});

it('displays the current date', () => {
  render(<DateDisplay />);
  expect(screen.getByText('June 15, 2025')).toBeInTheDocument();
});
```

### testing debounced input

```typescript
it('debounces the search input', async () => {
  vi.useFakeTimers();
  const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });
  // ↑ CRITICAL: pass advanceTimers so user-event can advance fake timers
  // during its internal delays (typing, clicking, etc.)

  const onSearch = vi.fn();
  render(<SearchInput onSearch={onSearch} debounceMs={300} />);

  await user.type(screen.getByRole('searchbox'), 'react');

  // debounce hasn't fired yet
  expect(onSearch).not.toHaveBeenCalled();

  // advance past the debounce delay
  await vi.advanceTimersByTimeAsync(300);

  expect(onSearch).toHaveBeenCalledWith('react');

  vi.useRealTimers();
});
```

### async timer helpers

vitest provides async versions of timer helpers that flush microtasks between
ticks. prefer these when your timers trigger React state updates:

| Sync | Async (preferred for React) |
|------|-----|
| `vi.advanceTimersByTime(ms)` | `vi.advanceTimersByTimeAsync(ms)` |
| `vi.runAllTimers()` | `vi.runAllTimersAsync()` |
| `vi.runOnlyPendingTimers()` | `vi.runOnlyPendingTimersAsync()` |

the async versions avoid the common pitfall of timer callbacks running
synchronously before React can process state updates.