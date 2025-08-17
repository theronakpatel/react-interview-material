# Custom React Hooks Guide - Easy Examples

## What are Custom Hooks?

Custom hooks are like **reusable functions** that let you share logic between components. Think of them as your own personal tools that you can use over and over again!

**Rule**: Custom hooks must start with "use" (like `useState`, `useEffect`).

---

## 1. useLocalStorage Hook
**What it does**: Saves data in the browser so it doesn't disappear when you refresh the page.

```tsx
import { useState, useEffect } from 'react';

function useLocalStorage<T>(key: string, initialValue: T) {
  // Get the value from localStorage, or use the initial value if nothing is saved
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.log('Error reading from localStorage:', error);
      return initialValue;
    }
  });

  // Save the value to localStorage whenever it changes
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.log('Error saving to localStorage:', error);
    }
  };

  return [storedValue, setValue] as const;
}

// Example Usage:
function UserProfile() {
  const [username, setUsername] = useLocalStorage('username', 'Guest');
  const [theme, setTheme] = useLocalStorage('theme', 'light');

  return (
    <div>
      <input 
        value={username} 
        onChange={(e) => setUsername(e.target.value)} 
        placeholder="Enter your name"
      />
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Switch to {theme === 'light' ? 'Dark' : 'Light'} Mode
      </button>
      <p>Hello, {username}!</p>
    </div>
  );
}
```

---

## 2. useDebounce Hook
**What it does**: Waits until you stop typing before doing something (like searching). This prevents too many API calls while you're typing.

```tsx
import { useState, useEffect } from 'react';

function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    // Set a timer to update the debounced value after the delay
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Cancel the timer if the value changes before the delay is up
    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Example Usage:
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500); // Wait 500ms

  useEffect(() => {
    if (debouncedSearchTerm) {
      // Only search after user stops typing for 500ms
      searchAPI(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      type="text"
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

## 3. useWindowSize Hook
**What it does**: Tells you the size of the browser window and updates when the window is resized.

```tsx
import { useState, useEffect } from 'react';

interface WindowSize {
  width: number;
  height: number;
}

function useWindowSize(): WindowSize {
  const [windowSize, setWindowSize] = useState<WindowSize>({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    // Listen for window resize events
    window.addEventListener('resize', handleResize);
    
    // Clean up the event listener when component unmounts
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}

// Example Usage:
function ResponsiveComponent() {
  const { width, height } = useWindowSize();

  return (
    <div>
      <h2>Window Size</h2>
      <p>Width: {width}px</p>
      <p>Height: {height}px</p>
      
      {width < 768 ? (
        <p>📱 Mobile view</p>
      ) : width < 1024 ? (
        <p>💻 Tablet view</p>
      ) : (
        <p>🖥️ Desktop view</p>
      )}
    </div>
  );
}
```

---

## 4. useClickOutside Hook
**What it does**: Detects when someone clicks outside of a specific element (useful for closing dropdowns, modals, etc.).

```tsx
import { useEffect, useRef } from 'react';

function useClickOutside(callback: () => void) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    function handleClickOutside(event: MouseEvent) {
      if (ref.current && !ref.current.contains(event.target as Node)) {
        callback();
      }
    }

    // Listen for clicks on the document
    document.addEventListener('mousedown', handleClickOutside);
    
    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, [callback]);

  return ref;
}

// Example Usage:
function DropdownMenu() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useClickOutside(() => setIsOpen(false));

  return (
    <div ref={dropdownRef} className="dropdown">
      <button onClick={() => setIsOpen(!isOpen)}>
        Menu ▼
      </button>
      
      {isOpen && (
        <div className="dropdown-menu">
          <a href="#">Profile</a>
          <a href="#">Settings</a>
          <a href="#">Logout</a>
        </div>
      )}
    </div>
  );
}
```

---

## 5. useOnlineStatus Hook
**What it does**: Tells you if the user is online or offline.

```tsx
import { useState, useEffect } from 'react';

function useOnlineStatus(): boolean {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }

    function handleOffline() {
      setIsOnline(false);
    }

    // Listen for online/offline events
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}

// Example Usage:
function ConnectionStatus() {
  const isOnline = useOnlineStatus();

  return (
    <div className={`status ${isOnline ? 'online' : 'offline'}`}>
      {isOnline ? (
        <span>🟢 You are online</span>
      ) : (
        <span>🔴 You are offline</span>
      )}
    </div>
  );
}
```

---

## 6. usePrevious Hook
**What it does**: Remembers the previous value of something (useful for comparing old vs new values).

```tsx
import { useRef, useEffect } from 'react';

