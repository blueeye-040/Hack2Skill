# Namma Area – System Design Specification

## Design Overview

This document outlines the technical design for Namma Area, a hyperlocal AI-powered digital commerce platform. The system follows a three-tier architecture with a dedicated AI microservices layer to ensure scalability, maintainability, and optimal performance.

## Architecture Design

### System Architecture

The system is built on a Three-Tier Architecture with an additional AI Microservices Layer:

1. Presentation Layer - React PWA (Progressive Web App)
2. Business Logic Layer - Spring Boot REST API
3. Data Persistence Layer - MySQL Database
4. AI Microservices Layer - Python FastAPI

This separation ensures:
- Independent scaling of AI services
- Low latency for core business operations
- Maintainability through modular design
- Technology-specific optimization (Java for business logic, Python for AI)

### Data Flow Architecture

```
Client (React PWA)
    ↓ HTTPS/REST
Spring Boot Backend (REST API)
    ↓ JDBC
MySQL Database
    ↓ HTTP/REST
AI Microservices (Python FastAPI)
    ↓ Response
Spring Boot Backend
    ↓ JSON
Client (React PWA)
```

## Component Design Specifications

### Frontend Design (React + PWA)

#### Technology Stack
- React 18+ for UI components
- Redux Toolkit for state management
- Service Workers for offline functionality
- React Router for navigation
- Axios for API communication

#### Key Modules

1. Vendor Admin Portal
   - Dashboard with order analytics
   - Product management interface
   - Order fulfillment workflow
   - Profile and settings management

2. Customer Marketplace Interface
   - Shop discovery with map view
   - Product browsing and search
   - Shopping cart and checkout
   - Order tracking interface
   - Review and rating system

3. Offline Support
   - Service Workers cache static assets
   - IndexedDB for offline data storage
   - Background sync for pending actions
   - Offline indicator UI

4. State Management (Redux)
   - Auth state (user, token, role)
   - Cart state (items, quantities, totals)
   - Shop state (nearby shops, products)
   - Order state (active orders, history)

#### Design Decisions
- PWA for mobile-first experience without app store dependency
- Redux for predictable state management across complex workflows
- Service Workers for offline capability in low-connectivity areas
- Responsive design with mobile breakpoints (320px, 768px, 1024px)

### Backend Design (Spring Boot)

#### Technology Stack
- Spring Boot 3.x
- Spring Security with JWT
- Spring Data JPA
- MySQL Connector
- RestTemplate for AI service communication

#### Controller Layer

1. AuthController
   - POST /api/auth/register - User registration
   - POST /api/auth/login - JWT token generation
   - POST /api/auth/refresh - Token refresh
   - GET /api/auth/profile - User profile

2. InventoryController
   - GET /api/products - List products (with filters)
   - POST /api/products - Create product (vendor only)
   - PUT /api/products/{id} - Update product
   - DELETE /api/products/{id} - Delete product
   - POST /api/products/upload - Upload product image (triggers AI)

3. OrderController
   - POST /api/orders - Create order
   - GET /api/orders - List orders (role-based)
   - PUT /api/orders/{id}/status - Update order status
   - GET /api/orders/{id}/track - Track order
   - POST /api/orders/{id}/review - Submit review

4. GeospatialController
   - GET /api/shops/nearby - Find shops within radius
   - GET /api/shops/{id}/location - Get shop location
   - POST /api/delivery/estimate - Calculate delivery ETA

#### Security Layer

1. JWT Authentication
   - Stateless token-based authentication
   - Token expiry: 24 hours
   - Refresh token: 7 days
   - HS256 signing algorithm

2. Role-Based Access Control (RBAC)
   - Roles: CUSTOMER, VENDOR, ADMIN
   - Method-level security with @PreAuthorize
   - Resource ownership validation

#### Service Layer Design Patterns
- Service interfaces for business logic abstraction
- DTO pattern for data transfer
- Repository pattern for data access
- Exception handling with @ControllerAdvice

### Database Design (MySQL)

#### Schema Design

