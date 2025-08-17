# Complete React Hooks Guide - All Available Hooks

## What are React Hooks?

**React Hooks** are functions that let you "hook into" React state and lifecycle features from function components. Think of them as special tools that give function components the same powers that class components have.

---

## Built-in React Hooks

### 1. useState - State Management

**What it does**: Manages local state in function components.

```tsx
import { useState } from 'react';

function StockPriceTracker() {
  const [price, setPrice] = useState(150.00);
  const [isWatching, setIsWatching] = useState(false);

  return (
    <div>
      <h2>Stock Price: ${price}</h2>
      <button onClick={() => setPrice(price + 1)}>
        Price +$1
      </button>
      <button onClick={() => setPrice(price - 1)}>
        Price -$1
      </button>
      <button onClick={() => setIsWatching(!isWatching)}>
        {isWatching ? 'Stop Watching' : 'Start Watching'}
      </button>
      {isWatching && <p>👀 Watching price changes...</p>}
    </div>
  );
}
```

### 2. useEffect - Side Effects

**What it does**: Handles side effects like API calls, subscriptions, or DOM updates.

```tsx
import { useState, useEffect } from 'react';

function StockDataFetcher() {
  const [stockData, setStockData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Fetch stock data when component mounts
    fetchStockData('AAPL')
      .then(data => {
        setStockData(data);
        setLoading(false);
      })
      .catch(error => {
        console.error('Error fetching stock data:', error);
        setLoading(false);
      });
  }, []); // Empty dependency array = run only once

  useEffect(() => {
    // Set up WebSocket connection for real-time updates
    const ws = new WebSocket('wss://api.stocks.com/price');
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setStockData(data);
    };

    // Cleanup: close connection when component unmounts
    return () => {
      ws.close();
    };
  }, []);

  if (loading) return <div>Loading stock data...</div>;
  if (!stockData) return <div>No data available</div>;

  return (
    <div>
      <h2>{stockData.symbol}</h2>
      <p>Price: ${stockData.price}</p>
      <p>Change: {stockData.change}%</p>
    </div>
  );
}
```

### 3. useContext - Context Consumption

**What it does**: Consumes values from React Context.

```tsx
import { createContext, useContext, useState } from 'react';

// Create context
const TradingContext = createContext();

// Provider component
function TradingProvider({ children }) {
  const [user, setUser] = useState({ name: 'John', balance: 10000 });
  const [theme, setTheme] = useState('light');

  return (
    <TradingContext.Provider value={{ user, setUser, theme, setTheme }}>
      {children}
    </TradingContext.Provider>
  );
}

// Consumer component
function PortfolioSummary() {
  const { user, theme } = useContext(TradingContext);

  return (
    <div className={`portfolio ${theme}`}>
      <h2>Welcome, {user.name}!</h2>
      <p>Account Balance: ${user.balance.toLocaleString()}</p>
    </div>
  );
}

// Usage
function App() {
  return (
    <TradingProvider>
      <PortfolioSummary />
    </TradingProvider>
  );
}
```

### 4. useReducer - Complex State Logic

**What it does**: Manages complex state logic with reducers.

```tsx
import { useReducer } from 'react';

// Action types
const ACTIONS = {
  BUY_STOCK: 'BUY_STOCK',
  SELL_STOCK: 'SELL_STOCK',
  UPDATE_PRICE: 'UPDATE_PRICE'
};

// Reducer function
function portfolioReducer(state, action) {
  switch (action.type) {
    case ACTIONS.BUY_STOCK:
      const totalCost = action.quantity * action.price;
      if (state.balance >= totalCost) {
        return {
          ...state,
          balance: state.balance - totalCost,
          stocks: [...state.stocks, {
            symbol: action.symbol,
            quantity: action.quantity,
            avgPrice: action.price
          }]
        };
      }
      return state;

    case ACTIONS.SELL_STOCK:
      const stock = state.stocks.find(s => s.symbol === action.symbol);
      if (stock && stock.quantity >= action.quantity) {
        const totalValue = action.quantity * action.price;
        return {
          ...state,
          balance: state.balance + totalValue,
          stocks: state.stocks.map(s => 
            s.symbol === action.symbol 
              ? { ...s, quantity: s.quantity - action.quantity }
              : s
          ).filter(s => s.quantity > 0)
        };
      }
      return state;

    case ACTIONS.UPDATE_PRICE:
      return {
        ...state,
        currentPrices: {
          ...state.currentPrices,
          [action.symbol]: action.price
        }
      };

    default:
      return state;
  }
}

function PortfolioManager() {
  const [portfolio, dispatch] = useReducer(portfolioReducer, {
    balance: 10000,
    stocks: [],
    currentPrices: {}
  });

  const buyStock = (symbol, quantity, price) => {
    dispatch({ type: ACTIONS.BUY_STOCK, symbol, quantity, price });
  };

  const sellStock = (symbol, quantity, price) => {
    dispatch({ type: ACTIONS.SELL_STOCK, symbol, quantity, price });
  };

  return (
    <div>
      <h2>Portfolio Balance: ${portfolio.balance}</h2>
      <button onClick={() => buyStock('AAPL', 10, 150)}>
        Buy 10 AAPL at $150
      </button>
      <button onClick={() => sellStock('AAPL', 5, 160)}>
        Sell 5 AAPL at $160
      </button>
      
      <h3>Your Stocks:</h3>
      {portfolio.stocks.map(stock => (
        <div key={stock.symbol}>
          {stock.symbol}: {stock.quantity} shares @ ${stock.avgPrice}
        </div>
      ))}
    </div>
  );
}
```

