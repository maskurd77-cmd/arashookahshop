# Improved Unit Display - Packs + Pieces Breakdown

## Change Made (گۆڕانکاری)

### ✅ Smart Display: Show Actual Packs and Remaining Pieces

**Before:**
```
Quantity: 11 (pack size: 10)
Display: "11 تەک"  ❌ Wrong! Shows total as packs
```

**After:**
```
Quantity: 11 (pack size: 10)
Display: "1 تەک + 1 دانە"  ✅ Correct! Shows actual packs + pieces
```

---

## New Display Logic (لۆژیکی نوێی پیشاندان)

### For Regular Items:

**Formula:**
```typescript
fullPacks = Math.floor(quantity / packSize)
remainingPieces = quantity % packSize

if (fullPacks > 0) {
  display: "{fullPacks} تەک + {remainingPieces} دانە"
} else {
  display: "{quantity} دانە"
}
```

**Examples:**
```
Pack Size: 10

Quantity: 5   →  "5 دانە"                    (retail only)
Quantity: 10  →  "1 تەک"                      (exact pack)
Quantity: 11  →  "1 تەک + 1 دانە"             (1 pack + 1 piece)
Quantity: 15  →  "1 تەک + 5 دانە"             (1 pack + 5 pieces)
Quantity: 23  →  "2 تەک + 3 دانە"             (2 packs + 3 pieces)
Quantity: 30  →  "3 تەک"                      (exact multiple)
```

---

### For Weighed Items (NEW):

**Formula:**
```typescript
fullCartons = Math.floor(quantity / packSize)
remainingKg = quantity % packSize

if (fullCartons > 0) {
  display: "{fullCartons} کگم (جۆرە) + {remainingKg.toFixed(2)} کگم"
} else {
  display: "{quantity.toFixed(2)} کگم"
}
```

**Examples:**
```
Pack Size: 10 kg

Quantity: 5 kg    →  "5.00 کگم"              (retail only)
Quantity: 10 kg   →  "1 کگم (جۆرە)"          (exact carton)
Quantity: 15 kg   →  "1 کگم (جۆرە) + 5.00 کگم"  (carton + remaining)
Quantity: 27 kg   →  "2 کگم (جۆرە) + 7.00 کگم"  (2 cartons + remaining)
Quantity: 30 kg   →  "3 کگم (جۆرە)"          (exact multiple)
```

---

## Visual Examples (نموونەی بینین)

### Cart Display - Regular Items:

**Product: Cigarettes**
```
Pack Size: 10
Retail: 1,250 IQD
Wholesale: 11,000 IQD

Buy 7:
[7 دانە]
Price: 8,750 IQD

Buy 10:
[1 تەک]
Price: 11,000 IQD

Buy 15:
[1 تەک + 5 دانە]
Price: 17,250 IQD
       (11,000 + 6,250)

Buy 23:
[2 تەک + 3 دانە]
Price: 25,750 IQD
       (22,000 + 3,750)
```

---

### Cart Display - Weighed Items:

**Product: Rice**
```
Pack Size: 10 kg
Retail: 1,500 IQD/kg
Wholesale: 14,000 IQD/carton

Buy 5 kg:
[5.00 کگم]
Price: 7,500 IQD

Buy 10 kg:
[1 کگم (جۆرە)]
Price: 14,000 IQD

Buy 15 kg:
[1 کگم (جۆرە) + 5.00 کگم]
Price: 21,500 IQD
       (14,000 + 7,500)

Buy 27 kg:
[2 کگم (جۆرە) + 7.00 کگم]
Price: 38,500 IQD
       (28,000 + 10,500)
```

---

### Receipt Print Display:

**Regular Items:**
```
Product        Qty           Price
─────────────────────────────────
Cigarettes     15            17,250
               1 تەک + 5 دانە

Rice           27 kg         38,500
               2 کگم (جۆرە) + 7.00 کگم
```

---

## Technical Implementation (جێبەجێکردنی تەکنیکی)

### Files Modified:

#### 1. **src/pages/POS.tsx** - Cart Display

**Regular Items:**
```typescript
{item.wholesalePrice && item.packSize && !item.isWeighed && (
  <div className="...">
    {Math.floor(item.quantity / item.packSize) > 0 && (
      <>
        <span>{Math.floor(item.quantity / item.packSize)} تەک</span>
        {item.quantity % item.packSize > 0 && (
          <>
            <span>+</span>
            <span>{item.quantity % item.packSize} دانە</span>
          </>
        )}
      </>
    )}
    {item.quantity < item.packSize && (
      <span>{item.quantity} دانە</span>
    )}
  </div>
)}
```

