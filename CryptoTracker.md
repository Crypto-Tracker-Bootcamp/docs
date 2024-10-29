# CryptoTracker Document

[TOC]



## 1.Abstract

Crypto Tracker is a web application designed for anyone interested in keeping up with cryptocurrency trends. The main goal of this project is to provide users with an easy way to track cryptocurrency prices, view market trends, and stay updated on the latest news. Using data from the CoinGecko API, the app lets users search for specific cryptocurrencies, check real-time market info, and view historical price charts. It also includes a news feed section for recent crypto updates. Users can create accounts, save their favorite cryptocurrencies, and even set up price alerts with Firebase authentication. Additionally, Crypto Tracker has a portfolio tracker, allowing users to add their holdings and see their total investment value in one place.



## 2.Functions

### 2.1 Log in and Register  

Allows users to create an account and securely log in to access personalized features, such as saving favorite cryptocurrencies and setting price alerts.

### 2.2 Cryptocurrency Price Search  & Historical Data  

Provides a search function for users to look up specific cryptocurrencies by name or symbol and view current price and market information. Offers charts and historical data, allowing users to analyze the performance of cryptocurrencies over time.

### 2.3 Market Trends  & News Feed

Displays real-time trends in the cryptocurrency market, including information on price fluctuations, trading volumes, and market capitalization. Integrates a news section to show the latest news and updates related to the cryptocurrency market, helping users stay informed.

### 2.4 User Accounts  

Enables users to create personal accounts through Firebase authentication, where they can save their favorite cryptocurrencies and set alerts for specific price points.

### 2.5 Portfolio Tracker

Allows users to add their cryptocurrency holdings, track their investments, and see the total value of their portfolio in real-time.

Here's a revised **Feasibility Study** section tailored to your **Crypto Tracker** project:

## 3.Authentication

We will use ``Firebase Cloud Firestore`` rather than other relational database by the need for **real-time synchronization**, **scalability**, and **flexibility** with a schema-less data structure. It also simplifies development with its built-in features like **serverless architecture**, **authentication integration**, and **offline support**.

### 3.1 Firebase Firestore: Advs

When choosing Firebase Firestore over a relational database like MySQL for our project, the decision is driven by several key advantages. First, Firestore’s **flexible NoSQL structure** allows us to adapt easily as our data evolves, without the rigid schemas and complex migrations MySQL requires. Second, it offers **real-time synchronization**, so our users can see cryptocurrency price changes and portfolio updates instantly, without the need for additional infrastructure. Firestore is also **serverless**, meaning it scales automatically without us worrying about managing databases or performance tuning as we grow. Moreover, it seamlessly integrates with **Firebase Authentication**, simplifying user management and security. Finally, Firestore’s **offline support** lets our users access and update data even when they're disconnected, with automatic syncing when they're back online. In short, Firestore provides a scalable, flexible, and real-time solution that is ideal for modern applications focus on real-time like ours.

### 3.2 Database Schema

Since Firestore is a NoSQL database and essentially stores documents in collections, I structure the schema in following way.

We have two main collections: **Users** and **Portfolios**. The **Users** collection will store user-related data, while the **Portfolios** collection will handle the user’s cryptocurrency holdings and transactions. Let's break it down:

#### 3.2.1. Users Collection

The `Users` collection stores user-specific data, primarily managing the user's profile and preferences. The document ID for each user corresponds to their Firebase Authentication UID.

**Fields**:

- **userId**: The Firebase Authentication UID that uniquely identifies the user.
- **email**: The user's email address.
- **firstName**: The user's first name.
- **lastName**: The user's last name.
- **createdAt**: Timestamp indicating when the user account was created.
- **preferences**: An object storing user-specific settings such as:
  - **currency**: The user's preferred currency (e.g., USD, EUR).
  - **theme**: The user's theme preference (e.g., light, dark, auto).

**Example Document:**

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