### 5. useCallback - Memoized Functions

**What it does**: Returns a memoized callback function.

```tsx
import { useState, useCallback } from 'react';

function StockOrderForm() {
  const [quantity, setQuantity] = useState(0);
  const [price, setPrice] = useState(0);

  // Memoized function - only recreates when quantity or price changes
  const handleBuyOrder = useCallback(() => {
    console.log(`Buying ${quantity} shares at $${price}`);
    // Place buy order logic here
  }, [quantity, price]);

  const handleSellOrder = useCallback(() => {
    console.log(`Selling ${quantity} shares at $${price}`);
    // Place sell order logic here
  }, [quantity, price]);

  return (
    <div>
      <input
        type="number"
        placeholder="Quantity"
        value={quantity}
        onChange={(e) => setQuantity(Number(e.target.value))}
      />
      <input
        type="number"
        placeholder="Price"
        value={price}
        onChange={(e) => setPrice(Number(e.target.value))}
      />
      <button onClick={handleBuyOrder}>Buy</button>
      <button onClick={handleSellOrder}>Sell</button>
    </div>
  );
}
```

### 6. useMemo - Memoized Values

**What it does**: Returns a memoized value.

```tsx
import { useState, useMemo } from 'react';

function PortfolioCalculator({ stocks }) {
  const [riskTolerance, setRiskTolerance] = useState('medium');

  // Expensive calculation - only runs when stocks or riskTolerance changes
  const portfolioMetrics = useMemo(() => {
    console.log('Calculating portfolio metrics...');
    
    const totalValue = stocks.reduce((sum, stock) => 
      sum + (stock.quantity * stock.price), 0
    );
    
    const totalCost = stocks.reduce((sum, stock) => 
      sum + (stock.quantity * stock.avgPrice), 0
    );
    
    const profit = totalValue - totalCost;
    const profitPercentage = (profit / totalCost) * 100;
    
    // Risk calculation based on tolerance
    const riskMultiplier = riskTolerance === 'high' ? 1.5 : 
                          riskTolerance === 'medium' ? 1.0 : 0.5;
    
    const riskScore = stocks.reduce((risk, stock) => 
      risk + (stock.volatility * riskMultiplier), 0
    );

    return {
      totalValue,
      profit,
      profitPercentage,
      riskScore
    };
  }, [stocks, riskTolerance]);

  return (
    <div>
      <select value={riskTolerance} onChange={(e) => setRiskTolerance(e.target.value)}>
        <option value="low">Low Risk</option>
        <option value="medium">Medium Risk</option>
        <option value="high">High Risk</option>
      </select>
      
      <h3>Portfolio Summary</h3>
      <p>Total Value: ${portfolioMetrics.totalValue.toFixed(2)}</p>
      <p>Profit: ${portfolioMetrics.profit.toFixed(2)} ({portfolioMetrics.profitPercentage.toFixed(2)}%)</p>
      <p>Risk Score: {portfolioMetrics.riskScore.toFixed(2)}</p>
    </div>
  );
}
```

### 7. useRef - Mutable References

**What it does**: Creates a mutable reference that persists across renders.

