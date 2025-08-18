# JavaScript Fundamentals - Advanced Concepts

## What is a Web Worker?

**Web Workers** are like having a separate worker thread that runs JavaScript code in the background without blocking the main thread (UI thread).

### **Why Use Web Workers?**
- **Prevent UI freezing** - Heavy calculations don't block the interface
- **Better performance** - Parallel processing
- **Responsive UI** - Main thread stays responsive

### **Example: Stock Data Processing**
```javascript
// Main thread (UI)
const worker = new Worker('stock-worker.js');

// Send data to worker
worker.postMessage({
  stocks: largeStockDataset,
  operation: 'calculatePortfolioRisk'
});

// Receive result from worker
worker.onmessage = function(event) {
  const riskScore = event.data.riskScore;
  updateRiskDisplay(riskScore);
};

// stock-worker.js
self.onmessage = function(event) {
  const { stocks, operation } = event.data;
  
  if (operation === 'calculatePortfolioRisk') {
    // Heavy calculation that would freeze UI
    const riskScore = calculateComplexRiskModel(stocks);
    
    // Send result back to main thread
    self.postMessage({ riskScore });
  }
};

function calculateComplexRiskModel(stocks) {
  // Complex risk calculation that takes time
  return stocks.reduce((risk, stock) => {
    return risk + (stock.volatility * stock.weight);
  }, 0);
}
```

---

## Web Worker vs Service Worker

| Feature | Web Worker | Service Worker |
|---------|------------|----------------|
| **Purpose** | Background processing | Network proxy, caching, offline support |
| **Lifecycle** | Lives with the page | Lives independently of pages |
| **Access** | No DOM access | No DOM access |
| **Network** | Can make network requests | Intercepts network requests |
| **Storage** | No storage access | Can access Cache API, IndexedDB |
| **Use Case** | Heavy calculations | Offline functionality, push notifications |

### **Web Worker Example (Trading Calculations)**
```javascript
// Main thread
const calculationWorker = new Worker('trading-calculator.js');

function calculatePortfolioMetrics(portfolio) {
  calculationWorker.postMessage({
    type: 'PORTFOLIO_ANALYSIS',
    data: portfolio
  });
}

calculationWorker.onmessage = function(event) {
  const { metrics } = event.data;
  updatePortfolioDisplay(metrics);
};

// trading-calculator.js
self.onmessage = function(event) {
  if (event.data.type === 'PORTFOLIO_ANALYSIS') {
    const portfolio = event.data.data;
    const metrics = {
      totalValue: calculateTotalValue(portfolio),
      riskScore: calculateRiskScore(portfolio),
      diversification: calculateDiversification(portfolio)
    };
    self.postMessage({ metrics });
  }
};
```

### **Service Worker Example (Trading App Caching)**
```javascript
// service-worker.js
self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open('trading-cache-v1').then(function(cache) {
      return cache.addAll([
        '/',
        '/static/js/app.js',
        '/static/css/styles.css',
        '/api/stocks/popular'
      ]);
    })
  );
});

self.addEventListener('fetch', function(event) {
  if (event.request.url.includes('/api/stocks/')) {
    event.respondWith(
      caches.match(event.request).then(function(response) {
        return response || fetch(event.request);
      })
    );
  }
});
```

---

## React-Window (Virtual Scrolling)

**React-window** is a library for efficiently rendering large lists and tabular data by only rendering items that are visible.

### **Why Use React-Window?**
- **Performance** - Only renders visible items
- **Memory efficiency** - Doesn't create DOM for all items
- **Smooth scrolling** - Handles large datasets

### **Example: Stock List with Virtual Scrolling**
```jsx
import { FixedSizeList as List } from 'react-window';

function StockList({ stocks }) {
  const Row = ({ index, style }) => {
    const stock = stocks[index];
    return (
      <div style={style} className="stock-row">
        <span className="symbol">{stock.symbol}</span>
        <span className="price">${stock.price}</span>
        <span className={`change ${stock.change >= 0 ? 'positive' : 'negative'}`}>
          {stock.change}%
        </span>
      </div>
    );
  };

  return (
    <List
      height={400}
      itemCount={stocks.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </List>
  );
}

// Usage
const stocks = Array.from({ length: 10000 }, (_, i) => ({
  symbol: `STOCK${i}`,
  price: Math.random() * 1000,
  change: (Math.random() - 0.5) * 10
}));

<StockList stocks={stocks} />
```

---

## Observable Pattern