1. Users Table
```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    role ENUM('CUSTOMER', 'VENDOR', 'ADMIN'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

2. Shops Table
```sql
CREATE TABLE shops (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    owner_id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    latitude DECIMAL(10,6) NOT NULL,
    longitude DECIMAL(10,6) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    operating_hours JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (owner_id) REFERENCES users(id)
);
```

3. Products Table
```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    shop_id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    image_url VARCHAR(500),
    category VARCHAR(100),
    is_available BOOLEAN DEFAULT TRUE,
    ai_metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (shop_id) REFERENCES shops(id)
);
```

4. Orders Table
```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    customer_id BIGINT NOT NULL,
    shop_id BIGINT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    status ENUM('PENDING', 'CONFIRMED', 'PREPARING', 'OUT_FOR_DELIVERY', 'DELIVERED', 'CANCELLED'),
    delivery_address TEXT,
    delivery_latitude DECIMAL(10,6),
    delivery_longitude DECIMAL(10,6),
    estimated_delivery_time TIMESTAMP,
    otp VARCHAR(6),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES users(id),
    FOREIGN KEY (shop_id) REFERENCES shops(id)
);
```

5. OrderItems Table
```sql
CREATE TABLE order_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

6. Reviews Table
```sql
CREATE TABLE reviews (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    shop_id BIGINT NOT NULL,
    rating INT CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    sentiment_score DECIMAL(3,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (product_id) REFERENCES products(id),
    FOREIGN KEY (shop_id) REFERENCES shops(id)
);
```

7. ShopAnalytics Table
```sql
CREATE TABLE shop_analytics (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    shop_id BIGINT NOT NULL,
    total_orders INT DEFAULT 0,
    average_rating DECIMAL(3,2),
    sentiment_score DECIMAL(3,2),
    trust_score DECIMAL(3,2),
    order_frequency INT DEFAULT 0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (shop_id) REFERENCES shops(id)
);
```

#### Indexing Strategy
- Index on (latitude, longitude) for geospatial queries
- Index on user_id, shop_id for foreign key lookups
- Composite index on (shop_id, is_available) for product queries
- Index on created_at for time-based queries

#### Design Decisions
- DECIMAL for monetary values to avoid floating-point errors
- JSON fields for flexible metadata storage
- ENUM for status fields to enforce valid states
- Timestamps for audit trails
- Geospatial fields as DECIMAL(10,6) for precision

## AI System Design

### AI Microservices Architecture

Technology Stack:
- Python 3.10+
- FastAPI for REST endpoints
- Tesseract OCR
- TensorFlow/Keras for MobileNetV2
- NLTK with VADER
- NumPy/SciPy for geospatial calculations
- scikit-learn for DBSCAN clustering

### 4.1 Hybrid Vision Pipeline

#### Design Overview
Combines OCR and image classification for robust product recognition.

#### Implementation Flow

1. Image Preprocessing
   - Resize to 224x224 for MobileNetV2
   - Grayscale conversion for OCR
   - Noise reduction

2. OCR Processing (Tesseract)
   - Extract text from product packaging
   - Clean and normalize extracted text
   - Fuzzy match against product database
   - Confidence threshold: 70%

3. Object Classification (MobileNetV2)
   - Pre-trained on ImageNet
   - Fine-tuned on product categories
   - Returns top-3 predictions with confidence
   - Confidence threshold: 60%

4. Result Fusion
   - Combine OCR and classification results
   - Weighted scoring: OCR (60%), Classification (40%)
   - Return best match with metadata

#### API Endpoint
```
POST /ai/vision/recognize
Request: { "image": "base64_encoded_image" }
Response: {
    "product_name": "string",
    "category": "string",
    "confidence": 0.85,
    "ocr_text": "string",
    "classification_label": "string"
}
```

#### Design Decisions
- Hybrid approach for better accuracy in diverse scenarios
- MobileNetV2 for efficiency on standard hardware
- Tesseract for zero-cost OCR solution
- Confidence thresholds to prevent false positives

### 4.2 Sentiment Analysis Pipeline

#### Design Overview
Analyzes customer reviews to generate trust scores for vendors.

#### Implementation Flow

1. Review Collection
   - Fetch reviews from MySQL database
   - Filter by shop_id and date range

2. Sentiment Analysis (VADER)
   - Process review text
   - Generate compound sentiment score (-1 to +1)
   - Store sentiment_score in reviews table