```tsx
import { useRef, useEffect } from 'react';

function StockChart() {
  const canvasRef = useRef(null);
  const animationRef = useRef(null);

  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    
    // Draw stock chart
    const drawChart = () => {
      // Chart drawing logic here
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      // Draw price line, candlesticks, etc.
      
      // Continue animation
      animationRef.current = requestAnimationFrame(drawChart);
    };
    
    drawChart();
    
    // Cleanup animation on unmount
    return () => {
      if (animationRef.current) {
        cancelAnimationFrame(animationRef.current);
      }
    };
  }, []);

  return <canvas ref={canvasRef} width={800} height={400} />;
}

// Another useRef example - focus management
function StockSearch() {
  const inputRef = useRef(null);

  const focusSearch = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <input ref={inputRef} placeholder="Search stocks..." />
      <button onClick={focusSearch}>Focus Search</button>
    </div>
  );
}
```

### 8. useLayoutEffect - Synchronous Effects

**What it does**: Similar to useEffect but runs synchronously after DOM mutations.

```tsx
import { useState, useLayoutEffect, useRef } from 'react';

function StockPriceDisplay({ price }) {
  const [displayPrice, setDisplayPrice] = useState(price);
  const priceRef = useRef(null);

  useLayoutEffect(() => {
    // Measure the element before DOM updates
    const element = priceRef.current;
    if (element) {
      const width = element.offsetWidth;
      
      // Adjust font size if price is too long
      if (width > 200) {
        element.style.fontSize = '14px';
      } else {
        element.style.fontSize = '18px';
      }
    }
  }, [price]);

  return (
    <div ref={priceRef} className="price-display">
      ${displayPrice}
    </div>
  );
}
```

### 9. useImperativeHandle - Custom Instance Value

**What it does**: Customizes the instance value that is exposed to parent components.

```tsx
import { forwardRef, useImperativeHandle, useState } from 'react';

const StockOrderForm = forwardRef((props, ref) => {
  const [orderData, setOrderData] = useState({ symbol: '', quantity: 0, price: 0 });

  // Expose specific methods to parent component
  useImperativeHandle(ref, () => ({
    submitOrder: () => {
      console.log('Submitting order:', orderData);
      // Submit order logic
    },
    clearForm: () => {
      setOrderData({ symbol: '', quantity: 0, price: 0 });
    },
    getOrderData: () => orderData
  }));

  return (
    <div>
      <input
        placeholder="Stock Symbol"
        value={orderData.symbol}
        onChange={(e) => setOrderData({ ...orderData, symbol: e.target.value })}
      />
      <input
        type="number"
        placeholder="Quantity"
        value={orderData.quantity}
        onChange={(e) => setOrderData({ ...orderData, quantity: Number(e.target.value) })}
      />
      <input
        type="number"
        placeholder="Price"
        value={orderData.price}
        onChange={(e) => setOrderData({ ...orderData, price: Number(e.target.value) })}
      />
    </div>
  );
});

// Parent component
function TradingDashboard() {
  const orderFormRef = useRef();

  const handleSubmit = () => {
    orderFormRef.current?.submitOrder();
  };

  const handleClear = () => {
    orderFormRef.current?.clearForm();
  };

  return (
    <div>
      <StockOrderForm ref={orderFormRef} />
      <button onClick={handleSubmit}>Submit Order</button>
      <button onClick={handleClear}>Clear Form</button>
    </div>
  );
}
```

### 10. useDebugValue - Debug Information

**What it does**: Displays a label for custom hooks in React DevTools.

```tsx
import { useState, useDebugValue } from 'react';

function useStockPrice(symbol) {
  const [price, setPrice] = useState(null);
  const [loading, setLoading] = useState(true);

  // This will show in React DevTools
  useDebugValue(price ? `$${price}` : 'Loading...');

  useEffect(() => {
    fetchStockPrice(symbol)
      .then(setPrice)
      .finally(() => setLoading(false));
  }, [symbol]);

  return { price, loading };
}

function StockDisplay({ symbol }) {
  const { price, loading } = useStockPrice(symbol);

  if (loading) return <div>Loading...</div>;
  return <div>{symbol}: ${price}</div>;
}
```

---

## Custom Hooks (Built with Built-in Hooks)

### 11. useLocalStorage - Persistent State