**Observable** is a design pattern that allows you to subscribe to data streams and react to changes.

### **Simple Observable Implementation**
```javascript
class Observable {
  constructor() {
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
    return () => {
      this.observers = this.observers.filter(obs => obs !== observer);
    };
  }

  notify(data) {
    this.observers.forEach(observer => observer(data));
  }
}

// Trading Example
const stockPriceObservable = new Observable();

// Subscribe to price changes
const unsubscribe = stockPriceObservable.subscribe(price => {
  console.log('Stock price changed:', price);
  updatePriceDisplay(price);
});

// Notify subscribers
stockPriceObservable.notify(150.25);
```

---

## Observer Pattern

**Observer** is a behavioral design pattern where objects subscribe to events and get notified when those events occur.

### **Example: Stock Market Observer**
```javascript
class StockMarket {
  constructor() {
    this.observers = [];
    this.stocks = {};
  }

  addObserver(observer) {
    this.observers.push(observer);
  }

  removeObserver(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }

  notifyObservers(stockSymbol, price) {
    this.observers.forEach(observer => {
      observer.update(stockSymbol, price);
    });
  }

  updateStockPrice(symbol, price) {
    this.stocks[symbol] = price;
    this.notifyObservers(symbol, price);
  }
}

class PriceAlert {
  constructor(symbol, targetPrice) {
    this.symbol = symbol;
    this.targetPrice = targetPrice;
  }

  update(symbol, price) {
    if (symbol === this.symbol && price >= this.targetPrice) {
      console.log(`Alert: ${symbol} reached ${price}!`);
    }
  }
}

// Usage
const market = new StockMarket();
const alert = new PriceAlert('AAPL', 150);
market.addObserver(alert);

market.updateStockPrice('AAPL', 155); // Triggers alert
```

---

## Promises - Deep Dive

**Promises** represent the eventual completion (or failure) of an asynchronous operation. Think of them as a contract that says "I promise to give you a result later."

### **Promise States:**
- **Pending**: Initial state (waiting for result)
- **Fulfilled**: Operation completed successfully (got the result)
- **Rejected**: Operation failed (got an error)

### **How Promises Work:**
```javascript
// Promise constructor takes a function with resolve and reject
const myPromise = new Promise((resolve, reject) => {
  // Do some async work
  const success = Math.random() > 0.5;
  
  if (success) {
    resolve('Operation succeeded!'); // Promise is fulfilled
  } else {
    reject('Operation failed!'); // Promise is rejected
  }
});

// Using the promise
myPromise
  .then(result => console.log('Success:', result))
  .catch(error => console.log('Error:', error));
```

### **Example: Stock API with Promises**
```javascript
function fetchStockPrice(symbol) {
  return new Promise((resolve, reject) => {
    // Simulate API call
    setTimeout(() => {
      const stockData = {
        AAPL: { price: 150.25, change: 2.5 },
        GOOGL: { price: 2800.50, change: -1.2 },
        MSFT: { price: 320.75, change: 0.8 }
      };
      
      if (stockData[symbol]) {
        resolve(stockData[symbol]);
      } else {
        reject(new Error(`Stock ${symbol} not found`));
      }
    }, 1000); // Simulate network delay
  });
}

// Usage
fetchStockPrice('AAPL')
  .then(stockData => {
    console.log('Apple stock price:', stockData.price);
    console.log('Price change:', stockData.change + '%');
    updatePriceDisplay(stockData);
  })
  .catch(error => {
    console.error('Error fetching price:', error);
    showErrorMessage(error.message);
  });
```

### **Promise.all() - Multiple Stock Prices**
```javascript
function fetchMultipleStockPrices(symbols) {
  const promises = symbols.map(symbol => fetchStockPrice(symbol));
  
  return Promise.all(promises)
    .then(stockDataArray => {
      const stockData = symbols.map((symbol, index) => ({
        symbol,
        ...stockDataArray[index]
      }));
      return stockData;
    });
}

// Usage
fetchMultipleStockPrices(['AAPL', 'GOOGL', 'MSFT'])
  .then(stocks => {
    stocks.forEach(stock => {
      console.log(`${stock.symbol}: $${stock.price} (${stock.change}%)`);
    });
  })
  .catch(error => {
    console.error('Error fetching multiple stocks:', error);
  });
```

