# 📡 Offline-First POS System - Complete Guide

## Overview (تێبینی گشتی)

This system implements a complete **Offline-First** architecture for your POS application. Orders can be saved even when there's no internet connection, and automatically sync to Firebase when connectivity is restored.

ئەم سیستەمە پەیڕەوی لە شێوازی **Offline-First** دەکات بۆ ئەپڵیکەیشنی POS. دەتوانرێت فرۆشتنەکان پاشەکەوت بکرێن تەنانەت کاتێک ئینتەرنێت نییە، و بە خۆکارایی هاوکات دەکرێن لەگەڵ Firebase کاتێک ئینتەرنێت دەبێتەوە.

---

## 🎯 Key Features (تایبەتمەندییە سەرەکییەکان)

### 1. **Automatic Online/Offline Detection**
- Detects when internet connection is lost or restored
- Shows visual indicators in the UI
- Automatically switches between online and offline modes

### 2. **Dual Storage Strategy**
- **Online Mode**: Saves directly to Firestore (Firebase)
- **Offline Mode**: Saves to IndexedDB using `idb` library
- **Fallback Mode**: If Firestore fails, saves to IndexedDB automatically

### 3. **Auto-Sync on Reconnection**
- When internet returns, all pending orders sync automatically
- Updates inventory in Firestore
- Removes synced orders from IndexedDB
- Shows sync progress in UI

### 4. **Manual Sync Control**
- Button to manually trigger sync
- Shows pending order count
- Displays last sync time

---

## 📁 File Structure (پێکهاتی فایلەکان)

```
src/
├── services/
│   ├── offlineService.ts      # IndexedDB operations (save, get, sync orders)
│   └── syncService.ts         # Online/offline detection & sync logic
├── hooks/
│   └── useOfflineSync.ts      # React hook for sync state management
├── components/
│   └── OfflineIndicator.tsx   # UI component showing online status
└── pages/
    └── POS.tsx                # Updated with offline-first checkout
```

---

## 🔧 How It Works (چۆن کار دەکات)

### **Online Mode (کاتی هەبوونی ئینتەرنێت)**

```javascript
// 1. User completes checkout
if (isOnline) {
  // Save to Firestore
  await addDoc(collection(db, 'sales'), orderData);
  
  // Update inventory
  await updateDoc(productRef, { stock: increment(-quantity) });
  
  // Handle debt records if needed
}
```

### **Offline Mode (کاتی نەبوونی ئینتەرنێت)**

```javascript
// 1. User completes checkout (no internet)
else {
  // Save to IndexedDB instead
  await saveOrderOffline(orderData);
  
  // Show success message
  alert('✅ Order saved locally (will sync when online)');
}
```

### **Reconnection & Auto-Sync (کاتی گەڕانەوەی ئینتەرنێت)**

```javascript
// Automatic process when coming back online:
window.addEventListener('online', async () => {
  // 1. Get all unsynced orders from IndexedDB
  const orders = await getUnsyncedOrders();
  
  // 2. Sync each order to Firestore
  for (const order of orders) {
    await addDoc(collection(db, 'sales'), order);
    await markOrderAsSynced(order.id);
    await deleteSyncedOrder(order.id);
  }
  
  // 3. Update UI
  console.log(`Synced ${orders.length} orders!`);
});
```

---

## 💻 Implementation Details (وردەکاری جێبەجێکردن)

### **1. Database Schema (IndexedDB)**

Each offline order contains:
```typescript
{
  id: string;                    // Unique ID (offline-{timestamp}-{random})
  items: CartItem[];             // Products in the order
  subtotal: number;              // Before discount
  discount: number;              // Discount amount
  total: number;                 // Final total
  paymentMethod: string;         // 'cash', 'debt', etc.
  amountPaid: number;            // Amount paid by customer
  customerId?: string;           // For debt payments
  section: string;               // 'general' or 'shisha'
  createdAt: number;             // Timestamp
  synced: boolean;               // Sync status
  syncAttempts: number;          // Number of sync attempts
  isOffline: boolean;            // Flag for offline origin
}
```

### **2. Service Functions**

#### **offlineService.ts** - Core Functions:

