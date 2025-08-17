# Code Splitting with React.lazy and Suspense

## What is Code Splitting?

**Code splitting** is like breaking a big book into smaller chapters. Instead of loading your entire app at once, you load only the parts that users actually need.

**Benefits:**
- 🚀 **Faster initial load** - Users see your app quicker
- 📱 **Better mobile performance** - Less data usage
- 💾 **Reduced memory usage** - Only load what's needed
- 🎯 **Better user experience** - App feels more responsive

---

## How React.lazy Works

**React.lazy** is like a smart loader that:
1. **Waits** until a component is actually needed
2. **Downloads** only that component's code
3. **Caches** it for future use

---

## 1. Basic Code Splitting

### ❌ **Without Code Splitting (Bad Performance)**
```tsx
import HeavyDashboard from './HeavyDashboard';
import ComplexChart from './ComplexChart';
import DataTable from './DataTable';
import UserProfile from './UserProfile';
import Settings from './Settings';

function App() {
  return (
    <div>
      {/* All these heavy components load immediately, even if user never visits them */}
      <HeavyDashboard />
      <ComplexChart />
      <DataTable />
      <UserProfile />
      <Settings />
    </div>
  );
}
```

### ✅ **With Code Splitting (Good Performance)**
```tsx
import { lazy, Suspense } from 'react';

// Lazy load components - they only load when actually used
const HeavyDashboard = lazy(() => import('./HeavyDashboard'));
const ComplexChart = lazy(() => import('./ComplexChart'));
const DataTable = lazy(() => import('./DataTable'));
const UserProfile = lazy(() => import('./UserProfile'));
const Settings = lazy(() => import('./Settings'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyDashboard />
        <ComplexChart />
        <DataTable />
        <UserProfile />
        <Settings />
      </Suspense>
    </div>
  );
}
```

---

## 2. Route-Based Code Splitting

**Most common use case**: Split code by different pages/routes.

### ❌ **Without Route Splitting**
```tsx
import Dashboard from './pages/Dashboard';
import Profile from './pages/Profile';
import Settings from './pages/Settings';
import Trading from './pages/Trading';

function App() {
  const [currentPage, setCurrentPage] = useState('dashboard');

  return (
    <div>
      <nav>
        <button onClick={() => setCurrentPage('dashboard')}>Dashboard</button>
        <button onClick={() => setCurrentPage('profile')}>Profile</button>
        <button onClick={() => setCurrentPage('settings')}>Settings</button>
        <button onClick={() => setCurrentPage('trading')}>Trading</button>
      </nav>

      {/* All pages load immediately */}
      {currentPage === 'dashboard' && <Dashboard />}
      {currentPage === 'profile' && <Profile />}
      {currentPage === 'settings' && <Settings />}
      {currentPage === 'trading' && <Trading />}
    </div>
  );
}
```

### ✅ **With Route Splitting**
```tsx
import { lazy, Suspense, useState } from 'react';

// Lazy load each page
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));
const Trading = lazy(() => import('./pages/Trading'));

function App() {
  const [currentPage, setCurrentPage] = useState('dashboard');

  // Loading component
  const LoadingSpinner = () => (
    <div className="loading">
      <div className="spinner"></div>
      <p>Loading {currentPage}...</p>
    </div>
  );

  return (
    <div>
      <nav>
        <button onClick={() => setCurrentPage('dashboard')}>Dashboard</button>
        <button onClick={() => setCurrentPage('profile')}>Profile</button>
        <button onClick={() => setCurrentPage('settings')}>Settings</button>
        <button onClick={() => setCurrentPage('trading')}>Trading</button>
      </nav>

      <Suspense fallback={<LoadingSpinner />}>
        {/* Only the current page loads */}
        {currentPage === 'dashboard' && <Dashboard />}
        {currentPage === 'profile' && <Profile />}
        {currentPage === 'settings' && <Settings />}
        {currentPage === 'trading' && <Trading />}
      </Suspense>
    </div>
  );
}
```

---

## 3. Conditional Code Splitting

**Load components based on user actions or conditions.**