### **Promise.race() - First to Complete**
```javascript
function fetchStockWithTimeout(symbol, timeout = 5000) {
  const stockPromise = fetchStockPrice(symbol);
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Request timeout')), timeout);
  });
  
  return Promise.race([stockPromise, timeoutPromise]);
}

// Usage
fetchStockWithTimeout('AAPL', 3000)
  .then(stockData => {
    console.log('Stock data received:', stockData);
  })
  .catch(error => {
    console.error('Request failed:', error.message);
  });
```

### **Promise.allSettled() - Wait for All (Success or Failure)**
```javascript
function fetchAllStockData(symbols) {
  const promises = symbols.map(symbol => 
    fetchStockPrice(symbol)
      .then(data => ({ status: 'fulfilled', value: data }))
      .catch(error => ({ status: 'rejected', reason: error }))
  );
  
  return Promise.allSettled(promises);
}

// Usage
fetchAllStockData(['AAPL', 'INVALID', 'GOOGL'])
  .then(results => {
    results.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        console.log(`${symbols[index]}: Success`, result.value);
      } else {
        console.log(`${symbols[index]}: Failed`, result.reason);
      }
    });
  });
```

### **Promise.any() - First Success**
```javascript
function fetchStockFromMultipleSources(symbol) {
  const sources = [
    fetchStockPrice(symbol),
    fetchStockPriceFromBackup(symbol),
    fetchStockPriceFromCache(symbol)
  ];
  
  return Promise.any(sources);
}

// Usage
fetchStockFromMultipleSources('AAPL')
  .then(stockData => {
    console.log('Stock data from fastest source:', stockData);
  })
  .catch(error => {
    console.error('All sources failed:', error);
  });
```

### **Promise Chaining - Complex Operations**
```javascript
function processStockOrder(symbol, quantity) {
  return fetchStockPrice(symbol)
    .then(stockData => {
      console.log('Stock price fetched:', stockData.price);
      return { ...stockData, quantity };
    })
    .then(orderData => {
      const totalCost = orderData.price * orderData.quantity;
      console.log('Total cost:', totalCost);
      return { ...orderData, totalCost };
    })
    .then(orderData => {
      return checkUserBalance(orderData.totalCost)
        .then(hasBalance => ({ ...orderData, hasBalance }));
    })
    .then(orderData => {
      if (orderData.hasBalance) {
        return placeOrder(orderData);
      } else {
        throw new Error('Insufficient balance');
      }
    })
    .then(orderResult => {
      console.log('Order placed successfully:', orderResult);
      return orderResult;
    })
    .catch(error => {
      console.error('Order failed:', error.message);
      throw error; // Re-throw to be handled by caller
    });
}

// Usage
processStockOrder('AAPL', 10)
  .then(result => {
    console.log('Order completed:', result);
  })
  .catch(error => {
    console.error('Order processing failed:', error);
  });
```

### **Error Handling Patterns**
```javascript
// 1. Individual error handling
fetchStockPrice('AAPL')
  .then(stockData => {
    // Handle success
    return stockData;
  })
  .catch(error => {
    // Handle specific error
    console.error('Stock fetch failed:', error);
    return { price: 0, change: 0 }; // Fallback data
  });

// 2. Global error handling
function fetchStockWithGlobalErrorHandling(symbol) {
  return fetchStockPrice(symbol)
    .then(stockData => {
      if (!stockData.price) {
        throw new Error('Invalid stock data');
      }
      return stockData;
    });
}

// 3. Error recovery
function fetchStockWithRetry(symbol, maxRetries = 3) {
  return new Promise((resolve, reject) => {
    let attempts = 0;
    
    function attempt() {
      fetchStockPrice(symbol)
        .then(resolve)
        .catch(error => {
          attempts++;
          if (attempts < maxRetries) {
            console.log(`Retry ${attempts} for ${symbol}`);
            setTimeout(attempt, 1000 * attempts); // Exponential backoff
          } else {
            reject(error);
          }
        });
    }
    
    attempt();
  });
}
```

### **Converting Callbacks to Promises**
```javascript
// Old callback-based function
function fetchStockPriceCallback(symbol, callback) {
  setTimeout(() => {
    const stockData = {
      AAPL: { price: 150.25, change: 2.5 },
      GOOGL: { price: 2800.50, change: -1.2 }
    };
    
    if (stockData[symbol]) {
      callback(null, stockData[symbol]);
    } else {
      callback(new Error(`Stock ${symbol} not found`), null);
    }
  }, 1000);
}

// Convert to Promise
function fetchStockPricePromise(symbol) {
  return new Promise((resolve, reject) => {
    fetchStockPriceCallback(symbol, (error, data) => {
      if (error) {
        reject(error);
      } else {
        resolve(data);
      }
    });
  });
}

// Usage
fetchStockPricePromise('AAPL')
  .then(data => console.log('Success:', data))
  .catch(error => console.error('Error:', error));
```

