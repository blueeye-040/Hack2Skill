# Namma Area – Requirements Document

## Project Overview

Namma Area is a hyperlocal AI-powered digital commerce platform designed to help small retail shops and mobile vendors transition into online business. The system enables vendors to create mini-websites, manage products, and serve customers within a 2–3 km radius while maintaining ownership of their brand and customer relationships.

The platform leverages AI to reduce digital literacy barriers, improve trust, and optimize hyperlocal logistics.

## Problem Statement

Small and micro retail businesses struggle to adopt digital commerce due to:
- Limited technical knowledge
- High onboarding complexity
- Commission-based aggregator dependency
- Poor delivery accuracy in unstructured address environments

AI is required to:
- Reduce manual data entry effort
- Improve trust through intelligent review analysis
- Optimize hyperlocal discovery and delivery accuracy
- Enable data-driven decision making for vendors

## User Stories

### Vendor User Stories

#### US-V1: Vendor Registration and Authentication
As a small retail vendor, I want to register and securely log in to the platform so that I can start selling my products online.

Acceptance Criteria:
- Vendor can register with email, phone, and shop details
- JWT-based authentication is implemented
- Vendor receives confirmation upon successful registration
- Login session persists securely across devices

#### US-V2: Mini-Website Generation
As a vendor, I want an automatically generated mini-website for my shop so that customers can discover and browse my products without me needing technical skills.

Acceptance Criteria:
- Mini-website is auto-generated upon vendor registration
- Website displays shop name, location, and product catalog
- Vendor can customize basic branding (logo, colors)
- Website is mobile-responsive and PWA-compatible

#### US-V3: Product Management
As a vendor, I want to add, edit, and delete products easily so that I can keep my inventory up to date.

Acceptance Criteria:
- Vendor can add products with name, price, description, and image
- Vendor can edit existing product details
- Vendor can delete products from inventory
- Changes reflect immediately on the mini-website

#### US-V4: AI-Assisted Product Recognition
As a vendor with limited typing skills, I want to upload product images and have the system automatically extract product information so that I can onboard products faster.

Acceptance Criteria:
- Vendor can upload product images
- OCR extracts text from product packaging
- MobileNetV2 classifies product category
- Extracted data pre-fills product form fields
- Vendor can review and edit AI-generated data before saving

#### US-V5: Order Management Dashboard
As a vendor, I want to view and manage incoming orders in a dashboard so that I can fulfill customer requests efficiently.

Acceptance Criteria:
- Dashboard displays all orders with status (pending, confirmed, delivered)
- Vendor can update order status
- Vendor can view customer details and delivery address
- Dashboard shows order history and analytics

#### US-V6: Delivery Status Management
As a vendor, I want to update delivery status in real-time so that customers are informed about their order progress.

Acceptance Criteria:
- Vendor can mark orders as "preparing", "out for delivery", "delivered"
- Status updates trigger customer notifications
- Delivery time estimates are displayed
- Vendor can add delivery notes

#### US-V7: Delegated Pickup with OTP
As a vendor, I want to verify delegated pickups using OTP so that orders are securely handed to authorized persons.

Acceptance Criteria:
- Customer can authorize a delegate with OTP
- Vendor receives OTP for verification
- Order is marked complete only after OTP validation
- System logs pickup details for audit

### Customer User Stories

#### US-C1: Shop Discovery within Radius
As a customer, I want to discover shops within 2–3 km of my location so that I can buy from nearby vendors.

Acceptance Criteria:
- Customer can enable location services
- System displays shops within 2–3 km radius
- Shops are sorted by distance and trust score
- Map view shows vendor locations

#### US-C2: Real-Time Vendor Availability
As a customer, I want to see which vendors are currently available so that I don't order from closed shops.

Acceptance Criteria:
- Vendor availability status is displayed (open/closed)
- Vendors can set operating hours
- System shows "currently unavailable" for closed shops
- Customers can filter by available vendors only

#### US-C3: Add to Cart and Checkout
As a customer, I want to add products to cart and complete checkout so that I can place orders easily.

Acceptance Criteria:
- Customer can add multiple products to cart
- Cart displays total price and item count
- Customer can modify quantities or remove items
- Checkout captures delivery address and contact details
- Order confirmation is displayed after successful checkout

#### US-C4: Live Tracking for Moving Vendors
As a customer, I want to track moving vendors in real-time so that I know when they'll arrive at my location.

Acceptance Criteria:
- Customer can view vendor's live location on map
- Estimated arrival time is displayed
- System updates location every 30 seconds
- Customer receives notification when vendor is nearby

#### US-C5: Review and Rating System
As a customer, I want to leave reviews and ratings for vendors so that I can share my experience with others.

Acceptance Criteria:
- Customer can rate vendors (1-5 stars)
- Customer can write text reviews
- Reviews are visible on vendor's mini-website
- Customer can edit or delete their own reviews

#### US-C6: Delegated Pickup Authorization
As a customer, I want to authorize someone else to pick up my order so that I have flexibility in receiving deliveries.