### **Example: Load Heavy Chart Only When User Clicks**
```tsx
import { lazy, Suspense, useState } from 'react';

// Lazy load the heavy chart component
const HeavyChart = lazy(() => import('./HeavyChart'));

function TradingDashboard() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <h2>Trading Dashboard</h2>
      
      <button onClick={() => setShowChart(!showChart)}>
        {showChart ? 'Hide' : 'Show'} Advanced Chart
      </button>

      {/* Chart only loads when user clicks the button */}
      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

### **Example: Load Features Based on User Permissions**
```tsx
import { lazy, Suspense } from 'react';

const AdminPanel = lazy(() => import('./AdminPanel'));
const PremiumFeatures = lazy(() => import('./PremiumFeatures'));

function App({ user }) {
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      
      {/* Admin features only load for admin users */}
      {user.isAdmin && (
        <Suspense fallback={<div>Loading admin panel...</div>}>
          <AdminPanel />
        </Suspense>
      )}

      {/* Premium features only load for premium users */}
      {user.isPremium && (
        <Suspense fallback={<div>Loading premium features...</div>}>
          <PremiumFeatures />
        </Suspense>
      )}
    </div>
  );
}
```

---

## 4. Component-Level Splitting

**Split individual components that are heavy or rarely used.**

### **Example: Modal Components**
```tsx
import { lazy, Suspense, useState } from 'react';

// Lazy load modal components
const LoginModal = lazy(() => import('./modals/LoginModal'));
const SettingsModal = lazy(() => import('./modals/SettingsModal'));
const HelpModal = lazy(() => import('./modals/HelpModal'));

function Header() {
  const [activeModal, setActiveModal] = useState(null);

  return (
    <header>
      <button onClick={() => setActiveModal('login')}>Login</button>
      <button onClick={() => setActiveModal('settings')}>Settings</button>
      <button onClick={() => setActiveModal('help')}>Help</button>

      {/* Modals only load when opened */}
      <Suspense fallback={<div>Loading modal...</div>}>
        {activeModal === 'login' && (
          <LoginModal onClose={() => setActiveModal(null)} />
        )}
        {activeModal === 'settings' && (
          <SettingsModal onClose={() => setActiveModal(null)} />
        )}
        {activeModal === 'help' && (
          <HelpModal onClose={() => setActiveModal(null)} />
        )}
      </Suspense>
    </header>
  );
}
```

---

## 5. Library Splitting

**Split third-party libraries that are heavy.**

### **Example: Chart Libraries**
```tsx
import { lazy, Suspense, useState } from 'react';

// Lazy load different chart types
const LineChart = lazy(() => import('chart-library/LineChart'));
const BarChart = lazy(() => import('chart-library/BarChart'));
const PieChart = lazy(() => import('chart-library/PieChart'));

function ChartSelector() {
  const [chartType, setChartType] = useState('line');

  return (
    <div>
      <select value={chartType} onChange={(e) => setChartType(e.target.value)}>
        <option value="line">Line Chart</option>
        <option value="bar">Bar Chart</option>
        <option value="pie">Pie Chart</option>
      </select>

      <Suspense fallback={<div>Loading chart...</div>}>
        {chartType === 'line' && <LineChart data={data} />}
        {chartType === 'bar' && <BarChart data={data} />}
        {chartType === 'pie' && <PieChart data={data} />}
      </Suspense>
    </div>
  );
}
```

---

## 6. Advanced Patterns

### **Preloading Components**
```tsx
import { lazy, Suspense, useEffect } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  const [showHeavy, setShowHeavy] = useState(false);

  // Preload the component when user hovers over button
  const preloadHeavyComponent = () => {
    import('./HeavyComponent');
  };

  return (
    <div>
      <button 
        onMouseEnter={preloadHeavyComponent}
        onClick={() => setShowHeavy(true)}
      >
        Load Heavy Component
      </button>

      {showHeavy && (
        <Suspense fallback={<div>Loading...</div>}>
          <HeavyComponent />
        </Suspense>
      )}
    </div>
  );
}
```

### **Error Boundaries with Lazy Loading**
```tsx
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Error boundary for lazy-loaded components
class LazyErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Lazy component failed to load:', error);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h3>Something went wrong loading this component.</h3>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

function App() {
  return (
    <LazyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </LazyErrorBoundary>
  );
}
```

---

## 7. Real-World Trading App Example

```tsx
import { lazy, Suspense, useState } from 'react';