3. Trust Score Calculation
   - Formula: S_trust = w1(R_avg) + w2(S_nlp) + w3(O_freq)
   - R_avg: Average rating (normalized 0-1)
   - S_nlp: Average sentiment score (normalized 0-1)
   - O_freq: Order frequency (normalized 0-1)
   - Weights: w1=0.4, w2=0.3, w3=0.3

4. Analytics Update
   - Update shop_analytics table
   - Recalculate on new review submission
   - Cache results for 1 hour

#### API Endpoint
```
POST /ai/sentiment/analyze
Request: { "shop_id": 123 }
Response: {
    "trust_score": 0.82,
    "average_rating": 4.2,
    "sentiment_score": 0.75,
    "order_frequency": 45,
    "total_reviews": 120
}
```

#### Design Decisions
- VADER for rule-based sentiment (no training required)
- Weighted formula balances multiple trust signals
- Transparent scoring for responsible AI
- Periodic recalculation to reflect recent performance

### 4.3 Geospatial Discovery

#### Design Overview
Finds nearby shops using distance calculation and clustering.

#### Implementation Flow

1. Haversine Distance Calculation
   - Calculate distance between customer and shops
   - Formula: d = 2r × arcsin(√(sin²(Δφ/2) + cos(φ1)cos(φ2)sin²(Δλ/2)))
   - Earth radius: 6371 km

2. Radius Filtering
   - Filter shops within 3 km radius
   - Sort by distance (ascending)

3. DBSCAN Clustering (Optional)
   - Cluster nearby delivery addresses
   - Epsilon: 0.5 km
   - Min samples: 3
   - Optimize delivery routes

4. Result Ranking
   - Primary: Distance
   - Secondary: Trust score
   - Tertiary: Availability

#### API Endpoint
```
POST /ai/geospatial/nearby
Request: {
    "latitude": 12.9716,
    "longitude": 77.5946,
    "radius_km": 3
}
Response: {
    "shops": [
        {
            "shop_id": 123,
            "name": "Shop Name",
            "distance_km": 1.2,
            "trust_score": 0.85,
            "is_available": true
        }
    ]
}
```

#### Design Decisions
- Haversine for accurate distance on Earth's surface
- 3 km radius balances hyperlocal focus with shop availability
- DBSCAN for address clustering in unstructured environments
- Trust score as secondary ranking factor

### 4.4 Dynamic Delivery Time Estimation (DDTE)

#### Design Overview
Estimates delivery time based on multiple factors.

#### Implementation Flow

1. Preparation Time (T_prep)
   - Base time: 15 minutes
   - Additional: 2 minutes per item
   - Max: 45 minutes

2. Travel Time Calculation
   - Distance (d) from Haversine
   - Average speed (V_avg): 20 km/h (urban)
   - Travel time: d / V_avg

3. Congestion Multiplier (α)
   - Peak hours (12-2 PM, 7-9 PM): α = 1.5
   - Normal hours: α = 1.0
   - Late night: α = 0.8

4. Buffer Time (T_buffer)
   - Fixed buffer: 5 minutes
   - Accounts for uncertainties

5. Final Estimation
   - Formula: T_est = T_prep + (d / V_avg) × α + T_buffer

#### API Endpoint
```
POST /ai/delivery/estimate
Request: {
    "shop_latitude": 12.9716,
    "shop_longitude": 77.5946,
    "customer_latitude": 12.9800,
    "customer_longitude": 77.6000,
    "order_items_count": 5,
    "current_time": "2024-01-15T19:30:00Z"
}
Response: {
    "estimated_minutes": 35,
    "preparation_time": 25,
    "travel_time": 8,
    "buffer_time": 5,
    "estimated_delivery_time": "2024-01-15T20:05:00Z"
}
```

#### Design Decisions
- Multi-factor estimation for accuracy
- Time-based congestion multiplier
- Buffer time for reliability
- Transparent breakdown for customer trust

## Responsible AI Considerations

### Transparency
- All AI decisions are explainable
- Trust score formula is documented and visible
- Confidence scores displayed to users
- Human override capability for vendors

### Fairness
- No hidden bias in vendor visibility
- Equal opportunity listing within locality
- Geographic proximity as primary ranking factor
- Trust score based on objective metrics

### Privacy
- No personal data used in AI training
- Location data used only for distance calculation
- Review sentiment analysis on aggregated data
- User consent for location tracking