### **Promise Utilities**
```javascript
// Delay utility
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Usage
delay(2000).then(() => {
  console.log('2 seconds have passed');
});

// Timeout utility
function timeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Timeout')), ms)
    )
  ]);
}

// Usage
timeout(fetchStockPrice('AAPL'), 3000)
  .then(data => console.log('Data received:', data))
  .catch(error => console.error('Request failed:', error));
```

---

## Async/Await

**Async/await** is syntactic sugar over promises that makes asynchronous code look synchronous.

### **Example: Trading Operations**
```javascript
async function placeStockOrder(symbol, quantity, price) {
  try {
    // Check if user has enough balance
    const balance = await getUserBalance();
    const totalCost = quantity * price;
    
    if (balance < totalCost) {
      throw new Error('Insufficient balance');
    }
    
    // Place the order
    const order = await createOrder(symbol, quantity, price);
    
    // Update user balance
    await updateBalance(balance - totalCost);
    
    return order;
  } catch (error) {
    console.error('Order failed:', error);
    throw error;
  }
}

// Usage
async function handleBuyOrder() {
  try {
    const order = await placeStockOrder('AAPL', 10, 150.00);
    console.log('Order placed successfully:', order);
    showSuccessMessage('Order placed!');
  } catch (error) {
    showErrorMessage(error.message);
  }
}
```

### **Parallel Operations with Promise.all()**
```javascript
async function updatePortfolioData() {
  try {
    const [stocks, balance, orders] = await Promise.all([
      fetchUserStocks(),
      getUserBalance(),
      getRecentOrders()
    ]);
    
    updatePortfolioDisplay({ stocks, balance, orders });
  } catch (error) {
    console.error('Failed to update portfolio:', error);
  }
}
```

---

## Callback Functions

**Callbacks** are functions passed as arguments to other functions, executed when the main function completes.

### **Example: Stock Data Processing**
```javascript
function fetchStockData(symbol, callback) {
  fetch(`/api/stocks/${symbol}`)
    .then(response => response.json())
    .then(data => callback(null, data))
    .catch(error => callback(error, null));
}

// Usage
fetchStockData('AAPL', (error, data) => {
  if (error) {
    console.error('Error:', error);
    return;
  }
  console.log('Stock data:', data);
  updateStockDisplay(data);
});
```

### **Callback Hell Problem**
```javascript
// ❌ Bad - Callback Hell
fetchStockData('AAPL', (error, stockData) => {
  if (error) return console.error(error);
  
  fetchUserBalance((error, balance) => {
    if (error) return console.error(error);
    
    calculateOrderCost(stockData.price, 10, (error, cost) => {
      if (error) return console.error(error);
      
      placeOrder('AAPL', 10, cost, (error, order) => {
        if (error) return console.error(error);
        console.log('Order placed:', order);
      });
    });
  });
});
```

### **Solution with Promises/Async-Await**
```javascript
// ✅ Good - Async/Await
async function placeOrderWithAsync() {
  try {
    const stockData = await fetchStockData('AAPL');
    const balance = await getUserBalance();
    const cost = await calculateOrderCost(stockData.price, 10);
    const order = await placeOrder('AAPL', 10, cost);
    
    console.log('Order placed:', order);
  } catch (error) {
    console.error('Error:', error);
  }
}
```

---

## RxJS (Reactive Extensions for JavaScript)

**RxJS** is a library for reactive programming using Observables.

### **Example: Real-time Stock Price Stream**
```javascript
import { fromEvent, interval, merge, map, filter, switchMap } from 'rxjs';

// Create observable from WebSocket
const stockPriceStream = new Observable(observer => {
  const ws = new WebSocket('wss://api.trading.com/prices');
  
  ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    observer.next(data);
  };
  
  ws.onerror = (error) => observer.error(error);
  
  return () => ws.close();
});

// Filter for specific stock
const applePriceStream = stockPriceStream.pipe(
  filter(data => data.symbol === 'AAPL'),
  map(data => data.price)
);

// Subscribe to price changes
applePriceStream.subscribe(price => {
  console.log('Apple price:', price);
  updatePriceDisplay(price);
});
```