// Lazy load different trading features
const StockChart = lazy(() => import('./components/StockChart'));
const OrderBook = lazy(() => import('./components/OrderBook'));
const TradeHistory = lazy(() => import('./components/TradeHistory'));
const Portfolio = lazy(() => import('./components/Portfolio'));
const NewsFeed = lazy(() => import('./components/NewsFeed'));

function TradingApp() {
  const [activeTab, setActiveTab] = useState('chart');

  // Custom loading component
  const LoadingComponent = () => (
    <div className="loading-container">
      <div className="spinner"></div>
      <p>Loading trading data...</p>
    </div>
  );

  return (
    <div className="trading-app">
      <nav className="trading-nav">
        <button 
          className={activeTab === 'chart' ? 'active' : ''}
          onClick={() => setActiveTab('chart')}
        >
          Chart
        </button>
        <button 
          className={activeTab === 'orderbook' ? 'active' : ''}
          onClick={() => setActiveTab('orderbook')}
        >
          Order Book
        </button>
        <button 
          className={activeTab === 'history' ? 'active' : ''}
          onClick={() => setActiveTab('history')}
        >
          History
        </button>
        <button 
          className={activeTab === 'portfolio' ? 'active' : ''}
          onClick={() => setActiveTab('portfolio')}
        >
          Portfolio
        </button>
        <button 
          className={activeTab === 'news' ? 'active' : ''}
          onClick={() => setActiveTab('news')}
        >
          News
        </button>
      </nav>

      <main className="trading-content">
        <Suspense fallback={<LoadingComponent />}>
          {activeTab === 'chart' && <StockChart />}
          {activeTab === 'orderbook' && <OrderBook />}
          {activeTab === 'history' && <TradeHistory />}
          {activeTab === 'portfolio' && <Portfolio />}
          {activeTab === 'news' && <NewsFeed />}
        </Suspense>
      </main>
    </div>
  );
}
```

---

## 8. Best Practices

### ✅ **Do's:**
- **Split by routes** - Each page gets its own chunk
- **Split heavy libraries** - Chart libraries, UI libraries
- **Use meaningful fallbacks** - Show loading states that match the component
- **Preload on hover** - Start loading before user clicks
- **Handle errors** - Use error boundaries for lazy components

### ❌ **Don'ts:**
- **Don't over-split** - Too many small chunks can hurt performance
- **Don't split everything** - Keep frequently used components in main bundle
- **Don't forget fallbacks** - Always provide loading states
- **Don't ignore errors** - Handle loading failures gracefully

---

## 9. Performance Monitoring

### **Check Bundle Size**
```bash
# Install bundle analyzer
npm install --save-dev webpack-bundle-analyzer

# Analyze your bundle
npm run build
npx webpack-bundle-analyzer build/static/js/*.js
```

### **Monitor Loading Performance**
```tsx
// Track when lazy components load
const HeavyComponent = lazy(() => 
  import('./HeavyComponent').then(module => {
    console.log('HeavyComponent loaded successfully');
    return module;
  })
);
```

---

## 10. Checklist for Code Splitting

### ✅ **Before Splitting:**
- [ ] Identify heavy components (use bundle analyzer)
- [ ] Find rarely-used features
- [ ] Check user behavior patterns
- [ ] Measure current bundle size

### ✅ **During Splitting:**
- [ ] Split by routes/pages
- [ ] Split heavy third-party libraries
- [ ] Add meaningful loading states
- [ ] Implement error boundaries
- [ ] Test loading performance

### ✅ **After Splitting:**
- [ ] Measure bundle size reduction
- [ ] Test loading times
- [ ] Verify error handling
- [ ] Check mobile performance
- [ ] Monitor user experience

---

## Summary

**Code splitting makes your app faster by:**
- 📦 **Loading only what's needed** - Reduce initial bundle size
- 🚀 **Faster startup** - Users see your app quicker
- 📱 **Better mobile experience** - Less data usage
- 🎯 **Improved performance** - Smoother interactions

**Key tools:**
- **React.lazy()** - Lazy load components
- **Suspense** - Show loading states
- **Error Boundaries** - Handle loading failures
- **Bundle Analyzer** - Monitor bundle size

**Remember**: Code splitting is about finding the right balance between performance and user experience! 