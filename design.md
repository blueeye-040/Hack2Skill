# Namma Area – System Design Document

## 1. System Architecture Overview

The system follows a Three-Tier Architecture:

1. Presentation Layer (React + PWA)
2. Business Logic Layer (Spring Boot REST APIs)
3. Data Persistence Layer (MySQL)
4. AI Microservices Layer (Python FastAPI)

AI services are decoupled to ensure scalability and low latency.

---

## 2. Architecture Diagram (Logical)

Client (React PWA)
        ↓
Spring Boot Backend (REST API)
        ↓
MySQL Database
        ↓
AI Microservices (Python - Vision & NLP)

---

## 3. Component Design

### 3.1 Frontend (React + PWA)
- Vendor Admin Portal
- Customer Marketplace Interface
- Offline caching via Service Workers
- Redux for state management

---

### 3.2 Backend (Spring Boot)
Controllers:
- AuthController
- InventoryController
- OrderController
- GeospatialController

Security:
- JWT-based stateless authentication
- Role-based access control

---

### 3.3 Database Design (MySQL)

Key Tables:
- Users
- Shops
- Products
- Orders
- OrderItems
- Reviews
- ShopAnalytics

Geospatial fields:
- latitude (DECIMAL 10,6)
- longitude (DECIMAL 10,6)

---

## 4. AI System Design

### 4.1 Hybrid Vision Pipeline

Step 1: OCR using Tesseract
- Extract product name
- Fuzzy match with database

Step 2: Object Classification (MobileNetV2)
- For unbranded items
- Returns product label + confidence

---

### 4.2 Sentiment Analysis Pipeline

- Reviews fetched from DB
- Processed using VADER
- Polarity score stored in ShopAnalytics
- Used in Trust Score ranking formula

Trust Score Formula:
Strust = w1(Ravg) + w2(Snlp) + w3(Ofreq)

---

### 4.3 Geospatial Discovery

- Haversine formula for distance calculation
- Radius filtering (≤ 3 km)
- DBSCAN clustering for address correction

---

### 4.4 Dynamic Delivery Time Estimation

Test = Tprep + (d / Vavg) × α + Tbuffer

Factors:
- Preparation time
- Distance
- Congestion multiplier
- Buffer margin

---

## 5. Responsible AI Considerations

- Transparent ranking logic
- No hidden bias in vendor visibility
- Equal opportunity listing within locality
- Human override capability
- Clear communication of AI decisions

---

## 6. Scalability Considerations

- AI services deployed independently
- Cloud hosting (AWS EC2)
- Horizontal scaling for microservices
- Caching for frequently accessed shops

---

## 7. Future Enhancements

- Voice-based vendor interaction (ASR)
- Predictive demand analytics (LSTM)
- Blockchain-based DPP ledger
