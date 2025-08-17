# Practical Coding Challenges for Aries Interview

## Challenge 1: Real-time Stock Price Component

### Requirements:
- Display real-time stock prices with WebSocket connection
- Handle connection failures gracefully
- Show price changes with color coding
- Implement reconnection logic
- Add loading states

### React Implementation:
```tsx
import React, { useState, useEffect, useCallback } from 'react';

interface StockData {
  symbol: string;
  price: number;
  change: number;
  changePercent: number;
}

interface StockPriceProps {
  symbol: string;
  initialPrice?: number;
}

const StockPrice: React.FC<StockPriceProps> = ({ symbol, initialPrice = 0 }) => {
  const [stockData, setStockData] = useState<StockData | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [ws, setWs] = useState<WebSocket | null>(null);

  const connectWebSocket = useCallback(() => {
    try {
      const websocket = new WebSocket(`wss://api.aries.com/stock/${symbol}`);
      
      websocket.onopen = () => {
        setIsConnected(true);
        setError(null);
        setIsLoading(false);
      };

      websocket.onmessage = (event) => {
        const data: StockData = JSON.parse(event.data);
        setStockData(data);
      };

      websocket.onclose = () => {
        setIsConnected(false);
        // Auto-reconnect after 3 seconds
        setTimeout(() => {
          if (!isConnected) {
            connectWebSocket();
          }
        }, 3000);
      };

      websocket.onerror = () => {
        setError('Connection failed');
        setIsLoading(false);
      };

      setWs(websocket);
    } catch (err) {
      setError('Failed to establish connection');
      setIsLoading(false);
    }
  }, [symbol, isConnected]);

  useEffect(() => {
    connectWebSocket();
    return () => {
      if (ws) {
        ws.close();
      }
    };
  }, [connectWebSocket]);

  if (isLoading) {
    return <div className="stock-price loading">Loading...</div>;
  }

  if (error) {
    return <div className="stock-price error">{error}</div>;
  }

  if (!stockData) {
    return <div className="stock-price">No data available</div>;
  }

  const isPositive = stockData.change >= 0;
  const connectionStatus = isConnected ? '🟢' : '🔴';

  return (
    <div className="stock-price">
      <div className="stock-header">
        <span className="symbol">{stockData.symbol}</span>
        <span className="connection-status">{connectionStatus}</span>
      </div>
      <div className="price-info">
        <span className="price">${stockData.price.toFixed(2)}</span>
        <span className={`change ${isPositive ? 'positive' : 'negative'}`}>
          {isPositive ? '+' : ''}{stockData.change.toFixed(2)} ({stockData.changePercent.toFixed(2)}%)
        </span>
      </div>
    </div>
  );
};

export default StockPrice;
```

### Vue Implementation:
```vue
<template>
  <div class="stock-price">
    <div v-if="isLoading" class="loading">Loading...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else-if="!stockData" class="no-data">No data available</div>
    <div v-else class="stock-content">
      <div class="stock-header">
        <span class="symbol">{{ stockData.symbol }}</span>
        <span class="connection-status">{{ connectionStatusIcon }}</span>
      </div>
      <div class="price-info">
        <span class="price">${{ stockData.price.toFixed(2) }}</span>
        <span :class="['change', { positive: isPositive, negative: !isPositive }]">
          {{ isPositive ? '+' : '' }}{{ stockData.change.toFixed(2) }} ({{ stockData.changePercent.toFixed(2) }}%)
        </span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';

interface StockData {
  symbol: string;
  price: number;
  change: number;
  changePercent: number;
}

interface Props {
  symbol: string;
  initialPrice?: number;
}

const props = withDefaults(defineProps<Props>(), {
  initialPrice: 0
});

const stockData = ref<StockData | null>(null);
const isConnected = ref(false);
const isLoading = ref(true);
const error = ref<string | null>(null);
const ws = ref<WebSocket | null>(null);

const isPositive = computed(() => stockData.value?.change >= 0);
const connectionStatusIcon = computed(() => isConnected.value ? '🟢' : '🔴');

const connectWebSocket = () => {
  try {
    const websocket = new WebSocket(`wss://api.aries.com/stock/${props.symbol}`);
    
    websocket.onopen = () => {
      isConnected.value = true;
      error.value = null;
      isLoading.value = false;
    };

    websocket.onmessage = (event) => {
      const data: StockData = JSON.parse(event.data);
      stockData.value = data;
    };

    websocket.onclose = () => {
      isConnected.value = false;
      // Auto-reconnect after 3 seconds
      setTimeout(() => {
        if (!isConnected.value) {
          connectWebSocket();
        }
      }, 3000);
    };

    websocket.onerror = () => {
      error.value = 'Connection failed';
      isLoading.value = false;
    };

    ws.value = websocket;
  } catch (err) {
    error.value = 'Failed to establish connection';
    isLoading.value = false;
  }
};