#### 3.2.2 Portfolios Collection

Each user has a corresponding portfolio document within the `Portfolios` collection, which tracks their cryptocurrency holdings and related financial data.

**Portfolio Fields:**

- **portfolioId**: Unique ID for the portfolio.
- **userId**: The ID of the user owning the portfolio, serving as a reference to the `Users` collection.
- **cryptocurrency**: An array that stores information about the user's current holdings, including:
  - **symbol**: Cryptocurrency symbol (e.g., BTC, ETH).
  - **amount**: The total quantity of the cryptocurrency owned.
  - **averagePrice**: The average price the user paid for the cryptocurrency (based on their buy transactions).
- **revenue**: The total revenue for the portfolio (calculated based on transactions).

**Example Portfolio Document:**

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

#### 3.2.3 Transactions Sub-Collection

Each portfolio document contains a **Transactions** sub-collection to track the user's **buy**, **sell**, and **transfer** activities. Every transaction is stored as a separate document inside this sub-collection.

**Transaction Fields:**

- **transactionId**: Auto-generated unique identifier for each transaction.
- **type**: The type of transaction (e.g., "buy", "sell", or "transfer").
- **cryptocurrency**: The cryptocurrency involved in the transaction (symbol).
- **amount**: The amount of cryptocurrency transacted.
- **price**: The price per unit at the time of the transaction (for buy/sell transactions).
- **transferType**: For transfers, indicates whether it's a transfer "in" (deposit) or "out" (withdrawal).
- **date**: Timestamp representing the date and time of the transaction.
- **fees**: (Optional) Transaction fees associated with the trade or transfer.
- **notes**: (Optional) Additional notes about the transaction.

**Example Transactions:**

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



#### 3.2.4 Summary

- **Users Collection** stores basic user information and preferences.

- **Portfolios Collection** stores each user’s cryptocurrency holdings and revenue.

- **Transactions Sub-Collection** within each portfolio tracks individual transactions (buy, sell, transfer) with relevant details such as amounts, dates, and fees.

  

## 4. Feasibility Study

### 4.1 Technological (T)

- **Front-end:** React is used to build an intuitive user interface, providing a smooth and responsive user experience. This choice supports faster learning and deeper understanding of front-end development for future enhancement.
- **Back-end:** The back-end is built with Node.js, using Express as the primary framework. This setup facilitates efficient API communication with the CoinGecko API for cryptocurrency data, ensuring quick data retrieval and processing.
- **Database:** Firebase serves as the primary database for user authentication and account management, providing secure storage and reliable user data handling.
- **Data API:** CoinGecko API provides all the necessary cryptocurrency market data, including real-time prices, historical data, and market trends. This API is freely accessible and well-documented, ideal for learning integration and data processing.
- **Caching:** Redis can be used for caching frequently accessed cryptocurrency data, enhancing performance by reducing repeated API calls and allowing faster retrieval for popular queries.
- **Containerization:** Docker may be used to deploy the application in a containerized environment, ensuring consistent operation across different systems and simplifying the deployment process.

### 4.2 Economic (E)

- **Hosting Costs:** Estimated cloud hosting costs for the Crypto Tracker application are around $50 to $100 per month, depending on user demand and storage needs.
- **Database Costs:** Firebase provides flexible pricing for authentication and database services, with a free tier for small-scale applications and affordable rates for additional usage.
- **API Usage:** CoinGecko API is free for non-commercial use, making it an economical choice for tracking cryptocurrency data.
- **Software:** React, Node.js, and Firebase are all open-source or free to use, which reduces development costs as there are no software licensing fees.
- **Additional Costs:** Domain registration and SSL certificate fees may apply, generally around $30 annually for domain registration, with free SSL options available.

### 4.3 Legal (L)

