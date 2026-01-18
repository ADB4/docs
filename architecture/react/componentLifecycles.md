# React Reference: Component Lifecycle
Modern React development primarily uses function components with hooks rather than class components. This guide covers the component lifecycle in the context of functional components and TypeScript.
# Table of Contents

- [1: Component Lifecycle Phases](#component-lifecycle-phases)
  - [1. Mounting Phase](#1-mounting-phase)
  - [2. Updating Phase](#2-updating-phase)
  - [3. Unmounting Phase](#3-unmounting-phase)
- [2: Essential Hooks for Lifecycle Management](#essential-hooks-for-lifecycle-management)
  - [useEffect](#useeffect)
  - [useLayoutEffect](#uselayouteffect)
  - [useInsertionEffect](#useinsertioneffect)
- [3: Modern Best Practices](#modern-best-practices)
  - [1. Avoid Empty Dependency Arrays Unless Necessary](#1-avoid-empty-dependency-arrays-unless-necessary)
  - [2. Always Clean Up Side Effects](#2-always-clean-up-side-effects)
  - [3. Use Custom Hooks for Reusable Lifecycle Logic](#3-use-custom-hooks-for-reusable-lifecycle-logic)
  - [4. Type Your Hook Dependencies](#4-type-your-hook-dependencies)
  - [5. Avoid Stale Closures](#5-avoid-stale-closures)
  - [6. Separate Concerns](#6-separate-concerns)
- [4: React 18+ Features](#react-18-features)
  - [Automatic Batching](#automatic-batching)
  - [useTransition](#usetransition)
  - [useDeferredValue](#usedeferredvalue)
- [5: Common Patterns](#common-patterns)
  - [Data Fetching with Cleanup](#data-fetching-with-cleanup)
  - [Event Listeners](#event-listeners)
  - [Intervals and Timers](#intervals-and-timers)




## Component Lifecycle Phases

### 1. Mounting Phase

The mounting phase occurs when a component is created and inserted into the DOM.

```typescript
import { useEffect, useState } from 'react';

interface UserProps {
  userId: string;
}

const UserProfile: React.FC<UserProps> = ({ userId }) => {
  const [user, setUser] = useState<User | null>(null);

  // Runs once after initial render (mounting)
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, []); // Empty dependency array = mount only

  return <div>{user?.name}</div>;
};
```

### 2. Updating Phase

The updating phase occurs when props or state change, triggering a re-render.

```typescript
const UserProfile: React.FC<UserProps> = ({ userId }) => {
  const [user, setUser] = useState<User | null>(null);

  // Runs on mount AND whenever userId changes
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]); // Dependency array tracks userId

  return <div>{user?.name}</div>;
};
```

### 3. Unmounting Phase

The unmounting phase occurs when a component is removed from the DOM.

```typescript
const WebSocketComponent: React.FC = () => {
  const [data, setData] = useState<string>('');

  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com');
    
    ws.onmessage = (event) => {
      setData(event.data);
    };

    // Cleanup function runs on unmount
    return () => {
      ws.close();
    };
  }, []);

  return <div>{data}</div>;
};
```

## Essential Hooks for Lifecycle Management

### useEffect

The primary hook for side effects and lifecycle events.

```typescript
useEffect(() => {
  // Effect logic here
  
  return () => {
    // Cleanup logic here
  };
}, [dependencies]);
```

**When it runs:**
- After the component renders
- After the DOM has been updated
- Dependencies determine if it re-runs

**Common use cases:**
- Data fetching
- Subscriptions
- Event listeners
- Timers

### useLayoutEffect

Similar to useEffect but runs synchronously after DOM mutations, before the browser paints.

```typescript
import { useLayoutEffect, useRef } from 'react';

const MeasuredComponent: React.FC = () => {
  const divRef = useRef<HTMLDivElement>(null);

  useLayoutEffect(() => {
    if (divRef.current) {
      const { height } = divRef.current.getBoundingClientRect();
      console.log('Height:', height);
    }
  }, []);

  return <div ref={divRef}>Content</div>;
};
```

**When to use:**
- Measuring DOM elements
- Preventing visual flicker
- Synchronous DOM mutations

**Warning:** Can hurt performance if overused.

### useInsertionEffect

Runs before DOM mutations. Primarily for CSS-in-JS libraries.

```typescript
import { useInsertionEffect } from 'react';

const useCSS = (rule: string) => {
  useInsertionEffect(() => {
    const style = document.createElement('style');
    style.textContent = rule;
    document.head.appendChild(style);
    
    return () => {
      document.head.removeChild(style);
    };
  }, [rule]);
};
```

## Modern Best Practices

### 1. Avoid Empty Dependency Arrays Unless Necessary

```typescript
// Avoid: Effect depends on userId but doesn't list it
useEffect(() => {
  fetchUser(userId).then(setUser);
}, []);

// Instead: All dependencies listed
useEffect(() => {
  fetchUser(userId).then(setUser);
}, [userId]);
```

### 2. Always Clean Up Side Effects

```typescript
useEffect(() => {
  const controller = new AbortController();
  
  fetchData(controller.signal)
    .then(setData)
    .catch(error => {
      if (error.name !== 'AbortError') {
        console.error(error);
      }
    });

  return () => {
    controller.abort(); // Cleanup
  };
}, []);
```

### 3. Use Custom Hooks for Reusable Lifecycle Logic

```typescript
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}
```

### 4. Type Your Hook Dependencies

```typescript
interface FetchOptions {
  endpoint: string;
  method?: string;
}

const useFetch = <T,>(options: FetchOptions) => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      const response = await fetch(options.endpoint, {
        method: options.method || 'GET',
      });
      const result = await response.json();
      setData(result);
      setLoading(false);
    };

    fetchData();
  }, [options.endpoint, options.method]);

  return { data, loading };
};
```

### 5. Avoid Stale Closures

```typescript
// Problem: count in timeout is stale
const Counter: React.FC = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setTimeout(() => {
      console.log(count); // Stale value
    }, 3000);

    return () => clearTimeout(timer);
  }, []); // count not in dependencies

  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
};

// Solution: Include dependency or use ref
const Counter: React.FC = () => {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);

  useEffect(() => {
    countRef.current = count;
  }, [count]);

  useEffect(() => {
    const timer = setTimeout(() => {
      console.log(countRef.current); // Current value
    }, 3000);

    return () => clearTimeout(timer);
  }, []);

  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
};
```

### 6. Separate Concerns

```typescript
// Bad: Multiple concerns in one effect
useEffect(() => {
  fetchUser(userId).then(setUser);
  subscribeToNotifications(userId);
  logAnalytics('page_view');
}, [userId]);

// Good: Separate effects for separate concerns
useEffect(() => {
  fetchUser(userId).then(setUser);
}, [userId]);

useEffect(() => {
  const unsubscribe = subscribeToNotifications(userId);
  return unsubscribe;
}, [userId]);

useEffect(() => {
  logAnalytics('page_view');
}, []);
```

## React 18+ Features

### Automatic Batching

React 18 batches all state updates automatically, even in async functions.

```typescript
const handleClick = async () => {
  // These updates are batched automatically in React 18
  setCount(c => c + 1);
  setFlag(f => !f);
  await fetchData();
  setData(newData); // Still batched
};
```

### useTransition

Mark state updates as non-urgent to keep the UI responsive.

```typescript
import { useState, useTransition } from 'react';

const SearchComponent: React.FC = () => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<string[]>([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value);
    
    // Mark this update as non-urgent
    startTransition(() => {
      setResults(expensiveSearch(e.target.value));
    });
  };

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <div>Loading...</div>}
      <Results data={results} />
    </div>
  );
};
```

### useDeferredValue

Defer updating a value to keep the UI responsive.

```typescript
import { useState, useDeferredValue } from 'react';

const SearchResults: React.FC = () => {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  // deferredQuery updates are deferred during input
  const results = useMemo(
    () => expensiveSearch(deferredQuery),
    [deferredQuery]
  );

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <Results data={results} />
    </div>
  );
};
```

## Common Patterns

### Data Fetching with Cleanup

```typescript
interface Post {
  id: number;
  title: string;
  body: string;
}

const PostViewer: React.FC<{ postId: number }> = ({ postId }) => {
  const [post, setPost] = useState<Post | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;

    const fetchPost = async () => {
      try {
        setLoading(true);
        setError(null);
        const response = await fetch(`/api/posts/${postId}`);
        const data = await response.json();
        
        if (!cancelled) {
          setPost(data);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err as Error);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchPost();

    return () => {
      cancelled = true;
    };
  }, [postId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{post?.title}</div>;
};
```

### Event Listeners

```typescript
const ClickTracker: React.FC = () => {
  useEffect(() => {
    const handleClick = (e: MouseEvent) => {
      console.log('Clicked at:', e.clientX, e.clientY);
    };

    document.addEventListener('click', handleClick);
    
    return () => {
      document.removeEventListener('click', handleClick);
    };
  }, []);

  return <div>Click anywhere to log coordinates</div>;
};
```

### Intervals and Timers

```typescript
const Timer: React.FC = () => {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);

    return () => clearInterval(interval);
  }, []);

  return <div>Seconds: {seconds}</div>;
};
```