onMounted(() => {
  connectWebSocket();
});

onUnmounted(() => {
  if (ws.value) {
    ws.value.close();
  }
});
</script>
```

## Challenge 2: Virtual Scrolling for Large Datasets

### Requirements:
- Handle thousands of stock symbols efficiently
- Smooth scrolling performance
- Search functionality
- Keyboard navigation
- Accessibility support

### React Implementation:
```tsx
import React, { useState, useMemo, useCallback, useRef } from 'react';

interface StockSymbol {
  id: string;
  symbol: string;
  name: string;
  price: number;
  change: number;
}

interface VirtualListProps {
  items: StockSymbol[];
  height: number;
  itemHeight: number;
}

const VirtualList: React.FC<VirtualListProps> = ({ items, height, itemHeight }) => {
  const [scrollTop, setScrollTop] = useState(0);
  const [searchTerm, setSearchTerm] = useState('');
  const containerRef = useRef<HTMLDivElement>(null);

  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.symbol.toLowerCase().includes(searchTerm.toLowerCase()) ||
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [items, searchTerm]);

  const totalHeight = filteredItems.length * itemHeight;
  const visibleItemCount = Math.ceil(height / itemHeight);
  const startIndex = Math.floor(scrollTop / itemHeight);
  const endIndex = Math.min(startIndex + visibleItemCount + 1, filteredItems.length);

  const visibleItems = filteredItems.slice(startIndex, endIndex);
  const offsetY = startIndex * itemHeight;

  const handleScroll = useCallback((e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.currentTarget.scrollTop);
  }, []);

  const handleKeyDown = useCallback((e: React.KeyboardEvent, index: number) => {
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      const nextIndex = Math.min(index + 1, filteredItems.length - 1);
      const nextElement = containerRef.current?.children[nextIndex] as HTMLElement;
      nextElement?.focus();
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      const prevIndex = Math.max(index - 1, 0);
      const prevElement = containerRef.current?.children[prevIndex] as HTMLElement;
      prevElement?.focus();
    }
  }, [filteredItems.length]);

  return (
    <div className="virtual-list-container">
      <input
        type="text"
        placeholder="Search symbols..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        className="search-input"
      />
      <div
        ref={containerRef}
        className="virtual-list"
        style={{ height, overflow: 'auto' }}
        onScroll={handleScroll}
      >
        <div style={{ height: totalHeight, position: 'relative' }}>
          <div style={{ transform: `translateY(${offsetY}px)` }}>
            {visibleItems.map((item, index) => (
              <div
                key={item.id}
                className="virtual-list-item"
                style={{ height: itemHeight }}
                tabIndex={0}
                onKeyDown={(e) => handleKeyDown(e, startIndex + index)}
              >
                <span className="symbol">{item.symbol}</span>
                <span className="name">{item.name}</span>
                <span className="price">${item.price.toFixed(2)}</span>
                <span className={`change ${item.change >= 0 ? 'positive' : 'negative'}`}>
                  {item.change >= 0 ? '+' : ''}{item.change.toFixed(2)}
                </span>
              </div>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
};

export default VirtualList;
```

## Challenge 3: Performance Optimization

### React Performance Hook:
```tsx
import { useEffect, useRef, useCallback } from 'react';

// Custom hook for performance monitoring
export const usePerformanceMonitor = (componentName: string) => {
  const renderCount = useRef(0);
  const lastRenderTime = useRef(performance.now());

  useEffect(() => {
    renderCount.current += 1;
    const currentTime = performance.now();
    const timeSinceLastRender = currentTime - lastRenderTime.current;
    
    console.log(`${componentName} rendered ${renderCount.current} times in ${timeSinceLastRender.toFixed(2)}ms`);
    
    lastRenderTime.current = currentTime;
  });

  const measureOperation = useCallback((operationName: string, operation: () => void) => {
    const start = performance.now();
    operation();
    const end = performance.now();
    console.log(`${operationName} took ${(end - start).toFixed(2)}ms`);
  }, []);

  return { measureOperation };
};

// Debounced hook for search
export const useDebounce = <T>(value: T, delay: number): T => {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
};
```

## Challenge 4: State Management Architecture

### Zustand Store for Trading App:
```tsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface Stock {
  symbol: string;
  price: number;
  change: number;
  changePercent: number;
}

interface Portfolio {
  symbol: string;
  shares: number;
  avgPrice: number;
}

interface TradingState {
  // State
  stocks: Record<string, Stock>;
  portfolio: Portfolio[];
  watchlist: string[];
  user: {
    id: string;
    balance: number;
    isAuthenticated: boolean;
  } | null;
  
  // Actions
  updateStock: (symbol: string, data: Partial<Stock>) => void;
  addToWatchlist: (symbol: string) => void;
  removeFromWatchlist: (symbol: string) => void;
  buyStock: (symbol: string, shares: number, price: number) => void;
  sellStock: (symbol: string, shares: number, price: number) => void;
  setUser: (user: TradingState['user']) => void;
  
  // Computed
  getPortfolioValue: () => number;
  getPortfolioChange: () => number;
}

export const useTradingStore = create<TradingState>()(
  devtools(
    persist(
      (set, get) => ({
        stocks: {},
        portfolio: [],
        watchlist: [],
        user: null,

        updateStock: (symbol, data) =>
          set((state) => ({
            stocks: {
              ...state.stocks,
              [symbol]: { ...state.stocks[symbol], ...data }
            }
          })),

        addToWatchlist: (symbol) =>
          set((state) => ({
            watchlist: [...new Set([...state.watchlist, symbol])]
          })),

        removeFromWatchlist: (symbol) =>
          set((state) => ({
            watchlist: state.watchlist.filter(s => s !== symbol)
          })),

        buyStock: (symbol, shares, price) =>
          set((state) => {
            const existingPosition = state.portfolio.find(p => p.symbol === symbol);
            const totalCost = shares * price;
            
            if (state.user && state.user.balance >= totalCost) {
              if (existingPosition) {
                // Update existing position
                const newShares = existingPosition.shares + shares;
                const newAvgPrice = ((existingPosition.shares * existingPosition.avgPrice) + totalCost) / newShares;
                
                return {
                  portfolio: state.portfolio.map(p =>
                    p.symbol === symbol
                      ? { ...p, shares: newShares, avgPrice: newAvgPrice }
                      : p
                  ),
                  user: {
                    ...state.user,
                    balance: state.user.balance - totalCost
                  }
                };
              } else {
                // Add new position
                return {
                  portfolio: [...state.portfolio, { symbol, shares, avgPrice: price }],
                  user: {
                    ...state.user,
                    balance: state.user.balance - totalCost
                  }
                };
              }
            }
            return state;
          }),

        sellStock: (symbol, shares, price) =>
          set((state) => {
            const position = state.portfolio.find(p => p.symbol === symbol);
            if (position && position.shares >= shares) {
              const totalValue = shares * price;
              const newShares = position.shares - shares;
              
              return {
                portfolio: newShares > 0
                  ? state.portfolio.map(p =>
                      p.symbol === symbol
                        ? { ...p, shares: newShares }
                        : p
                    )
                  : state.portfolio.filter(p => p.symbol !== symbol),
                user: state.user ? {
                  ...state.user,
                  balance: state.user.balance + totalValue
                } : null
              };
            }
            return state;
          }),

        setUser: (user) => set({ user }),

        getPortfolioValue: () => {
          const state = get();
          return state.portfolio.reduce((total, position) => {
            const stock = state.stocks[position.symbol];
            return total + (stock ? position.shares * stock.price : 0);
          }, 0);
        },

        getPortfolioChange: () => {
          const state = get();
          return state.portfolio.reduce((total, position) => {
            const stock = state.stocks[position.symbol];
            if (stock) {
              const currentValue = position.shares * stock.price;
              const costBasis = position.shares * position.avgPrice;
              return total + (currentValue - costBasis);
            }
            return total;
          }, 0);
        }
      }),
      {
        name: 'trading-storage',
        partialize: (state) => ({
          portfolio: state.portfolio,
          watchlist: state.watchlist,
          user: state.user
        })
      }
    )
  )
);
```

## Challenge 5: Error Boundary & Error Handling

```tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    
    // Send to error reporting service
    this.reportError(error, errorInfo);
    
    // Call custom error handler
    this.props.onError?.(error, errorInfo);
  }

  private reportError = (error: Error, errorInfo: ErrorInfo) => {
    // In a real app, send to Sentry, LogRocket, etc.
    fetch('/api/errors', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        error: error.message,
        stack: error.stack,
        componentStack: errorInfo.componentStack,
        timestamp: new Date().toISOString()
      })
    }).catch(console.error);
  };

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>We're sorry, but something unexpected happened.</p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
          <details>
            <summary>Error Details</summary>
            <pre>{this.state.error?.stack}</pre>
          </details>
        </div>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

## Practice Questions

1. **Implement a debounced search with API calls**
2. **Create a custom hook for WebSocket connections**
3. **Build a form validation system with TypeScript**
4. **Implement infinite scrolling with React Query**
5. **Create a performance monitoring system**
6. **Build a responsive trading chart component**
7. **Implement offline functionality with Service Workers**

## Key Points to Remember

- **Performance**: Always think about performance implications
- **Error Handling**: Implement comprehensive error boundaries
- **Accessibility**: Ensure keyboard navigation and screen reader support
- **TypeScript**: Use strict typing and advanced patterns
- **Testing**: Write unit tests for critical functionality
- **Documentation**: Comment complex logic and create README files
- **Security**: Validate inputs and handle sensitive data properly 