- **Data Privacy & Protection:** The system will comply with GDPR and CCPA for user data protection, using encryption for sensitive information, especially around user accounts and portfolios.
- **API Usage Agreement:** CoinGecko's API Terms of Use will be followed to ensure lawful data usage.
- **Licensing Compliance:** All open-source software, such as React, Node.js, and Firebase, will be used in compliance with their respective licenses.
- **Intellectual Property:** Proper measures will be in place to protect custom-developed code, UI elements, and trademarks associated with Crypto Tracker.

### 4.4 Operational (O)

- **Scalability:** The app is designed to be scalable, leveraging cloud services and Firebase's flexibility to handle increasing numbers of users as interest in cryptocurrency grows.
- **Maintenance:** Regular maintenance will ensure the app’s performance, security, and functionality, including updates to dependencies and frameworks.
- **Backup & Disaster Recovery:** Firebase and cloud services provide built-in backup options to maintain data integrity and recovery in case of unexpected downtime.
- **Training:** Basic tutorials and documentation will be available to help users understand the app's features and use it effectively.
- **Security:** Regular security reviews and best practices in coding, authentication, and data handling will be implemented to maintain system security.

Here’s the revised **4.5 Schedule (S)** section based on the new timeline you provided:

---

### 4.5 Schedule (S)

- **Week 1: Setup**
  - **Task 0:** Team formation (4-5 students) to allocate roles and responsibilities.
  - **Task 1:** Explore the CoinGecko API and understand available endpoints for retrieving cryptocurrency data.

- **Week 2: Initial Development**
  - **Task 2:** Set up the development environment, ensuring all necessary tools and libraries are installed.
  - **Task 3:** Design the database schema and set up the back-end framework, preparing for integration with the CoinGecko API.

- **Weeks 3-4: Backend Development**
  - **Task 4:** Implement cryptocurrency price search and market data retrieval functionality using the CoinGecko API, ensuring accurate and real-time data access.

- **Week 5: Mid-Checkpoint Presentation**
  - Present the backend development progress, API data fetching strategy, and the implementation of the search functionality to gather feedback and make improvements.

- **Week 6: Frontend Development and Integration**
  - **Task 5:** Develop the frontend structure and integrate price and market data displays, creating a user-friendly interface.
  - **Task 6:** Implement member profiles and integrate Firebase authentication for secure user account management.

- **Week 7: Finalizing Features and Testing**
  - **Task 7:** Complete the news feed feature to display the latest cryptocurrency-related news and add historical data charts for detailed market analysis.

- **Week 8: Deployment and Final Presentation Preparation**
  - **Task 8:** Deploy the application to a live environment, ensuring it is accessible and functional for end-users.
  - **Task 9:** Conduct final testing and prepare for the presentation, addressing any issues and refining the app based on user feedback.

- **Week 9: Final Presentation**
  - Present the completed Crypto Tracker application, showcasing all features, functionalities, and the user experience to demonstrate its effectiveness in tracking cryptocurrency trends and data.

## 5.Modeling

### 5.1 Use Case Diagram

![Snipaste_2024-10-29_07-23-07](C:\Users\yolanda\Desktop\md文件\md文件照片\Snipaste_2024-10-29_07-23-07.png)

### 5.2 ER Diagram

![Snipaste_2024-10-29_07-21-18](C:\Users\yolanda\Desktop\md文件\md文件照片\Snipaste_2024-10-29_07-21-18.png)

## 6.Conclusion

In summary, Crypto Tracker aims to be a helpful tool for anyone interested in tracking cryptocurrency markets in a simple and accessible way. By combining real-time data from the CoinGecko API with Firebase’s efficient database and authentication features, we provide users with a platform that’s not only easy to use but also highly responsive. From tracking price changes and viewing market trends to managing personal portfolios, the app covers a range of essential features for cryptocurrency enthusiasts. 

 Overall, Crypto Tracker is a practical project that has helped us apply various technical skills, and we hope it will offer valuable insights for users as they explore the ever-evolving world of cryptocurrency.