# Choice of Firebase Cloud Firestore as DB and Schema Design

We will use ``Firebase Cloud Firestore`` rather than other relational database by the need for **real-time synchronization**, **scalability**, and **flexibility** with a schema-less data structure. It also simplifies development with its built-in features like **serverless architecture**, **authentication integration**, and **offline support**.

## Firebase Firestore: Advs

> When choosing Firebase Firestore over a relational database like MySQL for our project, the decision is driven by several key advantages. First, Firestore’s **flexible NoSQL structure** allows us to adapt easily as our data evolves, without the rigid schemas and complex migrations MySQL requires. Second, it offers **real-time synchronization**, so our users can see cryptocurrency price changes and portfolio updates instantly, without the need for additional infrastructure. Firestore is also **serverless**, meaning it scales automatically without us worrying about managing databases or performance tuning as we grow. Moreover, it seamlessly integrates with **Firebase Authentication**, simplifying user management and security. Finally, Firestore’s **offline support** lets our users access and update data even when they're disconnected, with automatic syncing when they're back online. In short, Firestore provides a scalable, flexible, and real-time solution that is ideal for modern applications focus on real-time like ours.

## Database Schema

Since Firestore is a NoSQL database and essentially stores documents in collections, I structure the schema in following way.

We have two main collections: **Users** and **Portfolios**. The **Users** collection will store user-related data, while the **Portfolios** collection will handle the user’s cryptocurrency holdings and transactions. Let's break it down:

---

### **Database Schema Summary for Firebase Cloud Firestore:**

Your Firebase Firestore database consists of two main collections: **Users** and **Portfolios**. Here's a breakdown of the schema:

---

### **1. Users Collection**

The `Users` collection stores user-specific data, primarily managing the user's profile and preferences. The document ID for each user corresponds to their Firebase Authentication UID.

#### **Fields:**

- **userId**: The Firebase Authentication UID that uniquely identifies the user.
- **email**: The user's email address.
- **firstName**: The user's first name.
- **lastName**: The user's last name.
- **createdAt**: Timestamp indicating when the user account was created.
- **preferences**: An object storing user-specific settings such as:
  - **currency**: The user's preferred currency (e.g., USD, EUR).
  - **theme**: The user's theme preference (e.g., light, dark, auto).

#### **Example Document:**

```json
{
  "userId": "uid123",
  "email": "user@example.com",
  "firstName": "Azriel",
  "lastName": "Wang",
  "createdAt": { "seconds": 1672876800, "nanoseconds": 0 },
  "preferences": {
    "currency": "USD",
    "theme": "auto"
  }
}
```

---

### **2. Portfolios Collection**

Each user has a corresponding portfolio document within the `Portfolios` collection, which tracks their cryptocurrency holdings and related financial data.

#### **Portfolio Fields:**

- **portfolioId**: Unique ID for the portfolio.
- **userId**: The ID of the user owning the portfolio, serving as a reference to the `Users` collection.
- **cryptocurrency**: An array that stores information about the user's current holdings, including:
  - **symbol**: Cryptocurrency symbol (e.g., BTC, ETH).
  - **amount**: The total quantity of the cryptocurrency owned.
  - **averagePrice**: The average price the user paid for the cryptocurrency (based on their buy transactions).
- **revenue**: The total revenue for the portfolio (calculated based on transactions).

#### **Example Portfolio Document:**

```json
{
  "portfolioId": "portfolio123",
  "userId": "user123",
  "cryptocurrency": [
    {
      "symbol": "BTC",
      "amount": 0.1,
      "averagePrice": 40000
    },
    {
      "symbol": "ETH",
      "amount": 2.5,
      "averagePrice": 3000
    }
  ],
  "revenue": 1000
}
```

---

### **Transactions Sub-Collection**

Each portfolio document contains a **Transactions** sub-collection to track the user's **buy**, **sell**, and **transfer** activities. Every transaction is stored as a separate document inside this sub-collection.

#### **Transaction Fields:**

- **transactionId**: Auto-generated unique identifier for each transaction.
- **type**: The type of transaction (e.g., "buy", "sell", or "transfer").
- **cryptocurrency**: The cryptocurrency involved in the transaction (symbol).
- **amount**: The amount of cryptocurrency transacted.
- **price**: The price per unit at the time of the transaction (for buy/sell transactions).
- **transferType**: For transfers, indicates whether it's a transfer "in" (deposit) or "out" (withdrawal).
- **date**: Timestamp representing the date and time of the transaction.
- **fees**: (Optional) Transaction fees associated with the trade or transfer.
- **notes**: (Optional) Additional notes about the transaction.

#### **Example Transactions:**

1. **Buy Transaction:**

```json
{
  "transactionId": "transaction1",
  "type": "buy",
  "cryptocurrency": "BTC",
  "amount": 0.1,
  "price": 40000,
  "date": { "seconds": 1678876800, "nanoseconds": 0 },
  "fees": 10,
  "notes": "Purchased on Binance"
}
```

2. **Sell Transaction:**

```json
{
  "transactionId": "transaction2",
  "type": "sell",
  "cryptocurrency": "ETH",
  "amount": 1.0,
  "price": 3500,
  "date": { "seconds": 1682876800, "nanoseconds": 0 },
  "fees": 5,
  "notes": "Sold on Kraken"
}
```

3. **Transfer Transaction (In):**

```json
{
  "transactionId": "transaction3",
  "type": "transfer",
  "cryptocurrency": "BTC",
  "amount": 0.5,
  "transferType": "in",
  "date": { "seconds": 1685876800, "nanoseconds": 0 },
  "fees": 2,
  "notes": "Transferred from Coinbase"
}
```

4. **Transfer Transaction (Out):**

```json
{
  "transactionId": "transaction4",
  "type": "transfer",
  "cryptocurrency": "ETH",
  "amount": 0.8,
  "transferType": "out",
  "date": { "seconds": 1688876800, "nanoseconds": 0 },
  "fees": 3,
  "notes": "Transferred to external wallet"
}
```

---

### Summary

- **Users Collection** stores basic user information and preferences.
- **Portfolios Collection** stores each user’s cryptocurrency holdings and revenue.
- **Transactions Sub-Collection** within each portfolio tracks individual transactions (buy, sell, transfer) with relevant details such as amounts, dates, and fees.