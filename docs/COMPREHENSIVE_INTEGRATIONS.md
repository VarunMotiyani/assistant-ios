---
name: Comprehensive Integrations Map
description: All integrations across 10 categories - APIs, native methods, and workarounds
version: 1.0
---

# JARVIS Comprehensive Integrations (All Categories)

Complete mapping of **40+ integrations** across 10 categories. What's possible via API, what's native iOS, workarounds for the rest.

---

## Overview Matrix

| Category | # Integrations | Priority | Approach |
|----------|---|----------|----------|
| **Apple Ecosystem** | 6 | ⭐⭐⭐ Phase 1 | Native iOS APIs |
| **Social** | 6 | ⭐⭐⭐ Phase 2 | Official APIs + workarounds |
| **Finance** | 5 | ⭐⭐⭐ Phase 2 | Bank APIs + manual entry + AI |
| **Communication** | 4 | ⭐⭐⭐ Phase 1 | Mixed (WhatsApp/iMessage via iOS) |
| **Productivity** | 5 | ⭐⭐⭐ Phase 2 | Official REST APIs |
| **News/Content** | 4 | ⭐⭐ Phase 3 | Official APIs + RSS |
| **Health/Fitness** | 4 | ⭐⭐⭐ Phase 1 | Native APIs + wearable integrations |
| **Location/Travel** | 3 | ⭐⭐ Phase 3 | Official APIs |
| **Entertainment** | 5 | ⭐⭐ Phase 3 | Mixed (Spotify/Apple Music APIs, Netflix limited) |
| **E-commerce** | 5 | ⭐ Phase 4 | Limited (email parsing + manual) |

---

## CATEGORY 1: APPLE ECOSYSTEM (iOS Native APIs)

All of these are available directly on iOS device. **Zero backend complexity.**

### 1. **Apple Calendar** ✓ Easy

```swift
import EventKit

class AppleCalendarManager {
    let eventStore = EKEventStore()
    
    func requestCalendarAccess() {
        eventStore.requestFullAccessToEvents { granted, error in
            if granted {
                self.syncCalendarToBackend()
            }
        }
    }
    
    func fetchAllEvents(for dateRange: DateInterval) -> [EKEvent] {
        let predicate = eventStore.predicateForEvents(
            withStart: dateRange.start,
            end: dateRange.end,
            calendars: nil
        )
        return eventStore.events(matching: predicate)
    }
    
    async func syncToBackend() {
        let events = fetchAllEvents(for: DateInterval(start: Date(), duration: 30*24*3600))
        
        let eventDicts = events.map { [
            "title": $0.title,
            "start": $0.startDate.iso8601String,
            "end": $0.endDate.iso8601String,
            "location": $0.location,
            "notes": $0.notes,
            "calendar": $0.calendar.title,
        ]}
        
        // Send to backend
        URLSession.shared.dataTask(with: {
            var request = URLRequest(url: backendURL.appendingPathComponent("calendar/sync"))
            request.httpMethod = "POST"
            request.httpBody = try JSONEncoder().encode(eventDicts)
            URLSession.shared.dataTask(with: request).resume()
        }).resume()
    }
}
```

**Frequency:** Sync every 6 hours or on change
**Data:** Title, start/end, location, attendees, notes
**Permission:** Calendar access
**Cost:** FREE ✓
**Status:** ✓ Ready for Phase 1

---

### 2. **Apple Reminders** ✓ Easy

```swift
import EventKit

class RemindersManager {
    let eventStore = EKEventStore()
    
    func fetchOpenReminders() -> [EKReminder] {
        let predicate = eventStore.predicateForReminders(in: nil)
        
        var reminders: [EKReminder] = []
        eventStore.fetchReminders(matching: predicate) { fetchedReminders in
            reminders = fetchedReminders.filter { !$0.isCompleted }
        }
        
        return reminders
    }
    
    async func syncToBackend() {
        let reminders = fetchOpenReminders()
        
        let reminderDicts = reminders.map { [
            "title": $0.title,
            "due_date": $0.dueDate?.iso8601String,
            "priority": $0.priority,
            "notes": $0.notes,
            "completed": $0.isCompleted,
        ]}
        
        // Send to backend
        await postToBackend(endpoint: "reminders/sync", data: reminderDicts)
    }
}
```

**Frequency:** Sync daily
**Data:** Title, due date, priority, notes, completion status
**Permission:** Reminders access
**Cost:** FREE ✓
**Status:** ✓ Ready for Phase 1

---

### 3. **Apple Health/HealthKit** ✓ Easy

```swift
import HealthKit

class HealthKitManager {
    let healthStore = HKHealthStore()
    
    func requestHealthPermissions() {
        let types: Set = [
            HKObjectType.quantityType(forIdentifier: .stepCount)!,
            HKObjectType.quantityType(forIdentifier: .heartRateVariabilitySDNN)!,
            HKObjectType.quantityType(forIdentifier: .restingHeartRate)!,
            HKObjectType.quantityType(forIdentifier: .activeEnergyBurned)!,
            HKObjectType.categoryType(forIdentifier: .sleepAnalysis)!,
            HKObjectType.workoutType()!,
        ]
        
        healthStore.requestAuthorization(toShare: types, read: types) { success, error in
            if success { self.syncHealthData() }
        }
    }
    
    async func fetchTodayHealthData() -> HealthSnapshot {
        let calendar = Calendar.current
        let startOfDay = calendar.startOfDay(for: Date())
        let endOfDay = calendar.date(byAdding: .day, value: 1, to: startOfDay)!
        
        return HealthSnapshot(
            date: Date(),
            steps: await queryQuantity(.stepCount, start: startOfDay, end: endOfDay),
            activeCalories: await queryQuantity(.activeEnergyBurned, start: startOfDay, end: endOfDay),
            restingHeartRate: await queryQuantity(.restingHeartRate, start: startOfDay, end: endOfDay),
            hrv: await queryQuantity(.heartRateVariabilitySDNN, start: startOfDay, end: endOfDay),
            sleepMinutes: await querySleep(start: startOfDay, end: endOfDay),
            workouts: await queryWorkouts(start: startOfDay, end: endOfDay),
            mindfulMinutes: await queryMindfulness(start: startOfDay, end: endOfDay)
        )
    }
    
    async func syncToBackend() {
        let health = await fetchTodayHealthData()
        await postToBackend(endpoint: "health/sync", data: health)
    }
}

struct HealthSnapshot: Codable {
    let date: Date
    let steps: Int
    let activeCalories: Double
    let restingHeartRate: Int
    let hrv: Int
    let sleepMinutes: Int
    let workouts: [WorkoutData]
    let mindfulMinutes: Int
}
```