### **Debounced Search with RxJS**
```javascript
import { fromEvent, debounceTime, distinctUntilChanged, switchMap } from 'rxjs';

const searchInput = document.getElementById('stock-search');

const searchStream = fromEvent(searchInput, 'input').pipe(
  map(event => event.target.value),
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(searchTerm => fetchStocks(searchTerm))
);

searchStream.subscribe(stocks => {
  displaySearchResults(stocks);
});
```

---

## React Profiler

**React Profiler** is a tool for measuring the performance of React components.

### **Using React Profiler**
```jsx
import { Profiler } from 'react';

function onRenderCallback(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  console.log(`Component ${id} took ${actualDuration}ms to render`);
  
  // Send to analytics
  analytics.track('component-render', {
    component: id,
    duration: actualDuration,
    phase: phase
  });
}

function TradingDashboard() {
  return (
    <Profiler id="TradingDashboard" onRender={onRenderCallback}>
      <StockList />
      <PortfolioSummary />
      <ChartComponent />
    </Profiler>
  );
}
```

### **Custom Performance Hook**
```javascript
function usePerformanceMonitor(componentName) {
  useEffect(() => {
    const start = performance.now();
    
    return () => {
      const end = performance.now();
      const duration = end - start;
      
      if (duration > 16) { // Longer than one frame
        console.warn(`${componentName} took ${duration}ms to render`);
      }
    };
  });
}

// Usage
function StockChart() {
  usePerformanceMonitor('StockChart');
  
  return (
    <div>
      {/* Chart content */}
    </div>
  );
}
```

---

## Common Interview Questions

### **1. "What's the difference between Web Workers and Service Workers?"**
**Answer**: "Web Workers are for CPU-intensive tasks that run in the background without blocking the UI. I use them for heavy calculations like portfolio risk analysis or stock data processing. Service Workers act as network proxies, handling caching, offline functionality, and push notifications. For a trading app, I'd use Web Workers for complex financial calculations and Service Workers for caching stock data and enabling offline portfolio viewing."

### **2. "How do you handle large datasets in React?"**
**Answer**: "I use react-window for virtual scrolling to efficiently render large lists. For example, displaying 10,000 stocks by only rendering the visible items. I also implement pagination, lazy loading, and data virtualization. For real-time data, I use Web Workers for processing and RxJS for managing data streams efficiently."

### **3. "Explain the difference between Promises and Callbacks"**
**Answer**: "Callbacks are functions passed as arguments that execute when an operation completes. They can lead to callback hell with nested functions. Promises provide a cleaner way to handle async operations with .then() and .catch(). Async/await makes promises even more readable by making async code look synchronous. For trading operations, I prefer async/await for order placement and balance updates."

### **4. "What is RxJS and when would you use it?"**
**Answer**: "RxJS is a reactive programming library that uses Observables to handle asynchronous data streams. I use it for real-time stock price updates, user input handling with debouncing, and managing complex state flows. It's particularly useful for trading apps where you need to handle multiple data streams like price feeds, order updates, and user interactions simultaneously."

### **5. "How do you optimize React performance?"**
**Answer**: "I use React Profiler to identify slow components, implement React.memo for component memoization, use useMemo for expensive calculations, and useCallback for function memoization. I also use react-window for large lists, implement code splitting with React.lazy, and use Web Workers for heavy computations. For a trading app, I'd profile chart components and portfolio calculations specifically."

---

## Best Practices

### **Performance:**
- Use Web Workers for CPU-intensive tasks
- Implement virtual scrolling for large lists
- Use React Profiler to identify bottlenecks
- Debounce user inputs with RxJS

### **Error Handling:**
- Always handle promise rejections
- Use try-catch with async/await
- Implement proper error boundaries
- Log errors for debugging

### **Code Organization:**
- Prefer async/await over callbacks
- Use RxJS for complex data streams
- Implement proper cleanup in useEffect
- Use TypeScript for better type safety

---

## Summary

**Key JavaScript Concepts for Trading Apps:**
- 🧵 **Web Workers** - Background processing for heavy calculations
- 🔄 **Service Workers** - Offline functionality and caching
- 📜 **React-window** - Efficient rendering of large datasets
- 👀 **Observables** - Reactive data streams
- ⏳ **Promises/Async-Await** - Clean async code
- 📊 **RxJS** - Complex data stream management
- 🔍 **React Profiler** - Performance monitoring

**Remember**: Choose the right tool for the job and always consider performance implications for real-time trading applications! 