function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = value;
  });

  return ref.current;
}

// Example Usage:
function Counter() {
  const [count, setCount] = useState(0);
  const previousCount = usePrevious(count);

  return (
    <div>
      <h2>Count: {count}</h2>
      <p>Previous count was: {previousCount}</p>
      
      {previousCount !== undefined && (
        <p>
          {count > previousCount ? '📈 Increased' : '📉 Decreased'}
        </p>
      )}
      
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(count - 1)}>-1</button>
    </div>
  );
}
```

---

## 7. useToggle Hook
**What it does**: Creates a simple on/off switch (like a light switch).

```tsx
import { useState, useCallback } from 'react';

function useToggle(initialValue: boolean = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => {
    setValue(prev => !prev);
  }, []);

  const setTrue = useCallback(() => {
    setValue(true);
  }, []);

  const setFalse = useCallback(() => {
    setValue(false);
  }, []);

  return [value, toggle, setTrue, setFalse] as const;
}

// Example Usage:
function LightSwitch() {
  const [isOn, toggle, turnOn, turnOff] = useToggle(false);

  return (
    <div className={`light ${isOn ? 'on' : 'off'}`}>
      <h2>Light Switch</h2>
      <p>The light is {isOn ? 'ON' : 'OFF'}</p>
      
      <button onClick={toggle}>
        Toggle Light
      </button>
      
      <button onClick={turnOn}>
        Turn On
      </button>
      
      <button onClick={turnOff}>
        Turn Off
      </button>
    </div>
  );
}
```

---

## 8. useFetch Hook
**What it does**: Makes API calls and handles loading, error, and success states.

```tsx
import { useState, useEffect } from 'react';

interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    async function fetchData() {
      try {
        setState(prev => ({ ...prev, loading: true, error: null }));
        
        const response = await fetch(url);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        
        setState({
          data,
          loading: false,
          error: null,
        });
      } catch (error) {
        setState({
          data: null,
          loading: false,
          error: error instanceof Error ? error.message : 'An error occurred',
        });
      }
    }

    fetchData();
  }, [url]);

  return state;
}

// Example Usage:
function UserProfile() {
  const { data: user, loading, error } = useFetch<User>('/api/user/1');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Age: {user.age}</p>
    </div>
  );
}
```

---

## 9. useForm Hook
**What it does**: Handles form inputs, validation, and submission easily.

```tsx
import { useState, useCallback } from 'react';

interface FormData {
  [key: string]: string;
}

interface FormErrors {
  [key: string]: string;
}

function useForm(initialValues: FormData = {}) {
  const [values, setValues] = useState<FormData>(initialValues);
  const [errors, setErrors] = useState<FormErrors>({});

  const handleChange = useCallback((name: string, value: string) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  }, [errors]);

  const validate = useCallback((validationRules: Record<string, (value: string) => string | null>) => {
    const newErrors: FormErrors = {};
    
    Object.keys(validationRules).forEach(field => {
      const error = validationRules[field](values[field] || '');
      if (error) {
        newErrors[field] = error;
      }
    });
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }, [values]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
  }, [initialValues]);

  return {
    values,
    errors,
    handleChange,
    validate,
    reset,
  };
}

