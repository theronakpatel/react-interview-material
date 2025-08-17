# Redux Guide - Simple & Easy

## What is Redux?

Redux is like a **central storage box** for your entire app. Instead of passing data through many components, you put everything in one place that all components can access.

**Think of it like a bank vault** - all your money (data) is in one secure place, and you can access it from anywhere with the right key (actions).

---

## Why Use Redux?

### ✅ **Use Redux when:**
- **Large app** with many components that need the same data
- **Complex state** that's hard to manage with just useState
- **Multiple components** need to update the same data
- **Need to track** what changed and when (for debugging)
- **Server state** that needs to be shared across components

### ❌ **Don't use Redux when:**
- **Small app** with simple state
- **Only one component** needs the data
- **Simple forms** or local UI state
- **Just want to avoid prop drilling** (use Context API instead)

---

## Redux Core Concepts

### 1. **Store** - The Central Storage
```tsx
// This is your main storage box
const store = createStore(reducer);
```

### 2. **State** - The Data Inside
```tsx
// This is what's stored in your box
const state = {
  user: { name: 'John', email: 'john@example.com' },
  cart: [{ id: 1, name: 'Product 1', price: 10 }],
  theme: 'dark'
};
```

### 3. **Actions** - Instructions to Change Data
```tsx
// These are like notes telling Redux what to do
const addToCart = { type: 'ADD_TO_CART', payload: { id: 2, name: 'Product 2', price: 20 } };
const changeTheme = { type: 'CHANGE_THEME', payload: 'light' };
```

### 4. **Reducer** - The Worker That Updates Data
```tsx
// This function decides how to update the data based on the action
function reducer(state, action) {
  switch (action.type) {
    case 'ADD_TO_CART':
      return { ...state, cart: [...state.cart, action.payload] };
    case 'CHANGE_THEME':
      return { ...state, theme: action.payload };
    default:
      return state;
  }
}
```

---

## Simple Example: Shopping Cart

### 1. **Create the Store**
```tsx
import { createStore } from 'redux';

// Initial state
const initialState = {
  cart: [],
  total: 0
};

// Reducer function
function cartReducer(state = initialState, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const newCart = [...state.cart, action.payload];
      const newTotal = newCart.reduce((sum, item) => sum + item.price, 0);
      return { cart: newCart, total: newTotal };
    
    case 'REMOVE_ITEM':
      const filteredCart = state.cart.filter(item => item.id !== action.payload);
      const updatedTotal = filteredCart.reduce((sum, item) => sum + item.price, 0);
      return { cart: filteredCart, total: updatedTotal };
    
    default:
      return state;
  }
}

// Create the store
const store = createStore(cartReducer);
```

### 2. **Create Actions**
```tsx
// Action creators (functions that create actions)
const addItem = (item) => ({
  type: 'ADD_ITEM',
  payload: item
});

const removeItem = (itemId) => ({
  type: 'REMOVE_ITEM',
  payload: itemId
});
```

### 3. **Use in Components**
```tsx
import { useSelector, useDispatch } from 'react-redux';

function Cart() {
  // Get data from Redux store
  const { cart, total } = useSelector(state => state);
  
  // Get function to send actions
  const dispatch = useDispatch();

  const handleAddItem = () => {
    const newItem = { id: Date.now(), name: 'New Product', price: 15 };
    dispatch(addItem(newItem));
  };

  const handleRemoveItem = (itemId) => {
    dispatch(removeItem(itemId));
  };

  return (
    <div>
      <h2>Shopping Cart</h2>
      <p>Total: ${total}</p>
      
      <button onClick={handleAddItem}>Add Item</button>
      
      {cart.map(item => (
        <div key={item.id}>
          <span>{item.name} - ${item.price}</span>
          <button onClick={() => handleRemoveItem(item.id)}>Remove</button>
        </div>
      ))}
    </div>
  );
}
```

---

## Modern Redux with Redux Toolkit (Recommended)

Redux Toolkit makes Redux much easier to use:

### 1. **Create Slice**
```tsx
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    total: 0
  },
  reducers: {
    addItem: (state, action) => {
      state.items.push(action.payload);
      state.total = state.items.reduce((sum, item) => sum + item.price, 0);
    },
    removeItem: (state, action) => {
      state.items = state.items.filter(item => item.id !== action.payload);
      state.total = state.items.reduce((sum, item) => sum + item.price, 0);
    }
  }
});

export const { addItem, removeItem } = cartSlice.actions;
export default cartSlice.reducer;
```

### 2. **Setup Store**
```tsx
import { configureStore } from '@reduxjs/toolkit';
import cartReducer from './cartSlice';

const store = configureStore({
  reducer: {
    cart: cartReducer
  }
});
```

### 3. **Use in Components**
```tsx
import { useSelector, useDispatch } from 'react-redux';
import { addItem, removeItem } from './cartSlice';

function Cart() {
  const { items, total } = useSelector(state => state.cart);
  const dispatch = useDispatch();

  return (
    <div>
      <h2>Cart Total: ${total}</h2>
      <button onClick={() => dispatch(addItem({ id: 1, name: 'Product', price: 10 }))}>
        Add Product
      </button>
      
      {items.map(item => (
        <div key={item.id}>
          {item.name} - ${item.price}
          <button onClick={() => dispatch(removeItem(item.id))}>Remove</button>
        </div>
      ))}
    </div>
  );
}
```

---

## Redux vs Other Options

| Situation | Use This |
|-----------|----------|
| **Simple local state** | `useState` |
| **Avoid prop drilling** | `Context API` |
| **Complex app state** | `Redux` |
| **Server state** | `React Query` or `Redux` |
| **Form state** | `React Hook Form` or `Formik` |

---

## When to Use Redux in Your App

### ✅ **Good Redux Use Cases:**
```tsx
// User authentication
const userState = {
  isLoggedIn: true,
  user: { name: 'John', email: 'john@example.com' },
  permissions: ['read', 'write']
};

// Shopping cart
const cartState = {
  items: [...],
  total: 100,
  shipping: 10
};

// App settings
const settingsState = {
  theme: 'dark',
  language: 'en',
  notifications: true
};

// Real-time data (like stock prices)
const stockState = {
  stocks: { AAPL: 150, GOOGL: 2800 },
  lastUpdated: '2024-01-01T10:00:00Z'
};
```

### ❌ **Bad Redux Use Cases:**
```tsx
// Local form state
const [name, setName] = useState('');
const [email, setEmail] = useState('');

// UI state (dropdown open/closed)
const [isDropdownOpen, setIsDropdownOpen] = useState(false);

// Component-specific data
const [localCount, setLocalCount] = useState(0);
```

---

## Quick Setup Checklist

1. **Install Redux Toolkit:**
   ```bash
   npm install @reduxjs/toolkit react-redux
   ```

2. **Create a slice** for each feature (cart, user, etc.)

3. **Configure store** with all your reducers

4. **Wrap your app** with Provider

5. **Use useSelector** to get data

6. **Use useDispatch** to send actions

---

## Summary

**Redux is like a central database for your app:**
- ✅ **Store** - Where data lives
- ✅ **Actions** - How to change data  
- ✅ **Reducers** - What happens when data changes
- ✅ **useSelector** - How components get data
- ✅ **useDispatch** - How components change data

**Use Redux when you have:**
- Complex state that many components need
- Need to track state changes
- Large app with lots of data sharing

**Don't use Redux for:**
- Simple local state
- Just to avoid prop drilling (use Context instead)
- Small apps with simple data needs 