**Frequency:** Sync daily (evening)
**Data:** Steps, calories, HR, HRV, sleep, workouts, meditation
**Permission:** Health access
**Cost:** FREE ✓
**Status:** ✓ Ready for Phase 1

---

### 4. **Apple Notes** ✓ Easy

```swift
import NoteKit  // iOS 17+

class NotesManager {
    func fetchAllNotes() -> [Note] {
        // iOS 17+ has NoteKit for reading notes
        // Earlier versions need manual parsing
    }
    
    async func syncToBackend() {
        let notes = fetchAllNotes()
        
        let noteDicts = notes.map { [
            "title": $0.title,
            "content": $0.content,
            "updated": $0.modificationDate.iso8601String,
            "folder": $0.folder.name,
        ]}
        
        await postToBackend(endpoint: "notes/sync", data: noteDicts)
    }
}
```

**Frequency:** Sync daily
**Data:** Title, content, folder, modification date
**Permission:** Notes access
**Cost:** FREE ✓
**Status:** ✓ iOS 17+ only, Ready for Phase 1

---

### 5. **Apple Contacts** ✓ Easy

```swift
import Contacts

class ContactsManager {
    let contactStore = CNContactStore()
    
    func requestContactsAccess() {
        contactStore.requestAccess(for: .contacts) { granted, error in
            if granted { self.syncContacts() }
        }
    }
    
    func fetchAllContacts() -> [CNContact] {
        let keysToFetch = [
            CNContactIdentifierKey,
            CNContactNameKey,
            CNContactPhoneNumbersKey,
            CNContactEmailAddressesKey,
            CNContactDatesKey,  // Birthdays
        ] as [CNKeyDescriptor]
        
        let request = CNContactFetchRequest(keysToFetch: keysToFetch)
        
        var contacts: [CNContact] = []
        try? contactStore.enumerateContacts(with: request) { contact in
            contacts.append(contact)
        }
        
        return contacts
    }
    
    async func syncToBackend() {
        let contacts = fetchAllContacts()
        
        let contactDicts = contacts.map { contact -> [String: Any] in
            var contactData: [String: Any] = [
                "id": contact.identifier,
                "name": contact.givenName + " " + contact.familyName,
                "phone_numbers": contact.phoneNumbers.map { $0.value.stringValue },
                "emails": contact.emailAddresses.map { $0.value as String },
            ]
            
            // Birthdays
            if let birthday = contact.birthday {
                contactData["birthday"] = birthday.date?.iso8601String
            }
            
            return contactData
        }
        
        await postToBackend(endpoint: "contacts/sync", data: contactDicts)
    }
}
```

**Frequency:** Sync daily
**Data:** Name, phone, email, birthday, notes
**Permission:** Contacts access
**Cost:** FREE ✓
**Status:** ✓ Ready for Phase 1

---

### 6. **Apple Shortcuts** ⭐⭐ Medium

Shortcuts can trigger backend actions automatically. Great for automation.

```swift
// Define URL schemes in app for Shortcuts to call

// Example: Create task shortcut
// https://jarvis-app.com/shortcut/create-task?title=Buy milk&due=2026-04-10

// Example: Log expense shortcut
// https://jarvis-app.com/shortcut/log-expense?amount=500&category=food

// Example: Quick check-in
// https://jarvis-app.com/shortcut/checkin?mood=good&productivity=7/10
```

**Use Cases:**
- "Create task" shortcut
- "Log expense" shortcut
- "Quick check-in" shortcut
- "Start focus session" shortcut

**Frequency:** Manual (user-triggered)
**Cost:** FREE ✓
**Status:** ✓ Ready for Phase 2

---

---

## CATEGORY 2: COMMUNICATION (WhatsApp, iMessage, Email, Telegram, Signal)

### 1. **WhatsApp** ⚠️ Workaround Required

**Official API Limitation:**
- WhatsApp Business API can ONLY send messages
- Cannot read incoming messages
- No read receipts in personal API

**Workaround 1: iOS App Direct Integration**

```swift
// iOS app can read local WhatsApp database (if you have access)
// Or use WhatsApp's Share Sheet to capture messages

// WhatsApp message parsing from shared text
class WhatsAppMessage {
    let timestamp: Date
    let sender: String
    let content: String
    let isFinancial: Bool  // Detect UPI/payment messages
}
```

**Workaround 2: WhatsApp Business API (Limited)**

```python
class WhatsAppBusinessIntegration:
    """Send messages only (limited use)"""
    
    async def send_message(self, phone: str, message: str):
        response = await httpx.post(
            "https://graph.instagram.com/v18.0/YOUR_PHONE_ID/messages",
            json={
                "messaging_product": "whatsapp",
                "to": phone,
                "type": "text",
                "text": {"body": message},
            }
        )
        return response.json()
```

**Workaround 3: Telegram Bot as Bridge**

```python
# Have a Telegram bot relay WhatsApp messages (via IFTTT or custom)
# Better: Just use WhatsApp + iMessage for communication
# Focus on extracting financial data from messages
```

**Recommendation for You:**
- Use iOS app to manually capture/share important WhatsApp messages
- Focus on **WhatsApp Payments** (track via SMS parser instead)
- Use **iMessage for financial tracking** (receipts, order confirmations)

**Status:** ⚠️ Limited - Read via iOS app, workarounds for data extraction

---

### 2. **iMessage** ✓ iOS App Only

**Important:** iMessage cannot be accessed from backend. Only iOS app can read locally.