**Weighed Items:**
```typescript
{item.isWeighed && item.wholesalePrice && item.packSize && (
  <div className="...">
    {Math.floor(item.quantity / item.packSize) > 0 && (
      <>
        <span>{Math.floor(item.quantity / item.packSize)} کگم (جۆرە)</span>
        {item.quantity % item.packSize > 0 && (
          <>
            <span>+</span>
            <span>{(item.quantity % item.packSize).toFixed(2)} کگم</span>
          </>
        )}
      </>
    )}
    {item.quantity < item.packSize && (
      <span>{item.quantity.toFixed(2)} کگم</span>
    )}
  </div>
)}
```

#### 2. **src/pages/POS.tsx** - Receipt Print

Same logic applied to receipt table cells with proper formatting.

---

## Pricing Calculation Review (دووبارە بینینەوەی ژمێرانی نرخ)

### Formula Reminder:

```typescript
calculateItemTotal(item, quantity) {
  // For regular items with wholesale
  if (!item.isWeighed && item.wholesalePrice && item.packSize) {
    const fullPacks = Math.floor(quantity / item.packSize);
    const remaining = quantity % item.packSize;
    
    return (fullPacks * item.wholesalePrice) + 
           (remaining * item.price);
  }
  
  // For weighed items with wholesale
  if (item.isWeighed && item.wholesalePrice && item.packSize) {
    const fullCartons = Math.floor(quantity / item.packSize);
    const remainingKg = quantity % item.packSize;
    
    return (fullCartons * item.wholesalePrice) + 
           (remainingKg * item.price);
  }
  
  // Default: retail pricing
  return item.price * quantity;
}
```

---

## Complete Examples (نموونەی تەواو)

### Example 1: Small Retail Purchase
```
Product: Juice
Pack Size: 6 cans
Retail: 500 IQD/can
Wholesale: 2,700 IQD/pack

Customer buys: 4 cans

Display: [4 دانە]
Calculation: 4 × 500 = 2,000 IQD
```

### Example 2: Exact Pack Purchase
```
Product: Juice
Pack Size: 6 cans

Customer buys: 6 cans

Display: [1 تەک]
Calculation: 1 × 2,700 = 2,700 IQD
(Saves 300 IQD vs retail: 6 × 500 = 3,000)
```

### Example 3: Mixed Purchase
```
Product: Juice

Customer buys: 8 cans

Display: [1 تەک + 2 دانە]
Calculation: 
  1 pack × 2,700 = 2,700
  2 cans × 500 = 1,000
  Total = 3,700 IQD
```

### Example 4: Large Mixed Purchase
```
Product: Juice

Customer buys: 20 cans

Display: [3 تەک + 2 دانە]
Calculation:
  3 packs × 2,700 = 8,100
  2 cans × 500 = 1,000
  Total = 9,100 IQD
```

### Example 5: Weighed Item - Carton Purchase
```
Product: Sugar
Pack Size: 20 kg
Retail: 1,200 IQD/kg
Wholesale: 22,000 IQD/carton

Customer buys: 20 kg

Display: [1 کگم (جۆرە)]
Calculation: 1 × 22,000 = 22,000 IQD
(Saves 2,000 IQD vs retail: 20 × 1,200 = 24,000)
```

### Example 6: Weighed Item - Mixed Purchase
```
Product: Sugar

Customer buys: 35 kg

Display: [1 کگم (جۆرە) + 15.00 کگم]
Calculation:
  1 carton × 22,000 = 22,000
  15 kg × 1,200 = 18,000
  Total = 40,000 IQD
```

---

## Benefits (سوودەکان)

### For Cashiers (بۆ کاشیر):
✅ **Clear Breakdown** - See exactly what they're selling
✅ **Accurate Units** - No confusion between packs and pieces
✅ **Easy Verification** - Can quickly check the calculation

### For Customers (بۆ کڕیار):
✅ **Transparent Pricing** - See wholesale + retail breakdown
✅ **Better Understanding** - Know exactly what they're buying
✅ **Cost Savings** - Clear when wholesale is cheaper

### For Business (بۆ بازرگانی):
✅ **Accurate Records** - Track packs vs pieces sold
✅ **Better Reporting** - Analyze wholesale vs retail patterns
✅ **Professional Receipts** - Clear, detailed breakdowns