// Example Usage:
function ContactForm() {
  const { values, errors, handleChange, validate, reset } = useForm({
    name: '',
    email: '',
    message: '',
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    const isValid = validate({
      name: (value) => !value ? 'Name is required' : null,
      email: (value) => !value ? 'Email is required' : !value.includes('@') ? 'Invalid email' : null,
      message: (value) => !value ? 'Message is required' : value.length < 10 ? 'Message too short' : null,
    });

    if (isValid) {
      console.log('Form submitted:', values);
      reset();
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="text"
          placeholder="Name"
          value={values.name}
          onChange={(e) => handleChange('name', e.target.value)}
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>

      <div>
        <input
          type="email"
          placeholder="Email"
          value={values.email}
          onChange={(e) => handleChange('email', e.target.value)}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div>
        <textarea
          placeholder="Message"
          value={values.message}
          onChange={(e) => handleChange('message', e.target.value)}
        />
        {errors.message && <span className="error">{errors.message}</span>}
      </div>

      <button type="submit">Send Message</button>
    </form>
  );
}
```

---

## 10. useInterval Hook
**What it does**: Runs a function repeatedly at a set interval (like a timer or polling).

```tsx
import { useEffect, useRef } from 'react';

function useInterval(callback: () => void, delay: number | null) {
  const savedCallback = useRef(callback);

  // Remember the latest callback
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  // Set up the interval
  useEffect(() => {
    // Don't schedule if no delay is specified
    if (delay === null) {
      return;
    }

    const id = setInterval(() => savedCallback.current(), delay);

    return () => clearInterval(id);
  }, [delay]);
}

// Example Usage:
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useInterval(
    () => {
      setSeconds(s => s + 1);
    },
    isRunning ? 1000 : null // Run every 1000ms (1 second) if isRunning is true
  );

  return (
    <div>
      <h2>Timer: {seconds} seconds</h2>
      
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Start'}
      </button>
      
      <button onClick={() => setSeconds(0)}>
        Reset
      </button>
    </div>
  );
}
```

---

## 11. useKeyPress Hook
**What it does**: Detects when specific keys are pressed on the keyboard.

```tsx
import { useState, useEffect } from 'react';

function useKeyPress(targetKey: string): boolean {
  const [keyPressed, setKeyPressed] = useState(false);

  useEffect(() => {
    function downHandler({ key }: KeyboardEvent) {
      if (key === targetKey) {
        setKeyPressed(true);
      }
    }

    function upHandler({ key }: KeyboardEvent) {
      if (key === targetKey) {
        setKeyPressed(false);
      }
    }

    window.addEventListener('keydown', downHandler);
    window.addEventListener('keyup', upHandler);

    return () => {
      window.removeEventListener('keydown', downHandler);
      window.removeEventListener('keyup', upHandler);
    };
  }, [targetKey]);

  return keyPressed;
}

// Example Usage:
function KeyboardGame() {
  const wPressed = useKeyPress('w');
  const aPressed = useKeyPress('a');
  const sPressed = useKeyPress('s');
  const dPressed = useKeyPress('d');

  return (
    <div>
      <h2>Press WASD keys</h2>
      <div className="controls">
        <div className={`key ${wPressed ? 'pressed' : ''}`}>W</div>
        <div className={`key ${aPressed ? 'pressed' : ''}`}>A</div>
        <div className={`key ${sPressed ? 'pressed' : ''}`}>S</div>
        <div className={`key ${dPressed ? 'pressed' : ''}`}>D</div>
      </div>
      
      <p>
        {wPressed && 'Moving up! '}
        {aPressed && 'Moving left! '}
        {sPressed && 'Moving down! '}
        {dPressed && 'Moving right! '}
      </p>
    </div>
  );
}
```

---

## 12. useMediaQuery Hook
**What it does**: Detects if the screen matches certain conditions (like screen size, dark mode, etc.).

```tsx
import { useState, useEffect } from 'react';

function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    
    // Set initial value
    setMatches(media.matches);

    // Create event listener
    const listener = (event: MediaQueryListEvent) => {
      setMatches(event.matches);
    };

    // Add listener
    media.addEventListener('change', listener);

    // Clean up
    return () => media.removeEventListener('change', listener);
  }, [query]);

  return matches;
}

// Example Usage:
function ResponsiveApp() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 769px) and (max-width: 1024px)');
  const isDesktop = useMediaQuery('(min-width: 1025px)');
  const prefersDark = useMediaQuery('(prefers-color-scheme: dark)');

  return (
    <div className={`app ${prefersDark ? 'dark' : 'light'}`}>
      <h2>Responsive App</h2>
      
      {isMobile && <p>📱 Mobile Layout</p>}
      {isTablet && <p>💻 Tablet Layout</p>}
      {isDesktop && <p>🖥️ Desktop Layout</p>}
      
      <p>Theme: {prefersDark ? 'Dark' : 'Light'} Mode</p>
    </div>
  );
}
```

---

## Summary

These custom hooks help you:

1. **Reuse logic** between components
2. **Keep components clean** and focused
3. **Make code more readable** and maintainable
4. **Avoid repeating code** (DRY principle)

**Remember**: Always start custom hooks with "use" and follow React's rules of hooks!

**Pro Tip**: You can combine multiple hooks to create powerful, reusable functionality for your applications. 