Acceptance Criteria:
- Customer can generate OTP for delegated pickup
- OTP is valid for single use
- Customer can share OTP with delegate
- System notifies customer when order is picked up

### AI Feature User Stories

#### US-AI1: Vision-Based Product Recognition
As the system, I need to recognize products from images using OCR and classification so that vendors can onboard products with minimal effort.

Acceptance Criteria:
- OCR (Tesseract) extracts text from product images
- MobileNetV2 classifies product into categories
- System combines OCR and classification results
- Accuracy rate ≥75% for common product types
- Processing time <3 seconds per image

#### US-AI2: Sentiment Analysis for Trust Scoring
As the system, I need to analyze review sentiment using NLP so that vendor trust scores reflect customer satisfaction accurately.

Acceptance Criteria:
- VADER sentiment analysis processes review text
- Sentiment scores range from -1 (negative) to +1 (positive)
- Trust score formula incorporates sentiment, rating, and review count
- Trust scores update automatically when new reviews are added
- Transparent scoring logic is documented

#### US-AI3: Geospatial Clustering for Address Accuracy
As the system, I need to cluster nearby addresses using geospatial algorithms so that delivery accuracy improves in unstructured address environments.

Acceptance Criteria:
- Haversine formula calculates distances between coordinates
- DBSCAN clusters nearby delivery points
- System suggests clustered delivery zones to vendors
- Address accuracy improves by ≥40% compared to manual entry

#### US-AI4: Dynamic Delivery Time Estimation
As the system, I need to estimate delivery times dynamically based on distance, traffic, and preparation time so that customers receive accurate ETAs.

Acceptance Criteria:
- System calculates preparation time based on order size
- Distance is computed using Haversine formula
- Congestion multiplier adjusts for traffic conditions
- Buffer time accounts for uncertainties
- ETA accuracy within ±10 minutes for 80% of orders

## Technical Requirements

### Architecture
- Three-tier architecture: Presentation, Business Logic, Data Layer
- AI Microservices Layer (Python FastAPI) for AI processing
- React PWA for frontend (mobile-first, offline-capable)
- Spring Boot REST API for backend
- MySQL database with geospatial support

### Authentication & Security
- JWT-based authentication
- Role-based access control (RBAC)
- Secure password hashing
- HTTPS/TLS encryption
- Input validation and sanitization

### Performance
- API response time <2 seconds for core operations
- AI processing time <3 seconds per request
- Support for 1000+ concurrent users
- PWA loads in <3 seconds on 3G networks

### Scalability
- Horizontal scaling for API and AI services
- Database connection pooling
- Caching for frequently accessed data
- Load balancing for high availability

### Mobile & Offline Support
- Progressive Web App (PWA) architecture
- Service workers for offline caching
- Optimized for low-bandwidth (2G/3G)
- Works on devices with 2GB RAM
- Responsive design for all screen sizes

### Data Privacy
- GDPR-compliant data handling
- User consent for location tracking
- Secure storage of personal information
- Data encryption at rest and in transit

## Non-Functional Requirements

### Usability
- Target System Usability Scale (SUS) score ≥85
- Intuitive UI requiring minimal training
- Support for regional languages
- Accessible design following WCAG guidelines

### Reliability
- System uptime ≥99.5%
- Automated backup and recovery
- Error handling with user-friendly messages
- Graceful degradation when AI services are unavailable

### Maintainability
- Modular architecture for easy updates
- Comprehensive API documentation
- Automated testing (unit, integration, E2E)
- Logging and monitoring for debugging

### Responsible AI
- Transparent ranking and scoring algorithms
- Bias detection in product classification
- User control over AI-generated data
- Explainable AI decisions

## Constraints

### Technical Constraints
- Designed for low-resource devices (2GB RAM smartphones)
- Must work with intermittent internet connectivity
- Limited to 2–3 km hyperlocal radius
- AI models must run efficiently on standard cloud infrastructure

### User Constraints
- Vendors have minimal technical knowledge
- Customers may have limited digital literacy
- Unstructured address formats in target regions
- Variable network quality across service areas

### Business Constraints
- No commission-based model (vendor ownership priority)
- Must be cost-effective for small vendors
- Scalable without significant infrastructure investment

## Success Metrics

### Vendor Metrics
- ≥70% reduction in product onboarding time
- ≥15% increase in vendor order volume
- ≥80% vendor retention after 3 months
- Average vendor rating ≥4.0/5.0

### Customer Metrics
- ≥40% reduction in delivery errors
- ≥85% customer satisfaction score
- ≥60% repeat purchase rate
- Average order fulfillment time <30 minutes

### System Metrics
- System Usability Scale (SUS) score ≥85
- AI product recognition accuracy ≥75%
- Delivery time estimation accuracy ±10 minutes
- API response time <2 seconds (95th percentile)

### Business Metrics
- 100+ active vendors within 6 months
- 1000+ active customers within 6 months
- ≥70% month-over-month growth in transactions
- Platform uptime ≥99.5%