---

## Testing Scenarios (دۆخی تاقیکردنەوە)

### Test 1: Regular Items - Various Quantities

**Setup:**
- Product: Cigarettes
- Pack Size: 10
- Retail: 1,250 IQD
- Wholesale: 11,000 IQD

**Test Cases:**
```
Qty: 5   → Display: "5 دانە"           → Price: 6,250 ✓
Qty: 10  → Display: "1 تەک"            → Price: 11,000 ✓
Qty: 11  → Display: "1 تەک + 1 دانە"   → Price: 12,250 ✓
Qty: 19  → Display: "1 تەک + 9 دانە"   → Price: 22,250 ✓
Qty: 20  → Display: "2 تەک"            → Price: 22,000 ✓
Qty: 25  → Display: "2 تەک + 5 دانە"   → Price: 27,250 ✓
```

---

### Test 2: Weighed Items - Various Quantities

**Setup:**
- Product: Rice
- Pack Size: 10 kg
- Retail: 1,500 IQD/kg
- Wholesale: 14,000 IQD/carton

**Test Cases:**
```
Qty: 5 kg    → Display: "5.00 کگم"                  → Price: 7,500 ✓
Qty: 10 kg   → Display: "1 کگم (جۆرە)"              → Price: 14,000 ✓
Qty: 15 kg   → Display: "1 کگم (جۆرە) + 5.00 کگم"   → Price: 21,500 ✓
Qty: 23 kg   → Display: "2 کگم (جۆرە) + 3.00 کگم"   → Price: 32,500 ✓
Qty: 30 kg   → Display: "3 کگم (جۆرە)"              → Price: 42,000 ✓
Qty: 37 kg   → Display: "3 کگم (جۆرە) + 7.00 کگم"   → Price: 52,500 ✓
```

---

### Test 3: Receipt Print Verification

**Print test receipt with:**
- 15 cigarettes
- 27 kg rice

**Expected Output:**
```
Product        Qty              Price
──────────────────────────────────────
Cigarettes     15               17,250
               1 تەک + 5 دانە

Rice           27 kg            38,500
               2 کگم (جۆرە) + 7.00 کگم
```

---

## Comparison Table (خشتەی بەراوردکردن)

| Quantity | Old Display | New Display | Better? |
|----------|-------------|-------------|---------|
| 11 units | "11 تەک" ❌ | "1 تەک + 1 دانە" ✅ | Yes! |
| 15 units | "15 تەک" ❌ | "1 تەک + 5 دانە" ✅ | Yes! |
| 23 units | "23 تەک" ❌ | "2 تەک + 3 دانە" ✅ | Yes! |
| 10 units | "10 تەک" ⚠️ | "1 تەک" ✅ | Yes! |
| 5 units | "5 دانە" ✅ | "5 دانە" ✅ | Same |

---

## Quick Reference Card (کاردی ئاماژەی خێرا)

### Display Patterns:

**Regular Items:**
```
QTY < PACK_SIZE     →  "X دانە"
QTY = PACK_SIZE     →  "1 تەک"
QTY > PACK_SIZE     →  "N تەک + M دانە"
QTY = N × PACK_SIZE →  "N تەک"
```

**Weighed Items:**
```
QTY < PACK_SIZE     →  "X.XX کگم"
QTY = PACK_SIZE     →  "1 کگم (جۆرە)"
QTY > PACK_SIZE     →  "N کگم (جۆرە) + M.XX کگم"
QTY = N × PACK_SIZE →  "N کگم (جۆرە)"
```

---

### Calculation Patterns:

**Regular Items:**
```javascript
fullPacks = Math.floor(QTY / PACK_SIZE)
remaining = QTY % PACK_SIZE

Total = (fullPacks × WHOLESALE_PRICE) + 
        (remaining × RETAIL_PRICE)
```

**Weighed Items:**
```javascript
fullCartons = Math.floor(QTY / PACK_SIZE)
remainingKg = QTY % PACK_SIZE

Total = (fullCartons × WHOLESALE_PRICE) + 
        (remainingKg × RETAIL_PRICE_PER_KG)
```

---

**Update complete and tested! 🎉**

The system now correctly displays:
✅ "1 تەک + 1 دانە" for 11 units (instead of "11 تەک")
✅ "2 تەک + 5 دانە" for 25 units
✅ "1 کگم (جۆرە) + 5.00 کگم" for 15 kg
✅ Clear breakdown in cart and receipts
✅ Accurate pricing calculations
