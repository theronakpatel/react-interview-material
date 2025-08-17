# React Performance Optimization Guide

## Why Performance Matters?

**Slow apps = Bad user experience** 🐌
- Users leave if pages take too long to load
- Trading platforms need to be lightning-fast
- Mobile users have limited resources

---

## 1. React.memo - Prevent Unnecessary Re-renders

**What it does**: Stops components from re-rendering if their props haven't changed.

### ❌ **Without React.memo (Bad Performance)**
```tsx
function ExpensiveComponent({ data, onUpdate }) {
  console.log('ExpensiveComponent rendered!'); // Runs every time parent re-renders
  
  return (
    <div>
      <h2>Expensive Component</h2>
      <p>Data: {data}</p>
      <button onClick={onUpdate}>Update</button>
    </div>
  );
}

function Parent() {
  const [count, setCount] = useState(0);
  const [data, setData] = useState('Hello');

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      {/* This re-renders every time count changes, even though data didn't change */}
      <ExpensiveComponent data={data} onUpdate={() => setData('Updated')} />
    </div>
  );
}
```

### ✅ **With React.memo (Good Performance)**
```tsx
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data, onUpdate }) {
  console.log('ExpensiveComponent rendered!'); // Only runs when props actually change
  
  return (
    <div>
      <h2>Expensive Component</h2>
      <p>Data: {data}</p>
      <button onClick={onUpdate}>Update</button>
    </div>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  const [data, setData] = useState('Hello');

  // Memoize the callback to prevent unnecessary re-renders
  const handleUpdate = useCallback(() => {
    setData('Updated');
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      {/* Now this only re-renders when data or handleUpdate changes */}
      <ExpensiveComponent data={data} onUpdate={handleUpdate} />
    </div>
  );
}
```

---

## 2. useMemo - Memoize Expensive Calculations

**What it does**: Saves the result of expensive calculations and only recalculates when dependencies change.

### ❌ **Without useMemo (Bad Performance)**
```tsx
function StockList({ stocks }) {
  // This expensive calculation runs on every render
  const expensiveCalculation = stocks
    .filter(stock => stock.price > 100)
    .map(stock => ({
      ...stock,
      expensiveValue: Math.pow(stock.price, 2) + Math.sqrt(stock.volume)
    }))
    .sort((a, b) => b.expensiveValue - a.expensiveValue);

  return (
    <div>
      <h2>Expensive Stocks</h2>
      {expensiveCalculation.map(stock => (
        <div key={stock.id}>
          {stock.symbol}: ${stock.price} (Value: {stock.expensiveValue.toFixed(2)})
        </div>
      ))}
    </div>
  );
}
```

### ✅ **With useMemo (Good Performance)**
```tsx
function StockList({ stocks }) {
  // This expensive calculation only runs when 'stocks' changes
  const expensiveCalculation = useMemo(() => {
    console.log('Running expensive calculation...'); // Only logs when stocks change
    
    return stocks
      .filter(stock => stock.price > 100)
      .map(stock => ({
        ...stock,
        expensiveValue: Math.pow(stock.price, 2) + Math.sqrt(stock.volume)
      }))
      .sort((a, b) => b.expensiveValue - a.expensiveValue);
  }, [stocks]); // Only recalculate when stocks array changes

  return (
    <div>
      <h2>Expensive Stocks</h2>
      {expensiveCalculation.map(stock => (
        <div key={stock.id}>
          {stock.symbol}: ${stock.price} (Value: {stock.expensiveValue.toFixed(2)})
        </div>
      ))}
    </div>
  );
}
```

---

## 3. useCallback - Memoize Functions

**What it does**: Prevents functions from being recreated on every render.

### ❌ **Without useCallback (Bad Performance)**
```tsx
function StockChart({ stockData, onStockSelect }) {
  const [selectedStock, setSelectedStock] = useState(null);

  // This function is recreated on every render
  const handleStockClick = (stock) => {
    setSelectedStock(stock);
    onStockSelect(stock); // This prop function is also recreated every time
  };

  return (
    <div>
      {stockData.map(stock => (
        <div 
          key={stock.id} 
          onClick={() => handleStockClick(stock)} // New function every time
          className={selectedStock?.id === stock.id ? 'selected' : ''}
        >
          {stock.symbol}: ${stock.price}
        </div>
      ))}
    </div>
  );
}
```