| Function | Description |
|----------|-------------|
| `initDB()` | Initialize IndexedDB database |
| `saveOrderOffline()` | Save order when offline |
| `getUnsyncedOrders()` | Get all pending orders |
| `markOrderAsSynced()` | Mark order as synced |
| `deleteSyncedOrder()` | Remove synced order from IndexedDB |
| `getOfflineStats()` | Get sync statistics |

#### **syncService.ts** - Sync Logic:

| Function | Description |
|----------|-------------|
| `setupOnlineStatusListener()` | Listen to online/offline events |
| `syncPendingOrders()` | Sync all pending orders to Firestore |
| `setupAutoSync()` | Auto-sync when connection restored |
| `manualSync()` | Manual sync trigger |

### **3. React Hooks**

#### **useOfflineSync.ts** - Usage Example:

```typescript
import { useOfflineSync } from '../hooks/useOfflineSync';

function MyComponent() {
  const { 
    isOnline,      // Current online status
    syncing,       // Sync in progress?
    pendingCount,  // Orders waiting to sync
    sync,          // Manual sync function
    lastSyncTime   // Last successful sync
  } = useOfflineSync();
  
  return (
    <div>
      {isOnline ? '🟢 Online' : '🔴 Offline'}
      {pendingCount > 0 && (
        <button onClick={sync}>
          Sync {pendingCount} orders
        </button>
      )}
    </div>
  );
}
```

---

## 🎨 UI Components (پێکهاتەکانی ڕووکار)

### **OfflineIndicator Component**

Shows in bottom-left corner with:

1. **Online Status Badge**
   - Green: "ئینتەرنێت هەیە" (Online)
   - Red: "ئینتەرنێت نییە" (Offline)

2. **Pending Orders Alert** (when offline with pending orders)
   - Shows count of unsynced orders
   - Orange warning badge

3. **Sync Button** (when online with pending orders)
   - Blue button with sync icon
   - Shows count: "هاوکاتکردنی (5)"

4. **Last Sync Time**
   - Shows when last sync completed
   - Format: "دواین هاوکاتکردن: 14:30"

---

## 🚀 Setup Instructions (دەستپێکردن)

### **1. Install Dependencies**

```bash
npm install idb
```

### **2. Import Services in POS.tsx**

```typescript
import { useOfflineSync } from '../hooks/useOfflineSync';
import { saveOrderOffline } from '../services/offlineService';
import { OfflineIndicator, MiniOnlineStatus } from '../components/OfflineIndicator';
```

### **3. Use Hook in Component**

```typescript
export default function POS() {
  const { isOnline, syncing, pendingCount, sync, lastSyncTime } = useOfflineSync();
  
  // ... rest of your code
}
```

### **4. Update Checkout Logic**

```typescript
const handleCheckout = async (shouldPrint: boolean = true) => {
  // ... validation code
  
  const orderData = {
    items: cart,
    subtotal,
    discount,
    total,
    // ... other fields
    isOffline: !isOnline,
  };
  
  if (isOnline) {
    // Save to Firestore
    await addDoc(collection(db, 'sales'), orderData);
    // ... update inventory
  } else {
    // Save to IndexedDB
    await saveOrderOffline(orderData);
    alert('✅ فرۆشتنەکە پاشەکەوتکرا (هاوکات دەکرێت کاتێک ئینتەرنێت هەبێت)');
  }
};
```

### **5. Add Indicator to UI**

```typescript
return (
  <>
    <OfflineIndicator
      isOnline={isOnline}
      syncing={syncing}
      pendingCount={pendingCount}
      onSync={sync}
      lastSyncTime={lastSyncTime}
    />
    
    {/* Rest of your UI */}
  </>
);
```

---

## 📊 Testing Scenarios (دۆخی تاقیکردنەوە)

### **Scenario 1: Normal Online Purchase**
1. ✅ Ensure you're online (green indicator)
2. Add products to cart
3. Complete checkout
4. Verify order appears in Firestore immediately

### **Scenario 2: Offline Purchase**
1. Disconnect internet (turn off WiFi)
2. Red indicator appears: "ئینتەرنێت نییە"
3. Add products to cart
4. Complete checkout
5. See success message: "پاشەکەوتکرا بۆ کاتێک ئینتەرنێت هەبێت"
6. Orange badge shows pending orders count

