# 04 — Implementation: The Multi-Step Checkout Flow 💳

> **Core Concepts: `useReducer`, `useEffect`, and Custom Hooks.**

In this section, we build the **Multi-Step Checkout**. This is the most complex part of our e-commerce app, as it requires managing several related pieces of data (Address, Payment, Coupons) and communicating with an external "Payment API".

---

## 1. `useReducer` — Complex State Logic

### 🧠 Detailed Concept
**`useReducer`** is like a "Manager" for your state. Instead of many tiny `useState` variables, you use one `useReducer` to manage an object of related values. 

It follows the **Action → Reducer → State** pattern:
1. **Action**: "I want to set the shipping address."
2. **Reducer**: "Okay, I'll update the `shippingInfo` field and leave the others alone."
3. **State**: The updated checkout information.

### 🛒 Real-World Implementation
In ShopSmart, the user fills out their address and credit card details.

```jsx
import React, { useReducer } from 'react';

// 1. Define the Initial State.
const initialCheckoutState = {
  step: 1,
  address: '',
  paymentMethod: 'card', 
  isProcessing: false,
};

// 2. Define the Reducer function.
function checkoutReducer(state, action) {
  switch (action.type) {
    case 'SET_STEP':
      return { ...state, step: action.payload };
    case 'SET_ADDRESS':
      return { ...state, address: action.payload };
    case 'START_PAYMENT':
      return { ...state, isProcessing: true };
    case 'FINISH_PAYMENT':
      return { ...state, isProcessing: false, step: 3 }; // Redirect to Success step
    default:
      return state;
  }
}

function CheckoutFlow() {
  const [state, dispatch] = useReducer(checkoutReducer, initialCheckoutState);

  const handleNext = () => {
    dispatch({ type: 'SET_STEP', payload: state.step + 1 });
  };

  return (
    <div className="checkout-flow">
      <h2>Step {state.step} of 3</h2>
      {state.step === 1 && (
        <AddressForm 
          onSave={(addr) => dispatch({ type: 'SET_ADDRESS', payload: addr })} 
        />
      )}
      <button onClick={handleNext}>Next</button>
    </div>
  );
}
```

---

## 2. `useEffect` — Side Effects (Payment Simulation)

### 🧠 Detailed Concept
**`useEffect`** handles "Side Effects"—things that happen **outside** of React's view logic, like fetching data from an API, interacting with the DOM directly, or starting a timer.

### 🛒 Real-World Implementation
When the user clicks "Pay", we trigger a simulated payment API call.

```jsx
import { useEffect } from 'react';

function PaymentLoader({ isProcessing, onComplete }) {
  useEffect(() => {
    // 1. If we are not processing, do nothing.
    if (!isProcessing) return;

    // 2. Simulate a payment API call that takes 2 seconds.
    console.log("Processing payment via Stripe/PayPal...");
    const timer = setTimeout(() => {
      onComplete();
    }, 2000);

    // 3. Cleanup function: If the user leaves the page, cancel the call.
    return () => clearTimeout(timer);
  }, [isProcessing, onComplete]);

  return isProcessing ? <p>Processing Payment... Do not refresh.</p> : null;
}
```

---

## 3. Custom Hooks — Reusable Business Logic

### 🧠 Detailed Concept
**Custom Hooks** are functions that allow you to extract hook logic (like `useState` or `useEffect`) from a component so it can be **reused** elsewhere.

### 🛒 Real-World Implementation
We create a `usePaymentStatus` hook to track whether a transaction is successful.

```jsx
function usePaymentStatus() {
  const [status, setStatus] = useState('pending'); // pending, success, error

  const verifyPayment = async (txnId) => {
    const response = await fetch(`/api/verify/${txnId}`);
    const data = await response.json();
    setStatus(data.status);
  };

  return { status, verifyPayment };
}

// Usage in any component:
const { status, verifyPayment } = usePaymentStatus();
```

---

## 🏗️ The Full Flow Recap
1. **User enters** their shipping info.
2. The `checkoutReducer` updates the `address` (**useReducer**).
3. **User clicks "Pay"**. The `isProcessing` flag is set to `true`.
4. The `useEffect` sees the flag change and **triggers the payment timer** (**useEffect**).
5. After 2 seconds, the timer runs `onComplete` (**dispatch**), moving the user to the "Success" step.
6. The `usePaymentStatus` hook can now be used on the "Success" page to **verify the final transaction** (**Custom Hook**).
