# Namma Area – Requirements Document

## 1. Project Overview

Namma Area is a hyperlocal AI-powered digital commerce platform designed to help small retail shops and mobile vendors transition into online business. The system enables vendors to create mini-websites, manage products, and serve customers within a 2–3 km radius while maintaining ownership of their brand and customer relationships.

The platform leverages AI to reduce digital literacy barriers, improve trust, and optimize hyperlocal logistics.

---

## 2. Problem Statement

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

---

## 3. Functional Requirements

### 3.1 Vendor Module
- Vendor registration and authentication (JWT-based)
- Mini-website generation for each vendor
- Product management (Add, Edit, Delete)
- AI-assisted product recognition (Vision model)
- Order management dashboard
- Delivery status management
- Delegated Pickup (OTP verification system)

### 3.2 Customer Module
- Shop discovery within 2–3 km radius
- Real-time vendor availability
- Add to cart and checkout
- Live tracking for moving vendors
- Review and rating system
- Delegated pickup authorization

### 3.3 AI Features
- Hybrid Vision-based product recognition (OCR + MobileNetV2)
- Sentiment analysis using NLP (VADER)
- Trust-based ranking algorithm
- Geospatial clustering for address accuracy
- Dynamic Delivery Time Estimation (DDTE)

---

## 4. Non-Functional Requirements

- Scalability for multiple vendors
- Mobile-first and low-bandwidth support (PWA)
- Secure authentication and data privacy
- High usability (Target SUS > 80)
- Response time under 2 seconds for core APIs
- Responsible AI usage with transparent ranking logic

---

## 5. AI Justification

AI is essential in this system because:

1. Manual product entry is inefficient and error-prone.
2. Sentiment analysis captures nuanced trust signals beyond star ratings.
3. Hyperlocal logistics require intelligent spatial processing.
4. Vendors with low digital literacy need automation assistance.

The platform would not achieve scalability or usability without AI-powered assistance.

---

## 6. Constraints

- Designed for low-resource devices (2GB RAM smartphones)
- Intermittent internet connectivity
- Minimal vendor technical knowledge

---

## 7. Success Metrics

- ≥70% reduction in onboarding time
- ≥40% reduction in delivery errors
- ≥15% increase in vendor order volume
- SUS Score ≥ 85