### **Scenario 3: Reconnection & Auto-Sync**
1. Reconnect internet
2. Green indicator appears: "ئینتەرنێت هەیە"
3. Auto-sync starts after 2 seconds
4. Blue sync button appears with count
5. Click sync button (or wait for auto-sync)
6. Watch counter decrease as orders sync
7. Success message when complete

### **Scenario 4: Fallback Mode**
1. Be online but simulate Firestore error
2. System automatically saves to IndexedDB
3. Shows warning message
4. Will sync later when issue resolved

---

## 🔍 Debugging & Monitoring (چاودێری و هەڵەدۆزینەوە)

### **Check Console Logs**

The system logs all actions:
```javascript
// Online mode
🟢 Online - saving to Firestore
✅ Order synced to Firestore: abc123

// Offline mode
🔴 Offline - saving to IndexedDB
✅ Order saved offline: offline-1234567890-xyz

// Sync process
🔄 Starting sync of 5 orders...
✅ Synced order offline-123 (1/5)
🎉 Sync complete: 5 succeeded, 0 failed
```

### **Check IndexedDB Manually**

Open DevTools → Application → IndexedDB → pos-offline-orders

You'll see:
- All pending orders
- Their sync status
- Timestamps and attempt counts

### **Get Stats Programmatically**

```typescript
import { getOfflineStats } from '../services/offlineService';

const stats = await getOfflineStats();
console.log(stats);
// { total: 10, unsynced: 3, synced: 7, pendingSync: 3 }
```

---

## ⚙️ Advanced Configuration (ڕێکخستنی پێشکەوتوو)

### **Customize Sync Delay**

In `syncService.ts`, adjust auto-sync delay:

```typescript
setTimeout(async () => {
  const status = await syncPendingOrders();
  if (onSyncComplete) {
    onSyncComplete(status);
  }
}, 2000); // Change 2000 to desired milliseconds
```

### **Retry Logic**

Orders that fail to sync get retry attempts tracked:

```typescript
// In syncService.ts - syncOrderToFirestore
if (!success) {
  await incrementSyncAttempts(order.id);
  // After 5 attempts, you could implement exponential backoff
}
```

### **Custom Error Handling**

Add custom error handlers in `handleCheckout`:

```typescript
catch (error: any) {
  if (error.code === 'permission-denied') {
    alert('هەڵەی مۆڵەت - تکایە پەیوەندی بە پشتگیریکردن بکە');
  } else if (error.code === 'unavailable') {
    alert('Firestore بەردەست نییە - پاشەکەوتکردن لە ناوخۆدا');
    await saveOrderOffline(orderData);
  }
}
```

---

## 📝 Best Practices (باشترین ڕێکارەکان)

### ✅ DO:
- Always check `isOnline` before attempting Firestore operations
- Show clear feedback to users about sync status
- Test thoroughly in offline mode
- Monitor console logs during development
- Clear old synced orders periodically

### ❌ DON'T:
- Don't rely solely on `navigator.onLine` (can be unreliable)
- Don't block UI during sync operations
- Don't delete IndexedDB data without confirmation
- Don't ignore failed sync attempts
- Don't sync too frequently (battery/data usage)

---

## 🎓 Learning Resources (سەرچاوەی فێربوون)

- [IndexedDB Documentation](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [idb Library](https://github.com/jakearchibald/idb)
- [Firebase Offline Support](https://firebase.google.com/docs/firestore/manage-data/enable-offline)
- [PWA Offline Cookbook](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook)

---

## 🆘 Troubleshooting (چارەسەرکردنی کێشەکان)

### **Problem: Orders not syncing**
**Solution:** Check console for errors, verify Firebase rules allow writes

### **Problem: IndexedDB full**
**Solution:** Clear synced orders: `await clearAllOrders()`

### **Problem: Sync happening too slowly**
**Solution:** Reduce delay in `setupAutoSync()` or increase batch size

### **Problem: UI not updating**
**Solution:** Ensure `useOfflineSync` hook is properly imported and called

---

## 📞 Support (پشتگیری)

For issues or questions:
1. Check console logs first
2. Review this guide
3. Test in different network conditions
4. Verify all files are properly imported

---

**Built with ❤️ for reliable offline-first POS systems**