```tsx
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('Error reading from localStorage:', error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('Error saving to localStorage:', error);
    }
  };

  return [storedValue, setValue];
}

// Usage
function TradingPreferences() {
  const [theme, setTheme] = useLocalStorage('trading-theme', 'light');
  const [defaultStock, setDefaultStock] = useLocalStorage('default-stock', 'AAPL');

  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light Theme</option>
        <option value="dark">Dark Theme</option>
      </select>
      <input
        value={defaultStock}
        onChange={(e) => setDefaultStock(e.target.value)}
        placeholder="Default stock symbol"
      />
    </div>
  );
}
```

### 12. useDebounce - Delayed Updates

```tsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function StockSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);

  useEffect(() => {
    if (debouncedSearchTerm) {
      searchStocks(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search stocks..."
    />
  );
}
```

### 13. useOnlineStatus - Network Status

```tsx
import { useState, useEffect } from 'react';

function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}

// Usage
function TradingStatus() {
  const isOnline = useOnlineStatus();

  return (
    <div className={`status ${isOnline ? 'online' : 'offline'}`}>
      {isOnline ? '🟢 Live Trading' : '🔴 Offline Mode'}
    </div>
  );
}
```

### 14. usePrevious - Previous Value

```tsx
import { useRef, useEffect } from 'react';

function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  });

  return ref.current;
}

// Usage
function StockPriceChange({ price }) {
  const previousPrice = usePrevious(price);

  return (
    <div>
      <p>Current: ${price}</p>
      <p>Previous: ${previousPrice}</p>
      {previousPrice && (
        <p className={price > previousPrice ? 'positive' : 'negative'}>
          {price > previousPrice ? '📈' : '📉'} 
          {((price - previousPrice) / previousPrice * 100).toFixed(2)}%
        </p>
      )}
    </div>
  );
}
```

### 15. useInterval - Timed Effects

```tsx
import { useEffect, useRef } from 'react';

function useInterval(callback, delay) {
  const savedCallback = useRef();

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    if (delay !== null) {
      const id = setInterval(() => savedCallback.current(), delay);
      return () => clearInterval(id);
    }
  }, [delay]);
}

// Usage
function StockPriceTicker({ symbol }) {
  const [price, setPrice] = useState(null);

  useInterval(() => {
    fetchStockPrice(symbol).then(setPrice);
  }, 5000); // Update every 5 seconds

  return <div>{symbol}: ${price || 'Loading...'}</div>;
}
```

---

## Hook Rules and Best Practices

### **Rules of Hooks:**
1. **Only call hooks at the top level** - Don't call inside loops, conditions, or nested functions
2. **Only call hooks from React functions** - Call from React function components or custom hooks
3. **Hook names must start with "use"** - This tells React it's a hook

### **Good Practices:**
- **Use multiple useState calls** instead of one large state object
- **Use useCallback** for functions passed to optimized child components
- **Use useMemo** for expensive calculations
- **Clean up effects** to prevent memory leaks
- **Create custom hooks** to reuse stateful logic

### **Common Mistakes:**
```tsx
// ❌ Bad - Calling hooks conditionally
function BadComponent({ shouldFetch }) {
  if (shouldFetch) {
    useEffect(() => {
      // This breaks the rules of hooks
    }, []);
  }
}

// ✅ Good - Always call hooks
function GoodComponent({ shouldFetch }) {
  useEffect(() => {
    if (shouldFetch) {
      // Fetch data
    }
  }, [shouldFetch]);
}
```

---

## Summary

**Essential Hooks for Trading Apps:**
- 🎯 **useState** - Local state management
- 🔄 **useEffect** - Side effects and API calls
- 🌐 **useContext** - Global state sharing
- 🧮 **useReducer** - Complex state logic
- ⚡ **useCallback** - Optimized functions
- 💾 **useMemo** - Cached calculations
- 🔗 **useRef** - DOM references and mutable values
- 🎨 **useLayoutEffect** - Synchronous DOM updates
- 🎛️ **useImperativeHandle** - Custom component APIs
- 🐛 **useDebugValue** - Debug information

**Custom Hooks for Trading:**
- 💾 **useLocalStorage** - Persistent user preferences
- ⏱️ **useDebounce** - Optimized search
- 🌐 **useOnlineStatus** - Network connectivity
- 📊 **usePrevious** - Price change tracking
- ⏰ **useInterval** - Real-time updates

**Remember**: Hooks make function components powerful and are essential for modern React development! 