### ✅ **With useCallback (Good Performance)**
```tsx
function StockChart({ stockData, onStockSelect }) {
  const [selectedStock, setSelectedStock] = useState(null);

  // This function is only recreated when onStockSelect changes
  const handleStockClick = useCallback((stock) => {
    setSelectedStock(stock);
    onStockSelect(stock);
  }, [onStockSelect]);

  return (
    <div>
      {stockData.map(stock => (
        <div 
          key={stock.id} 
          onClick={() => handleStockClick(stock)} // Same function reference
          className={selectedStock?.id === stock.id ? 'selected' : ''}
        >
          {stock.symbol}: ${stock.price}
        </div>
      ))}
    </div>
  );
}
```

---

## 4. Virtual Scrolling - Handle Large Lists

**What it does**: Only renders items that are visible on screen.

### ❌ **Without Virtual Scrolling (Bad Performance)**
```tsx
function StockList({ stocks }) {
  return (
    <div>
      <h2>All Stocks ({stocks.length})</h2>
      {/* This renders ALL 10,000 stocks, even if only 10 are visible */}
      {stocks.map(stock => (
        <div key={stock.id} className="stock-item">
          <span>{stock.symbol}</span>
          <span>${stock.price}</span>
          <span>{stock.change}%</span>
        </div>
      ))}
    </div>
  );
}
```

### ✅ **With Virtual Scrolling (Good Performance)**
```tsx
import { FixedSizeList as List } from 'react-window';

function StockList({ stocks }) {
  const Row = ({ index, style }) => {
    const stock = stocks[index];
    return (
      <div style={style} className="stock-item">
        <span>{stock.symbol}</span>
        <span>${stock.price}</span>
        <span>{stock.change}%</span>
      </div>
    );
  };

  return (
    <div>
      <h2>All Stocks ({stocks.length})</h2>
      {/* Only renders ~20 items at a time, regardless of total count */}
      <List
        height={400}
        itemCount={stocks.length}
        itemSize={50}
        width="100%"
      >
        {Row}
      </List>
    </div>
  );
}
```

---

## 5. Lazy Loading - Load Components When Needed

**What it does**: Loads components only when they're actually needed.

### ❌ **Without Lazy Loading (Bad Performance)**
```tsx
import HeavyDashboard from './HeavyDashboard';
import ComplexChart from './ComplexChart';
import DataTable from './DataTable';

function App() {
  return (
    <div>
      {/* All these heavy components load immediately */}
      <HeavyDashboard />
      <ComplexChart />
      <DataTable />
    </div>
  );
}
```

### ✅ **With Lazy Loading (Good Performance)**
```tsx
import { lazy, Suspense } from 'react';

// Lazy load heavy components
const HeavyDashboard = lazy(() => import('./HeavyDashboard'));
const ComplexChart = lazy(() => import('./ComplexChart'));
const DataTable = lazy(() => import('./DataTable'));

function App() {
  const [activeTab, setActiveTab] = useState('dashboard');

  return (
    <div>
      <nav>
        <button onClick={() => setActiveTab('dashboard')}>Dashboard</button>
        <button onClick={() => setActiveTab('chart')}>Chart</button>
        <button onClick={() => setActiveTab('table')}>Table</button>
      </nav>

      <Suspense fallback={<div>Loading...</div>}>
        {activeTab === 'dashboard' && <HeavyDashboard />}
        {activeTab === 'chart' && <ComplexChart />}
        {activeTab === 'table' && <DataTable />}
      </Suspense>
    </div>
  );
}
```

---

## 6. Bundle Optimization

### **Tree Shaking**
```tsx
// ❌ Bad - imports entire library
import _ from 'lodash';

// ✅ Good - imports only what you need
import { debounce } from 'lodash';
```

### **Dynamic Imports**
```tsx
// ❌ Bad - loads everything at once
import { Chart, LineChart, BarChart, PieChart } from 'chart-library';

// ✅ Good - loads only what you need
const Chart = lazy(() => import('chart-library/Chart'));
const LineChart = lazy(() => import('chart-library/LineChart'));
```