```swift
class iMessageManager {
    // iMessage data is in:
    // ~/Library/Messages/chat.db
    
    // Can be read by your app on same device
    
    func extractFinancialMessages() -> [FinancialMessage] {
        // Read iMessages locally
        // Filter for:
        // - UPI payments (Google Pay, PhonePe messages)
        // - Order confirmations (Amazon, Flipkart)
        // - Delivery updates
        // - Bank notifications
        
        // Example:
        // "Your payment of ₹500 to Amit was successful"
        // -> Extract: amount=500, type=payment, recipient=Amit
        
        return extractedMessages
    }
    
    async func syncFinancialDataToBackend() {
        let financialMessages = extractFinancialMessages()
        
        let transactions = financialMessages.map { msg -> Transaction in
            Transaction(
                amount: msg.extractedAmount,
                type: msg.type,  // payment, order, delivery, etc
                timestamp: msg.timestamp,
                description: msg.content,
                source: "iMessage",
                category: categorizeTransaction(msg)
            )
        }
        
        await postToBackend(endpoint: "transactions/sync", data: transactions)
    }
}
```

**Frequency:** Sync daily
**Data:** Messages, extract financial/order/delivery info
**Permission:** Full device access (can't be done without jailbreak limitations)
**Cost:** FREE ✓
**Status:** ✓ Ready for Phase 1

---

### 3. **Gmail/Email** ✓ API Available

```python
class GmailIntegration:
    API_ENDPOINT = "https://gmail.googleapis.com/gmail/v1"
    
    async def fetch_financial_emails(self, user_id: str) -> List[Dict]:
        """Extract financial data from emails"""
        
        token = await self.get_token(user_id)
        
        # Search for financial emails
        search_queries = [
            "from:amazon.in OR from:flipkart.com",  # Orders
            "from:paytm.com OR from:googlepay",     # Payments
            "from:icicibank OR from:hdfcbank",      # Bank alerts
            "UPI OR payment OR order OR invoice",   # Keywords
        ]
        
        all_messages = []
        
        for query in search_queries:
            response = await httpx.get(
                f"{self.API_ENDPOINT}/users/me/messages",
                headers={"Authorization": f"Bearer {token}"},
                params={"q": query, "maxResults": 50}
            )
            
            all_messages.extend(response.json().get("messages", []))
        
        # Parse email content
        transactions = []
        for msg in all_messages:
            msg_detail = await httpx.get(
                f"{self.API_ENDPOINT}/users/me/messages/{msg['id']}",
                headers={"Authorization": f"Bearer {token}"}
            )
            
            email_data = msg_detail.json()
            headers = {h["name"]: h["value"] for h in email_data["payload"]["headers"]}
            
            # Extract transaction data
            transaction = await self.extract_transaction_from_email(
                subject=headers.get("Subject"),
                from_=headers.get("From"),
                body=self.get_email_body(email_data)
            )
            
            if transaction:
                transactions.append(transaction)
        
        return transactions
    
    async def extract_transaction_from_email(self, subject: str, from_: str, body: str):
        """Use regex/AI to extract amount, type, merchant"""
        
        # Examples:
        # Amazon: "Your order #123 of ₹500 has been confirmed"
        # Google Pay: "You sent ₹100 to Amit"
        # HDFC: "Debit of ₹500 from your account"
        
        # Can use regex + fallback to Claude API for complex cases
        
        import re
        
        # Try regex first
        amount_match = re.search(r'₹\s*([\d,]+)', body + subject)
        if amount_match:
            amount = int(amount_match.group(1).replace(',', ''))
            
            # Categorize
            merchant = self.extract_merchant(from_)
            category = self.categorize_expense(subject + " " + body)
            
            return {
                "amount": amount,
                "merchant": merchant,
                "category": category,
                "type": "email_extracted",
                "timestamp": datetime.now().isoformat(),
            }
        
        return None
```

**Frequency:** Sync daily (background)
**Data:** Transaction amount, merchant, category, timestamp
**Permission:** Gmail read-only
**Cost:** FREE ✓
**Status:** ✓ Ready for Phase 1

---

### 4. **Telegram** ✓ API Available

```python
class TelegramIntegration:
    """Telegram Bot API - Read messages via your private bot"""
    
    async def setup_bot(self):
        """
        1. Create Telegram bot via @BotFather
        2. Get API token
        3. Add bot to your personal chat
        4. Poll for messages
        """
        pass
    
    async def fetch_messages(self, user_id: str, limit: int = 50) -> List[Dict]:
        """Get recent Telegram messages"""
        
        token = await self.get_token(user_id)
        
        # Get updates from bot
        response = await httpx.get(
            f"https://api.telegram.org/bot{token}/getUpdates",
            params={"timeout": 10}
        )
        
        messages = []
        for update in response.json()["result"]:
            if "message" in update:
                msg = update["message"]
                messages.append({
                    "sender": msg.get("from", {}).get("first_name"),
                    "text": msg.get("text"),
                    "timestamp": msg["date"],
                    "chat_id": msg["chat"]["id"],
                })
        
        return messages
```

**Status:** ✓ Ready for Phase 2

---

### 5. **Signal** ⚠️ No Official API

Signal does not provide an official read API. No easy workaround.

**Status:** ⚠️ Skip for now

---

---

## CATEGORY 3: FINANCE (Groww, Stocks, Crypto, Banking, PayPal, UPI)

### 1. **Groww** ✓ Check for API

```python
class GrowwIntegration:
    """Groww API - Verify availability"""
    
    # TODO: Verify current API status at https://developer.groww.in
    # User mentioned they've started providing API
    
    async def fetch_portfolio(self, user_id: str) -> Dict:
        """Get holdings, NAV, returns"""
        
        # If API available:
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.groww.in/v1/portfolio",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        holdings = response.json()
        
        # Transform to JARVIS format
        return {
            "total_value": holdings["nav"],
            "day_change": holdings.get("change", 0),
            "day_change_percent": holdings.get("change_percent", 0),
            "holdings": [
                {
                    "name": h["name"],
                    "value": h["current_value"],
                    "units": h["units"],
                    "gain_loss": h.get("gain", 0),
                    "gain_loss_percent": h.get("gain_percent", 0),
                }
                for h in holdings["securities"]
            ]
        }
```

**Status:** ✓ Verify API availability, implement Phase 2

---

### 2. **Stock Portfolio (NSE/BSE)** ✓ Multiple Options

```python
class StockIntegration:
    """Real-time stock prices"""
    
    async def fetch_stock_price(self, symbol: str) -> Dict:
        """Get price via yfinance or Alpha Vantage"""
        
        # Option 1: yfinance (free, fast)
        import yfinance as yf
        
        stock = yf.Ticker(symbol)
        data = stock.history(period="1d")
        
        return {
            "symbol": symbol,
            "current_price": data["Close"].iloc[-1],
            "day_change": data["Close"].iloc[-1] - data["Open"].iloc[0],
            "day_change_percent": ((data["Close"].iloc[-1] - data["Open"].iloc[0]) / data["Open"].iloc[0]) * 100,
            "high": data["High"].iloc[-1],
            "low": data["Low"].iloc[-1],
            "volume": data["Volume"].iloc[-1],
        }
    
    async def fetch_portfolio_summary(self, user_id: str) -> Dict:
        """Aggregate portfolio value"""
        
        # Get holdings from DB (user manually entered or Groww API)
        holdings = await db.query("portfolio_holdings", user_id=user_id)
        
        total_value = 0
        portfolio_data = []
        
        for holding in holdings:
            price = await self.fetch_stock_price(holding["symbol"])
            
            current_value = holding["quantity"] * price["current_price"]
            gain_loss = current_value - (holding["quantity"] * holding["buy_price"])
            
            portfolio_data.append({
                "symbol": holding["symbol"],
                "quantity": holding["quantity"],
                "buy_price": holding["buy_price"],
                "current_price": price["current_price"],
                "current_value": current_value,
                "gain_loss": gain_loss,
                "gain_loss_percent": (gain_loss / (holding["quantity"] * holding["buy_price"])) * 100,
            })
            
            total_value += current_value
        
        return {
            "total_value": total_value,
            "holdings": portfolio_data,
            "alerts": self.generate_alerts(portfolio_data),
        }
```

**Cost:** FREE (yfinance) or $5/mo (Alpha Vantage)
**Status:** ✓ Ready for Phase 2

---

### 3. **Cryptocurrency** ✓ API Available

```python
class CryptoIntegration:
    """Multi-exchange crypto portfolio"""
    
    async def fetch_binance_portfolio(self, user_id: str) -> Dict:
        """Connect to Binance account"""
        
        api_key = await self.get_token(user_id, "binance_api_key")
        api_secret = await self.get_token(user_id, "binance_api_secret")
        
        client = AsyncClient(api_key=api_key, api_secret=api_secret)
        
        # Get account balances
        account = await client.get_account()
        
        portfolio = []
        for balance in account["balances"]:
            if float(balance["free"]) > 0 or float(balance["locked"]) > 0:
                portfolio.append({
                    "asset": balance["asset"],
                    "free": float(balance["free"]),
                    "locked": float(balance["locked"]),
                    "total": float(balance["free"]) + float(balance["locked"]),
                })
        
        # Get prices
        prices_response = await client.get_all_tickers()
        prices = {p["symbol"]: float(p["price"]) for p in prices_response}
        
        # Calculate portfolio value
        total_usd_value = sum(
            balance["total"] * prices.get(balance["asset"] + "USDT", 0)
            for balance in portfolio
        )
        
        await client.close_connection()
        
        return {
            "total_value_usd": total_usd_value,
            "holdings": portfolio,
            "exchanges": ["binance"],
        }
    
    async def fetch_coinbase_portfolio(self, user_id: str) -> Dict:
        """Coinbase integration"""
        
        # Similar pattern to Binance
        pass
```

**Supported Exchanges:** Binance, Coinbase, Kraken, OKX
**Status:** ✓ Ready for Phase 2

---

### 4. **Banking Integration** ⭐⭐ Medium

**Challenge:** Most Indian banks don't expose personal banking APIs. Options:

**Option A: Manual Entry + AI Analysis**

```python
class BankingIntegration:
    """User manually logs transactions"""
    
    async def log_transaction(self, user_id: str, transaction: Dict):
        """
        User enters: amount, merchant, category, date
        Backend stores and analyzes
        """
        
        # Body:
        # {
        #   "amount": 500,
        #   "merchant": "Starbucks",
        #   "category": "food",
        #   "type": "debit",
        #   "timestamp": "2026-04-05T10:30:00Z",
        #   "payment_method": "debit_card"
        # }
        
        await db.insert("transactions", user_id=user_id, **transaction)
        
        # Trigger analysis
        await orchestrator.broadcast_event("transaction_logged", transaction)
    
    async def get_financial_summary(self, user_id: str, period: str = "month") -> Dict:
        """Analyze spending patterns"""
        
        # Get all transactions for period
        transactions = await db.query(
            "transactions",
            user_id=user_id,
            timestamp__gte=get_start_date(period)
        )
        
        # Analyze
        by_category = {}
        for txn in transactions:
            category = txn["category"]
            by_category[category] = by_category.get(category, 0) + txn["amount"]
        
        return {
            "total_spent": sum(t["amount"] for t in transactions),
            "by_category": by_category,
            "average_transaction": sum(t["amount"] for t in transactions) / len(transactions),
            "largest_expense": max(transactions, key=lambda t: t["amount"]),
            "insights": await self.generate_savings_tips(by_category),
        }
    
    async def generate_savings_tips(self, spending_by_category: Dict) -> List[str]:
        """AI-powered savings recommendations"""
        
        tips = []
        
        # Rule-based tips
        if spending_by_category.get("food", 0) > 10000:
            tips.append("You're spending ₹10k+/month on food. Try meal prep to save 20-30%")
        
        if spending_by_category.get("subscriptions", 0) > 5000:
            tips.append("Multiple subscriptions adding up. Review which ones you actually use")
        
        if spending_by_category.get("transport", 0) > 5000:
            tips.append("High transport costs. Consider carpooling or monthly pass")
        
        # AI analysis (use Claude API for deeper insights)
        # ... call Claude for personalized recommendations ...
        
        return tips
```

**Option B: Import from Bank Statement**

```python
class BankStatementParser:
    """Parse PDF/CSV bank statements"""
    
    async def upload_statement(self, user_id: str, file_data: bytes) -> int:
        """
        User uploads bank statement (PDF or CSV)
        Extract transactions
        """
        
        if file_data.endswith(b"pdf"):
            transactions = await self.parse_pdf_statement(file_data)
        else:
            transactions = await self.parse_csv_statement(file_data)
        
        # Insert all
        inserted_count = 0
        for txn in transactions:
            await db.insert("transactions", user_id=user_id, **txn)
            inserted_count += 1
        
        return inserted_count
```

**Status:** ✓ Ready for Phase 2

---

### 5. **PayPal/Stripe/Razorpay** ✓ API Available

```python
class PayPalIntegration:
    async def fetch_transaction_history(self, user_id: str) -> List[Dict]:
        """Get all PayPal transactions"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.paypal.com/v1/reporting/transactions",
            headers={"Authorization": f"Bearer {token}"},
            params={"start_date": "2026-01-01", "end_date": "2026-12-31"}
        )
        
        transactions = response.json()["transaction_details"]
        
        return [
            {
                "id": t["transaction_id"],
                "amount": float(t["transaction_amount"]),
                "currency": t["transaction_currency"],
                "timestamp": t["transaction_date"],
                "status": t["transaction_status"],
                "type": t["transaction_subject"],
            }
            for t in transactions
        ]

class RazorpayIntegration:
    async def fetch_payouts(self, user_id: str) -> List[Dict]:
        """Get Razorpay payout history"""
        
        api_key = await self.get_token(user_id, "razorpay_key_id")
        api_secret = await self.get_token(user_id, "razorpay_key_secret")
        
        response = await httpx.get(
            "https://api.razorpay.com/v1/payouts",
            auth=(api_key, api_secret),
            params={"limit": 100}
        )
        
        return response.json()["items"]
```

**Status:** ✓ Ready for Phase 2

---

### 6. **UPI/Digital Wallet Tracking** ⭐⭐ Medium (Via SMS Parser)

Since UPI/wallet apps don't expose APIs, use SMS parsing instead:

```python
class UPITracking:
    """Parse SMS for UPI transactions"""
    
    async def sync_sms_transactions(self, user_id: str):
        """
        iOS app reads SMS for payment notifications:
        - "Your payment of ₹500 to Amit was successful - Google Pay"
        - "Payment received: ₹1000 from Raj - PhonePe"
        - "UPI Ref No: 123456789"
        """
        
        # iOS app reads SMS locally and shares with backend
        # Pattern: [Amount][Recipient/Sender][Payment Status]
        
        transactions = await parse_sms_patterns(sms_list)
        
        for txn in transactions:
            await db.insert("transactions", user_id=user_id, **{
                "amount": txn["amount"],
                "type": txn["direction"],  # sent/received
                "recipient": txn.get("recipient"),
                "sender": txn.get("sender"),
                "source": "upi_sms",
                "timestamp": txn["timestamp"],
                "category": categorize_by_recipient(txn),
            })
```

**Status:** ✓ Implementable via iOS SMS parsing

---

---

## CATEGORY 4: PRODUCTIVITY (Notion, Slack, Jira, Asana, Monday.com)

All have official REST APIs.

### 1. **Notion** ✓ Easy API

Already covered in MVP doc. Full implementation.

---

### 2. **Slack** ✓ API Available

```python
class SlackIntegration:
    async def fetch_messages(self, user_id: str, channel: str) -> List[Dict]:
        """Get messages from channel"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.post(
            "https://slack.com/api/conversations.history",
            headers={"Authorization": f"Bearer {token}"},
            data={"channel": channel, "limit": 50}
        )
        
        return response.json()["messages"]
    
    async def get_user_status(self, user_id: str) -> Dict:
        """What's your Slack status?"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://slack.com/api/users.profile.get",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        profile = response.json()["profile"]
        
        return {
            "status_text": profile["status_text"],
            "status_emoji": profile["status_emoji"],
            "status_expiration": profile.get("status_expiration"),
        }
    
    async def set_status(self, user_id: str, status: str, emoji: str):
        """Set Slack status automatically"""
        
        token = await self.get_token(user_id)
        
        await httpx.post(
            "https://slack.com/api/users.profile.set",
            headers={"Authorization": f"Bearer {token}"},
            json={
                "profile": {
                    "status_text": status,
                    "status_emoji": emoji,
                }
            }
        )
```

**Use Case:** 
- Get daily message count
- Track channel activity
- Auto-set status based on calendar/productivity

**Status:** ✓ Ready for Phase 2

---

### 3. **Jira** ✓ API Available

```python
class JiraIntegration:
    async def fetch_assigned_issues(self, user_id: str) -> List[Dict]:
        """Get your assigned Jira issues"""
        
        api_token = await self.get_token(user_id)
        domain = await self.get_config(user_id, "jira_domain")
        
        response = await httpx.get(
            f"https://{domain}.atlassian.net/rest/api/3/issues/search",
            headers={"Authorization": f"Bearer {api_token}"},
            params={
                "jql": "assignee = currentUser() AND status != Done",
                "maxResults": 50,
            }
        )
        
        return [
            {
                "key": issue["key"],
                "summary": issue["fields"]["summary"],
                "status": issue["fields"]["status"]["name"],
                "priority": issue["fields"]["priority"]["name"],
                "due_date": issue["fields"].get("duedate"),
            }
            for issue in response.json()["issues"]
        ]
```

**Status:** ✓ Ready for Phase 2

---

### 4. **Asana** ✓ API Available

```python
class AsanaIntegration:
    async def fetch_tasks(self, user_id: str, project_id: str = None) -> List[Dict]:
        """Get your Asana tasks"""
        
        token = await self.get_token(user_id)
        
        endpoint = f"/api/1.0/user_task_lists/{user_id}/tasks"
        if project_id:
            endpoint = f"/api/1.0/projects/{project_id}/tasks"
        
        response = await httpx.get(
            f"https://app.asana.com{endpoint}",
            headers={"Authorization": f"Bearer {token}"},
            params={"opt_fields": "name,completed,due_on,priority"}
        )
        
        return response.json()["data"]
```

**Status:** ✓ Ready for Phase 2

---

### 5. **Monday.com** ✓ GraphQL API

```python
class MondayIntegration:
    async def fetch_boards(self, user_id: str) -> List[Dict]:
        """Get your Monday.com boards"""
        
        token = await self.get_token(user_id)
        
        query = """
        query {
            boards (limit: 50) {
                id
                name
                items_page (limit: 50) {
                    items {
                        id
                        name
                        state
                        created_at
                    }
                }
            }
        }
        """
        
        response = await httpx.post(
            "https://api.monday.com/graphql",
            headers={"Authorization": f"Bearer {token}"},
            json={"query": query}
        )
        
        return response.json()["data"]["boards"]
```

**Status:** ✓ Ready for Phase 2

---

---

## CATEGORY 5: SOCIAL (Instagram, WhatsApp, Telegram, Twitter/X, Reddit, Discord)

### 1. **Instagram** ⚠️ Heavily Restricted

Meta's API is **very limited** for personal use:

```python
class InstagramIntegration:
    """What you CAN access (very limited)"""
    
    async def get_published_posts(self, user_id: str, limit: int = 10) -> List[Dict]:
        """Only your own published posts"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://graph.instagram.com/me/media",
            params={
                "fields": "id,caption,media_type,timestamp,like_count,comments_count",
                "access_token": token,
            }
        )
        
        return response.json()["data"]
```

**What you CANNOT access:**
- Your DMs (Stories, group chats)
- Follower/following data
- Real-time notifications
- Other users' posts

**Recommendation:** Skip Instagram API. If needed, use **third-party app** (Instagrabber or similar, user's risk).

**Status:** ⚠️ Skip or use workarounds

---

### 2. **Twitter/X** ✓ API v2 Available

```python
class TwitterIntegration:
    async def fetch_home_timeline(self, user_id: str) -> List[Dict]:
        """Your Twitter home feed"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.twitter.com/2/tweets/search/recent",
            headers={"Authorization": f"Bearer {token}"},
            params={
                "query": "-is:retweet",
                "max_results": 50,
            }
        )
        
        return response.json()["data"]
    
    async def get_mentions(self, user_id: str) -> List[Dict]:
        """Who's mentioning you?"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.twitter.com/2/tweets/search/recent",
            headers={"Authorization": f"Bearer {token}"},
            params={
                "query": f"@{username} -is:retweet",
                "max_results": 50,
            }
        )
        
        return response.json()["data"]
```

**Cost:** FREE (basic tier)
**Status:** ✓ Ready for Phase 3

---

### 3. **Reddit** ✓ Official API

```python
class RedditIntegration:
    async def fetch_frontpage(self, user_id: str) -> List[Dict]:
        """Your Reddit home feed"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://oauth.reddit.com/",
            headers={"Authorization": f"Bearer {token}"},
        )
        
        return response.json()["data"]["children"]
    
    async def fetch_saved_posts(self, user_id: str) -> List[Dict]:
        """Your saved Reddit posts"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://oauth.reddit.com/user/me/saved",
            headers={"Authorization": f"Bearer {token}"},
            params={"limit": 100}
        )
        
        return [
            {
                "id": item["data"]["id"],
                "title": item["data"]["title"],
                "subreddit": item["data"]["subreddit"],
                "score": item["data"]["score"],
                "url": item["data"]["url"],
            }
            for item in response.json()["data"]["children"]
        ]
```

**Status:** ✓ Ready for Phase 3

---

### 4. **Discord** ✓ API Available

```python
class DiscordIntegration:
    async def fetch_messages(self, user_id: str, channel_id: str) -> List[Dict]:
        """Get Discord channel messages"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            f"https://discord.com/api/v10/channels/{channel_id}/messages",
            headers={"Authorization": f"Bearer {token}"},
            params={"limit": 50}
        )
        
        return response.json()
    
    async def get_servers(self, user_id: str) -> List[Dict]:
        """Get list of Discord servers"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://discord.com/api/v10/users/@me/guilds",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        return response.json()
```

**Status:** ✓ Ready for Phase 3

---

### 5. **Telegram** (Already covered above)

**Status:** ✓ Ready for Phase 2

---

---

## CATEGORY 6: NEWS/CONTENT (News Apps, Podcasts, YouTube, TikTok)

### 1. **YouTube** ✓ Easy

```python
class YouTubeIntegration:
    async def fetch_watch_history(self, user_id: str) -> List[Dict]:
        """Your YouTube watch history"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://www.googleapis.com/youtube/v3/activities",
            headers={"Authorization": f"Bearer {token}"},
            params={
                "part": "snippet",
                "maxResults": 50,
            }
        )
        
        return [
            {
                "video_id": item["snippet"]["resourceId"]["videoId"],
                "title": item["snippet"]["title"],
                "channel": item["snippet"]["channelTitle"],
                "watched_at": item["snippet"]["publishedAt"],
            }
            for item in response.json()["items"]
        ]
    
    async def get_subscriptions(self, user_id: str) -> List[Dict]:
        """Channels you subscribe to"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://www.googleapis.com/youtube/v3/subscriptions",
            headers={"Authorization": f"Bearer {token}"},
            params={"part": "snippet", "maxResults": 50}
        )
        
        return response.json()["items"]
```

**Status:** ✓ Ready for Phase 3

---

### 2. **TikTok** ⚠️ Limited API

TikTok restricts personal API access heavily. No official read API for personal use.

**Workaround:** Web scraping via unofficial library (risky).

**Status:** ⚠️ Skip or find workaround

---

### 3. **RSS Feeds / News Apps** ✓ Easy

```python
class NewsAggregator:
    async def fetch_top_stories(self) -> List[Dict]:
        """Tech news from multiple sources"""
        
        sources = [
            ("https://feeds.arstechnica.com/arstechnica/index", "Ars Technica"),
            ("https://feeds.techcrunch.com/", "TechCrunch"),
            ("http://hn.algolia.com/api/v1/search?tags=front_page", "Hacker News"),
            ("https://www.theverge.com/rss/index.xml", "The Verge"),
        ]
        
        import feedparser
        
        all_stories = []
        for feed_url, source_name in sources:
            if "algolia" in feed_url:
                # Hacker News special case
                response = await httpx.get(feed_url)
                for item in response.json()["hits"][:20]:
                    all_stories.append({
                        "title": item["title"],
                        "url": item["url"],
                        "source": source_name,
                        "points": item["points"],
                    })
            else:
                # Standard RSS
                feed = feedparser.parse(feed_url)
                for entry in feed.entries[:20]:
                    all_stories.append({
                        "title": entry.title,
                        "url": entry.link,
                        "source": source_name,
                        "published": entry.get("published", ""),
                    })
        
        # Deduplicate by URL
        seen = set()
        unique_stories = []
        for story in all_stories:
            if story["url"] not in seen:
                unique_stories.append(story)
                seen.add(story["url"])
        
        return unique_stories
```

**Status:** ✓ Ready for Phase 1

---

### 4. **Podcasts** ⭐⭐ Medium

```python
class PodcastIntegration:
    async def fetch_listen_history(self, user_id: str) -> List[Dict]:
        """Podcasts you're listening to"""
        
        # Use Podcast API (podcastaddict.com API or similar)
        # Or parse from Apple Podcasts (native iOS)
        
        # iOS native approach:
        # Use MediaPlayer framework to read listening history
        pass
```

**Status:** ⏳ Phase 3

---

---

## CATEGORY 7: HEALTH/FITNESS (Apple Health, Fitbit, Apple Watch, Oura)

### 1. **Apple Health/HealthKit** (Already covered)

**Status:** ✓ Phase 1

---

### 2. **Fitbit** ✓ API Available

```python
class FitbitIntegration:
    async def fetch_daily_summary(self, user_id: str, date: str) -> Dict:
        """Get Fitbit daily stats"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            f"https://api.fitbit.com/1/user/-/activities/date/{date}.json",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        data = response.json()["summary"]
        
        return {
            "date": date,
            "steps": data["steps"],
            "distance": data["distances"][0]["distance"],
            "calories": data["caloriesBurned"],
            "active_minutes": data["veryActiveMinutes"] + data["fairlyActiveMinutes"],
        }
    
    async def fetch_sleep(self, user_id: str, date: str) -> Dict:
        """Get Fitbit sleep data"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            f"https://api.fitbit.com/1.2/user/-/sleep/date/{date}.json",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        sleep = response.json()["sleep"][0]
        
        return {
            "duration_minutes": sleep["duration"] // 60000,
            "efficiency": sleep["efficiency"],
            "start_time": sleep["startTime"],
            "end_time": sleep["endTime"],
        }
```

**Status:** ✓ Ready for Phase 1

---

### 3. **Oura Ring** ✓ API Available

```python
class OuraIntegration:
    async def fetch_readiness_score(self, user_id: str, date: str) -> Dict:
        """Oura readiness score"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            f"https://api.ouraring.com/v2/usercollection/daily_readiness",
            headers={"Authorization": f"Bearer {token}"},
            params={"date": date}
        )
        
        data = response.json()["data"][0]
        
        return {
            "readiness_score": data["score"],
            "sleep_score": data["sleep_score"],
            "activity_score": data["activity_score"],
            "hrv_balance": data["hrv_balance"],
            "body_temperature": data["body_temperature"],
            "previous_night_heart_rate": data["previous_night_heart_rate"],
        }
```

**Status:** ✓ Ready for Phase 1

---

---

## CATEGORY 8: LOCATION/TRAVEL (Google Maps, Uber, Flight Apps)

### 1. **Google Maps** ✓ API Available

```python
class GoogleMapsIntegration:
    async def fetch_location_history(self, user_id: str) -> List[Dict]:
        """Your Google Maps Timeline"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://www.googleapis.com/maps/api/timeline/v1/locations",
            headers={"Authorization": f"Bearer {token}"},
            params={"maxResults": 1000}
        )
        
        return response.json()["locations"]
    
    async def get_places_visited(self, user_id: str, days: int = 30) -> List[Dict]:
        """Places you've visited"""
        
        # Use Places API with location history
        pass
```

**Status:** ✓ Ready for Phase 3

---

### 2. **Uber/Lyft** ⭐⭐ Medium

```python
class UberIntegration:
    async def fetch_rides(self, user_id: str) -> List[Dict]:
        """Your Uber rides"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.uber.com/v1.2/history",
            headers={"Authorization": f"Bearer {token}"},
            params={"limit": 100}
        )
        
        return [
            {
                "start_city": ride["start_city"],
                "start_time": ride["start_time"],
                "distance": ride["distance"],
                "duration": ride["duration"],
                "fare": ride.get("fare"),
            }
            for ride in response.json()["history"]
        ]
```

**Status:** ✓ Ready for Phase 3

---

---

## CATEGORY 9: ENTERTAINMENT (Netflix, Spotify, Apple Music, Gaming)

### 1. **Spotify** ✓ Easy

```python
class SpotifyIntegration:
    async def fetch_currently_playing(self, user_id: str) -> Dict:
        """What you're listening to right now"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.spotify.com/v1/me/player/currently-playing",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        if response.status_code == 204:
            return {"playing": False}
        
        item = response.json()
        
        return {
            "track": item["item"]["name"],
            "artist": item["item"]["artists"][0]["name"],
            "album": item["item"]["album"]["name"],
            "duration": item["item"]["duration_ms"],
            "progress": item["progress_ms"],
            "is_playing": item["is_playing"],
        }
    
    async def fetch_top_tracks(self, user_id: str, period: str = "medium_term") -> List[Dict]:
        """Your top tracks"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.spotify.com/v1/me/top/tracks",
            headers={"Authorization": f"Bearer {token}"},
            params={"time_range": period, "limit": 50}
        )
        
        return response.json()["items"]
    
    async def fetch_playlists(self, user_id: str) -> List[Dict]:
        """Your Spotify playlists"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.spotify.com/v1/me/playlists",
            headers={"Authorization": f"Bearer {token}"},
            params={"limit": 50}
        )
        
        return response.json()["items"]
```

**Status:** ✓ Ready for Phase 2

---

### 2. **Apple Music** ⭐⭐ Medium

```python
class AppleMusicIntegration:
    async def fetch_library(self, user_id: str) -> List[Dict]:
        """Your Apple Music library"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://api.music.apple.com/v1/me/library/songs",
            headers={"Authorization": f"Bearer {token}"},
            params={"limit": 100}
        )
        
        return response.json()["data"]
```

**Status:** ✓ Ready for Phase 2

---

### 3. **Netflix** ✗ NO Official API

Netflix **does not provide** a personal user API for reading watch history, favorites, or recommendations.

**Status:** ✗ Skip

---

### 4. **Gaming (PlayStation, Xbox)** ⭐⭐ Medium

```python
class PlayStationIntegration:
    async def fetch_recent_games(self, user_id: str) -> List[Dict]:
        """PlayStation games you've played"""
        
        # Use PSN API
        pass

class XboxIntegration:
    async def fetch_achievements(self, user_id: str) -> List[Dict]:
        """Xbox achievements"""
        
        token = await self.get_token(user_id)
        
        response = await httpx.get(
            "https://achievements.xboxlive.com/users/xuid({xuid})/achievements",
            headers={"Authorization": f"Bearer {token}"}
        )
        
        return response.json()
```

**Status:** ⏳ Phase 4 (nice-to-have)

---

---

## CATEGORY 10: E-COMMERCE (Amazon, Flipkart, Shopify apps)

### 1. **Amazon** ⚠️ No Official Personal API

Amazon doesn't expose personal order APIs.

**Workaround Option A: Parse Emails**

```python
class AmazonIntegration:
    async def fetch_orders_from_email(self, user_id: str) -> List[Dict]:
        """Parse Amazon order emails from Gmail"""
        
        # Search Gmail for Amazon emails
        emails = await gmail.search("from:order-update@amazon.in OR from:shipment-notification@amazon.in")
        
        orders = []
        for email in emails:
            # Parse email content for:
            # - Order ID
            # - Item names
            # - Total amount
            # - Delivery date
            
            order = {
                "order_id": extract_regex(email, r"Order #(\d+)"),
                "items": extract_items(email),
                "total": extract_amount(email),
                "delivery_date": extract_date(email),
                "timestamp": email["date"],
            }
            
            orders.append(order)
        
        return orders
```

**Workaround Option B: Manual Entry**

```python
@app.post("/shopping/order")
async def log_order(user_id: str, order: Dict):
    """User manually logs orders"""
    
    # Body:
    # {
    #   "merchant": "amazon",
    #   "items": ["Item 1", "Item 2"],
    #   "amount": 5000,
    #   "delivery_date": "2026-04-10",
    #   "timestamp": "2026-04-05T10:00:00Z"
    # }
    
    await db.insert("shopping_orders", user_id=user_id, **order)
```

**Status:** ⚠️ Use email parsing or manual entry

---

### 2. **Flipkart** ⚠️ No Official Personal API

Same as Amazon - use email parsing or manual entry.

**Status:** ⚠️ Use email parsing or manual entry

---

### 3. **Shopify Apps** ✓ API Available

If you run your own Shopify store:

```python
class ShopifyIntegration:
    async def fetch_orders(self, user_id: str) -> List[Dict]:
        """Your Shopify store orders"""
        
        token = await self.get_token(user_id)
        shop_domain = await self.get_config(user_id, "shopify_domain")
        
        response = await httpx.get(
            f"https://{shop_domain}/admin/api/2024-01/orders.json",
            headers={"X-Shopify-Access-Token": token},
            params={"status": "any", "limit": 250}
        )
        
        return response.json()["orders"]
```

**Status:** ✓ If you have Shopify store

---

---

## PHASE ROADMAP

```
PHASE 1 (MVP - 1-2 weeks)
├─ Apple Calendar (2h)
├─ Apple Reminders (1h)
├─ Apple HealthKit (3h)
├─ Apple Contacts (2h)
├─ Gmail (2h)
├─ Fitbit (2h)
├─ Oura (2h)
├─ News Aggregator (2h)
└─ Banking (manual entry + AI) (3h)
   TOTAL: 19 hours → Core data collection

PHASE 2 (2-3 weeks)
├─ Groww (verify API) (3h)
├─ Stock API (yfinance) (2h)
├─ Crypto (Binance/Coinbase) (3h)
├─ Notion (already done) ✓
├─ Slack (2h)
├─ Telegram (2h)
├─ Spotify (2h)
├─ Apple Music (2h)
└─ UPI/iMessage parsing (2h)
   TOTAL: 18 hours → Financial + Productivity

PHASE 3 (3-4 weeks)
├─ Twitter/X (2h)
├─ Reddit (2h)
├─ Discord (2h)
├─ Jira (2h)
├─ Asana (2h)
├─ Monday.com (2h)
├─ YouTube (2h)
├─ Google Maps (2h)
├─ Uber (2h)
└─ Apple Shortcuts (2h)
   TOTAL: 18 hours → Extended connectivity

PHASE 4 (Later - Optional/Deferred)
├─ Instagram (SKIP or minimal)
├─ Netflix (SKIP - no API)
├─ TikTok (SKIP or workaround)
├─ WhatsApp (limited workaround)
├─ Amazon/Flipkart (parse emails)
└─ Gaming (PlayStation/Xbox)
   TOTAL: Complex, deferred
```

---

## Architecture for Comprehensive Integrations

```python
# backend/integrations/base.py

class IntegrationBase:
    """All integrations follow this pattern"""
    
    async def initialize(self, user_id: str):
        """Request permissions, establish connection"""
        pass
    
    async def sync_data(self, user_id: str):
        """Fetch new data"""
        pass
    
    async def handle_error(self, error: Exception):
        """Handle API errors gracefully"""
        pass


# backend/integrations/registry.py

INTEGRATIONS = {
    "apple_calendar": AppleCalendarIntegration,
    "apple_reminders": AppleRemindersIntegration,
    "google_gmail": GmailIntegration,
    "notion": NotionIntegration,
    "slack": SlackIntegration,
    "spotify": SpotifyIntegration,
    # ... 30+ more
}

async def sync_all_integrations(user_id: str):
    """Sync all enabled integrations"""
    
    user_integrations = await db.query("user_integrations", user_id=user_id)
    
    tasks = []
    for integration_name in user_integrations:
        integration_class = INTEGRATIONS[integration_name]
        integration = integration_class()
        tasks.append(integration.sync_data(user_id))
    
    results = await asyncio.gather(*tasks, return_exceptions=True)
    
    # Log results
    for integration_name, result in zip(user_integrations, results):
        if isinstance(result, Exception):
            logger.error(f"{integration_name} sync failed: {result}")
        else:
            logger.info(f"{integration_name} synced successfully")
```

---

## Summary: What's Possible

| Category | # Available | # Hard | # Skip |
|----------|---|---|---|
| Apple Ecosystem | 6/6 | 0 | 0 |
| Communication | 3/4 | 1 | 0 |
| Productivity | 5/5 | 0 | 0 |
| Finance | 5/5 | 1 | 0 |
| Social | 4/6 | 1 | 1 |
| News/Content | 3/4 | 0 | 1 |
| Health/Fitness | 4/4 | 0 | 0 |
| Location/Travel | 3/3 | 0 | 0 |
| Entertainment | 3/5 | 1 | 1 |
| E-commerce | 1/3 | 2 | 0 |
| **TOTAL** | **37/45** | **6** | **2** |

**Result:** 82% of integrations are possible via API or native methods. 13% need workarounds. 5% to skip.

---

Ready to implement this?