### Accountability
- Audit logs for AI decisions
- Regular bias testing on classification models
- Feedback mechanism for incorrect predictions
- Manual review process for disputed rankings

## Scalability Design

### Horizontal Scaling Strategy
- Spring Boot API: Multiple instances behind load balancer
- AI Microservices: Independent scaling based on demand
- Database: Read replicas for query distribution
- Caching: Redis for frequently accessed data

### Cloud Deployment (AWS EC2)
- Application servers: t3.medium instances
- AI services: t3.large instances (CPU-optimized)
- Database: RDS MySQL with Multi-AZ
- Load balancer: Application Load Balancer (ALB)
- Storage: S3 for product images

### Caching Strategy
- Shop data: 1-hour TTL
- Product listings: 30-minute TTL
- Trust scores: 1-hour TTL
- User sessions: Redis with 24-hour expiry

### Performance Optimization
- Database connection pooling (HikariCP)
- Lazy loading for large datasets
- Pagination for list endpoints (20 items per page)
- Image compression and CDN delivery
- API response compression (gzip)

## Future Enhancements

### Phase 2 Features
1. Voice-Based Vendor Interaction
   - Automatic Speech Recognition (ASR)
   - Voice commands for order management
   - Regional language support

2. Predictive Demand Analytics
   - LSTM models for demand forecasting
   - Inventory optimization recommendations
   - Seasonal trend analysis

3. Blockchain-Based Delegated Pickup
   - Immutable pickup ledger
   - Smart contract for OTP verification
   - Transparent audit trail

### Technical Debt Considerations
- Migrate to microservices architecture if scale exceeds 10,000 vendors
- Implement GraphQL for flexible client queries
- Add real-time features with WebSockets
- Implement advanced monitoring with Prometheus/Grafana

## Implementation Tasks

### Phase 1: Core Infrastructure (Weeks 1-2)
- [ ] Set up Spring Boot project structure
- [ ] Configure MySQL database and create schema
- [ ] Implement JWT authentication
- [ ] Set up React PWA project
- [ ] Configure Redux store
- [ ] Implement Service Workers for offline support

### Phase 2: Vendor Module (Weeks 3-4)
- [ ] Implement vendor registration and login
- [ ] Create mini-website generation logic
- [ ] Build product management CRUD APIs
- [ ] Develop vendor dashboard UI
- [ ] Implement order management backend
- [ ] Create order fulfillment UI

### Phase 3: Customer Module (Weeks 5-6)
- [ ] Implement customer registration and login
- [ ] Build shop discovery API with geospatial filtering
- [ ] Create marketplace UI with map view
- [ ] Implement shopping cart functionality
- [ ] Build checkout and order placement flow
- [ ] Create order tracking interface

### Phase 4: AI Services (Weeks 7-8)
- [ ] Set up Python FastAPI project
- [ ] Implement OCR with Tesseract
- [ ] Integrate MobileNetV2 for classification
- [ ] Build hybrid vision pipeline
- [ ] Implement VADER sentiment analysis
- [ ] Create trust score calculation logic
- [ ] Implement Haversine distance calculation
- [ ] Build DBSCAN clustering for addresses
- [ ] Create delivery time estimation algorithm

### Phase 5: Integration & Testing (Weeks 9-10)
- [ ] Integrate AI services with Spring Boot backend
- [ ] Implement review and rating system
- [ ] Build delegated pickup with OTP
- [ ] Create shop analytics dashboard
- [ ] Perform end-to-end testing
- [ ] Conduct usability testing (target SUS ≥85)
- [ ] Performance testing and optimization
- [ ] Security audit and penetration testing

### Phase 6: Deployment (Week 11)
- [ ] Set up AWS EC2 instances
- [ ] Configure load balancer
- [ ] Deploy Spring Boot application
- [ ] Deploy AI microservices
- [ ] Set up RDS MySQL database
- [ ] Configure S3 for image storage
- [ ] Set up monitoring and logging
- [ ] Perform production smoke tests

### Phase 7: Launch & Monitoring (Week 12)
- [ ] Soft launch with pilot vendors
- [ ] Monitor system performance
- [ ] Collect user feedback
- [ ] Fix critical bugs
- [ ] Optimize based on real-world usage
- [ ] Full public launch