---

## 7. Performance Monitoring

### **React DevTools Profiler**
```tsx
import { Profiler } from 'react';

function onRenderCallback(id, phase, actualDuration) {
  console.log(`Component ${id} took ${actualDuration}ms to render`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <YourApp />
    </Profiler>
  );
}
```

### **Performance Hook**
```tsx
function usePerformanceMonitor(componentName) {
  useEffect(() => {
    const start = performance.now();
    
    return () => {
      const end = performance.now();
      console.log(`${componentName} render took ${end - start}ms`);
    };
  });
}
```

---

## 8. Real-World Trading App Example

```tsx
import React, { memo, useMemo, useCallback, lazy, Suspense } from 'react';

// Lazy load heavy components
const StockChart = lazy(() => import('./StockChart'));
const OrderBook = lazy(() => import('./OrderBook'));

// Memoized stock item component
const StockItem = memo(function StockItem({ stock, onSelect, isSelected }) {
  const handleClick = useCallback(() => {
    onSelect(stock);
  }, [stock, onSelect]);

  return (
    <div 
      onClick={handleClick}
      className={`stock-item ${isSelected ? 'selected' : ''}`}
    >
      <span>{stock.symbol}</span>
      <span>${stock.price}</span>
      <span className={stock.change >= 0 ? 'positive' : 'negative'}>
        {stock.change}%
      </span>
    </div>
  );
});

function TradingDashboard({ stocks, selectedStock }) {
  // Memoize expensive calculations
  const topGainers = useMemo(() => {
    return stocks
      .filter(stock => stock.change > 0)
      .sort((a, b) => b.change - a.change)
      .slice(0, 10);
  }, [stocks]);

  const topLosers = useMemo(() => {
    return stocks
      .filter(stock => stock.change < 0)
      .sort((a, b) => a.change - b.change)
      .slice(0, 10);
  }, [stocks]);

  // Memoize callbacks
  const handleStockSelect = useCallback((stock) => {
    // Handle stock selection
  }, []);

  return (
    <div className="trading-dashboard">
      <div className="stock-lists">
        <div className="gainers">
          <h3>Top Gainers</h3>
          {topGainers.map(stock => (
            <StockItem
              key={stock.id}
              stock={stock}
              onSelect={handleStockSelect}
              isSelected={selectedStock?.id === stock.id}
            />
          ))}
        </div>

        <div className="losers">
          <h3>Top Losers</h3>
          {topLosers.map(stock => (
            <StockItem
              key={stock.id}
              stock={stock}
              onSelect={handleStockSelect}
              isSelected={selectedStock?.id === stock.id}
            />
          ))}
        </div>
      </div>

      <Suspense fallback={<div>Loading chart...</div>}>
        {selectedStock && <StockChart stock={selectedStock} />}
      </Suspense>
    </div>
  );
}
```

---

## Performance Checklist

### ✅ **Before Optimization:**
- [ ] Use React DevTools Profiler to identify slow components
- [ ] Check bundle size with webpack-bundle-analyzer
- [ ] Monitor Core Web Vitals (LCP, FID, CLS)

### ✅ **Optimization Techniques:**
- [ ] Use React.memo for expensive components
- [ ] Use useMemo for expensive calculations
- [ ] Use useCallback for function props
- [ ] Implement virtual scrolling for large lists
- [ ] Use lazy loading for heavy components
- [ ] Optimize bundle size with tree shaking

### ✅ **After Optimization:**
- [ ] Test performance improvements
- [ ] Monitor for memory leaks
- [ ] Check mobile performance
- [ ] Validate accessibility isn't broken

---

## Summary

**Performance optimization is crucial for trading apps:**
- 🚀 **React.memo** - Stop unnecessary re-renders
- 🧮 **useMemo** - Cache expensive calculations
- 🔄 **useCallback** - Prevent function recreation
- 📜 **Virtual Scrolling** - Handle large datasets
- ⏳ **Lazy Loading** - Load components when needed
- 📦 **Bundle Optimization** - Reduce initial load time

**Remember**: Always measure performance before and after optimization! 