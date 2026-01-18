# React Reference: Hooks and Components
# Table of Contents

- [1: Hooks](#part-1-hooks)
  - [Examples](#examples)
    - [useLocalStorage to sync state with browser storage](#uselocalstorage-to-sync-state-with-browser-storage)
      - [Usage](#usage)
    - [useFetch to fetch data from an API](#usefetch-to-fetch-data-from-an-api)
      - [usage](#usage-1)
    - [useWindowSize to track window dimensions](#usewindowsize-to-track-window-dimensions)
- [2: Hooks vs. Components](#part-2-hooks-vs-components)
  - [Testing Considerations](#testing-considerations)
## Part 1: Hooks

Hooks are functions that allow you to use state and other React features in functional components.

> **Context:** 
>Before hooks were introduced in React 16.8, you could only use state and lifecycle methods in class components.

The idea is that you can "hook into" React features like state management, side effects, context, and more, in a way that is reuseable.

Some hooks you've probably encountered are useState and useEffect, but there are a myriad of usecases for custom hook, listed below. In the case of RAMP (as of 01/26/26), another common hook is useForm from *react-hook-forms*.

There are a few important rules for hooks:
1. Hooks must be called at the top level of your component. Not inside loops or conditions.
2. Hooks can only be called from React functional components or custom hooks.
3. (optional, but best practice) Name your custom hooks using the convention "use{YourCustomHook}"

You'll want to use a hook when you want to reuse logic across components, when managing state or side effects without rendering, and when separation of concerns is needed.

### Examples

#### useLocalStorage to sync state with browser storage:
```TypeScript
const [user, setUser] = useLocalStorage('user', null);

import { useState, useEffect, useCallback } from 'react';

function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  // get initial value from localStorage or use provided initial value
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') {
      return initialValue;
    }

    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  // update localStorage when state changes
  const setValue = useCallback(
    (value: T | ((prev: T) => T)) => {
      try {
        setStoredValue((prev) => {
          const valueToStore = value instanceof Function ? value(prev) : value;
          
          if (typeof window !== 'undefined') {
            window.localStorage.setItem(key, JSON.stringify(valueToStore));
          }
          
          return valueToStore;
        });
      } catch (error) {
        console.error(`Error setting localStorage key "${key}":`, error);
      }
    },
    [key]
  );

  // sync across tabs/windows
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === key && e.newValue) {
        try {
          setStoredValue(JSON.parse(e.newValue));
        } catch (error) {
          console.error(`Error parsing storage event for key "${key}":`, error);
        }
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [key]);

  return [storedValue, setValue];
}

export default useLocalStorage;
```
#### Usage:
```TypeScript
  const [formData, setFormData] = useLocalStorage<FormData>(
    'contact-form-draft',
    INITIAL_FORM_DATA
  );
```
### useFetch to fetch data from an API:
```TypeScript
import { useState, useEffect, useCallback } from 'react';

interface UseFetchOptions extends RequestInit {
  skip?: boolean;
}

interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useFetch<T>(
  url: string,
  options?: UseFetchOptions
): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<Error | null>(null);

  const { skip = false, ...fetchOptions } = options || {};

  const fetchData = useCallback(async () => {
    if (skip) {
      setLoading(false);
      return;
    }
    // AbortController is a built-in web API that allows you to cancel async operations
    const abortController = new AbortController();
    
    try {
      setLoading(true);
      setError(null);

      const response = await fetch(url, {
        ...fetchOptions,
        signal: abortController.signal,
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result = await response.json();
      setData(result);
    } catch (err) {
      if (err instanceof Error) {
        if (err.name !== 'AbortError') {
          setError(err);
        }
      } else {
        setError(new Error('An unknown error occurred'));
      }
    } finally {
      setLoading(false);
    }

    return () => abortController.abort();
  }, [url, skip, fetchOptions]);

  useEffect(() => {
    const cleanup = fetchData();
    return () => {
      cleanup?.then((abort) => abort?.());
    };
  }, [fetchData]);

  const refetch = useCallback(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch };
}

export default useFetch;
```
#### usage:
```TypeScript
  const { data, loading, error, refetch } = useFetch<User[]>(
    '/api/users',
    {
      headers: { 'Content-Type': 'application/json' },
    }
  );
```
### useWindowSize to track window dimensions
```TypeScript
import { useState, useEffect } from 'react';

interface WindowSize {
  width: number;
  height: number;
}

function useWindowSize(): WindowSize {
  const [windowSize, setWindowSize] = useState<WindowSize>(() => {
    // SSR-safe initialization
    if (typeof window === 'undefined') {
      return { width: 0, height: 0 };
    }
    return {
      width: window.innerWidth,
      height: window.innerHeight,
    };
  });

  useEffect(() => {
    // Ensure we're in the browser
    if (typeof window === 'undefined') {
      return;
    }

    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    // Add event listener
    window.addEventListener('resize', handleResize);

    // Call handler immediately to set initial size
    handleResize();

    // Cleanup
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}

export default useWindowSize;
```
## Part 2: Hooks vs. Components
Components have a lot in common with hooks, but the main difference between the two is that components contain visuals as well as logic, whereas hooks contain only logic. Components render UI and return TSX, while hooks encapsulate logic and return data/functions.

Use a component when:
- You're rendering UI
- You want to reuse both logic AND presentation together
- You need component lifecycle (render, mount, and unmount)

Additionally, you'll often use them together when calling a hook within a component. But, not the other way around---hooks do not render.
### Testing Considerations
One thing to note is that testing hooks is simpler because you can just call them. Components, on the other hand, require rendering.