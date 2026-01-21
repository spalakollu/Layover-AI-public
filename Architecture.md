# Layover AI - System Architecture

## Overview

Layover AI is a modern, full-stack travel application built with a hybrid architecture that combines web, iOS, and Android platforms. The system leverages serverless functions, real-time databases, and AI services to deliver a seamless user experience across all platforms.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          Client Layer                               │
├─────────────────────────────────────────────────────────────────────┤
│  Web (React/Vite)    │  iOS (Capacitor)   │  Android (Capacitor)  │
│  • PWA Support       │  • Native Plugins  │  • Native Plugins     │
│  • Responsive UI     │  • Apple Sign-In   │  • Google Sign-In     │
│  • Browser Auth      │  • Push Notifs     │  • Push Notifications │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        API Gateway Layer                            │
├─────────────────────────────────────────────────────────────────────┤
│  Vercel Serverless Functions (Node.js/TypeScript)                  │
│  • 13+ API endpoints                                               │
│  • JWT Authentication                                              │
│  • Rate Limiting & Security                                        │
│  • Request/Response Caching                                        │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌───────────────────────────────┐  ┌───────────────────────────────┐
│      Data Layer               │  │    Real-Time Layer            │
├───────────────────────────────┤  ├───────────────────────────────┤
│  PostgreSQL (Neon)            │  │  Firebase Firestore           │
│  • User data & profiles       │  │  • Real-time messaging        │
│  • Social connections         │  │  • Online presence            │
│  • Flight information         │  │  • Live notifications         │
│  • Airport data               │  │  • Activity feeds             │
│  • Business logic             │  └───────────────────────────────┘
└───────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      External Services Layer                        │
├─────────────────────────────────────────────────────────────────────┤
│  AI Services          │  Payment    │  Maps        │  Flight Data   │
│  • OpenAI/Groq        │  • Stripe   │  • Google    │  • Aerodata    │
│  • TensorFlow.js      │  • RevenueCat│  Maps API   │  • Aviasales   │
│  • ML Predictions     │             │              │                │
└─────────────────────────────────────────────────────────────────────┘
```

## Technology Stack

### Frontend

- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite 4
- **Styling**: Tailwind CSS with Radix UI components
- **State Management**: TanStack Query (React Query) for server state
- **Routing**: Wouter (lightweight router)
- **Forms**: React Hook Form with Zod validation
- **Internationalization**: i18next
- **Mobile**: Capacitor 7 for iOS and Android native apps

### Backend

- **Runtime**: Node.js 18+ with TypeScript
- **Deployment**: Vercel Serverless Functions
- **Authentication**: 
  - JWT tokens with HTTP-only cookies (primary)
  - Firebase Auth bridge for real-time features
  - Native authentication plugins for mobile
- **Database**: 
  - PostgreSQL (Neon) - primary data store
  - Firebase Firestore - real-time messaging and presence
  - Redis - caching and rate limiting

### AI & Machine Learning

- **AI Services**: 
  - Groq API (primary, fast inference)
  - OpenAI GPT-4 (fallback)
  - Custom prompt engineering for travel context
- **ML Models**:
  - TensorFlow.js for wait time predictions
  - Custom training pipeline for airport wait times
  - PostgreSQL pgvector for semantic search

### Infrastructure

- **Hosting**: Vercel Edge Network
- **CDN**: Vercel Edge Network (global distribution)
- **Database**: Neon PostgreSQL (serverless Postgres)
- **Real-time**: Firebase Firestore
- **Caching**: Redis (via Vercel KV or external)
- **Monitoring**: Sentry for error tracking
- **Analytics**: Vercel Analytics

### Third-Party Integrations

- **Payment Processing**: 
  - Stripe (web subscriptions)
  - RevenueCat (mobile IAP for iOS/Android)
- **Maps**: Google Maps API for airport visualizations
- **Flight Data**: 
  - Aerodata API for flight information
  - Aviasales API for flight search
- **Push Notifications**: Firebase Cloud Messaging

## System Components

### 1. Authentication System

The application uses a **hybrid authentication architecture**:

#### JWT Authentication (Primary)
- **Purpose**: Main application authentication and API access
- **Storage**: HTTP-only cookies (secure, XSS-protected)
- **Scope**: All application features except real-time messaging
- **Security**: Server-side validation, secure cookie attributes

#### Firebase Authentication (Secondary)
- **Purpose**: Real-time messaging, presence, and live updates
- **Method**: Custom Firebase tokens generated from JWT
- **Bridge**: Automatic sign-in when real-time features are needed
- **Mobile**: Native authentication plugins for iOS/Android

```
User Login → JWT Generation → Cookie Storage → Firebase Bridge → Real-time Access
```

### 2. User Management

#### User Data Storage
- **Primary**: PostgreSQL `users` table
- **Fields**: Profile, preferences, settings, location, subscription tier
- **Relationships**: Connections, flights, favorites, travel hubs
- **Privacy**: User-controlled visibility settings

#### Real-time Presence
- **Storage**: Firebase Firestore `presence` collection
- **Features**: Online/offline status, last seen, typing indicators
- **Updates**: Real-time synchronization across all clients
- **Privacy**: User-controlled online status visibility

### 3. Social Features

#### Connections System
- **Storage**: PostgreSQL `connections` table
- **Purpose**: Connect travelers with similar itineraries
- **States**: Pending, accepted, rejected, blocked
- **Features**: Connection requests, mutual connections, privacy controls

#### Messaging System
- **Primary**: Firebase Firestore for real-time messaging
- **Features**: 
  - Text messages with typing indicators
  - Read receipts
  - GIF support (Giphy integration)
  - Emoji reactions
  - Offline message synchronization
- **Security**: User-to-user messaging with verification

#### Travel Hubs
- **Purpose**: Topic-based travel communities
- **Features**: Airport-specific hubs, time-limited events, group messaging
- **Storage**: PostgreSQL for hub metadata, Firestore for real-time updates

### 4. Flight Management

#### Flight Data
- **Storage**: PostgreSQL `trips` and `flights` tables
- **Sources**: 
  - User input (manual entry)
  - Boarding pass scanning (barcode/Aztec code)
  - Flight number lookup via Aerodata API
- **Features**: 
  - Real-time flight status tracking
  - Gate change notifications
  - Delay alerts
  - Round-trip flight management

#### Boarding Pass Scanner
- **Technology**: Capacitor Camera plugin + ZXing barcode scanner
- **Supported Formats**: PDF417, Aztec, QR codes
- **Features**: Automatic flight data extraction

### 5. AI Assistant

#### AI Integration
- **Primary Provider**: Groq API (Llama 3 models) for fast responses
- **Fallback**: OpenAI GPT-4 for complex queries
- **Context**: Airport-specific knowledge, user preferences, flight information
- **Features**:
  - Voice input (speech-to-text)
  - Contextual recommendations
  - Layover planning
  - Restaurant and lounge suggestions
  - Terminal navigation guidance

#### AI Query Management
- **Rate Limiting**: 
  - Free tier: 5 queries/day
  - Pro tier: Unlimited queries
- **Caching**: Redis caching for common queries
- **Context**: Conversation history and user preferences

### 6. Airport Intelligence

#### Airport Data
- **Storage**: PostgreSQL `airport_places` table
- **Coverage**: 200+ airports worldwide
- **Data**: IATA/ICAO codes, coordinates, timezone, terminal information
- **Integration**: Google Maps API for visualization

#### Wait Time Predictions
- **ML Model**: TensorFlow.js model for TSA/immigration/customs wait times
- **Data Sources**: User reports, historical data, real-time factors
- **Features**: Terminal-specific predictions, lane type filtering
- **Storage**: PostgreSQL for reports, ML inference service for predictions

### 7. Location Services

#### User Location
- **Storage**: PostgreSQL `users` table (latitude/longitude)
- **Privacy**: User-controlled location sharing
- **Features**: 
  - Nearby travelers
  - Airport proximity detection
  - Location-based recommendations
- **Accuracy**: GPS-based with user consent

## Data Flow

### Authentication Flow

```
1. User logs in → Backend validates credentials
2. JWT token generated → Stored in HTTP-only cookie
3. User accesses real-time features → Firebase custom token generated
4. Firebase authentication → Real-time messaging access
```

### Messaging Flow

```
1. User sends message → Frontend validates input
2. Message sent to API → Server validates permissions
3. Message stored in Firestore → Real-time listeners trigger
4. Recipient receives update → Push notification sent (if offline)
5. Read receipt updated → Real-time sync across devices
```

### Flight Search Flow

```
1. User enters flight number → Frontend validates format
2. API request to Aerodata → Flight information retrieved
3. Data cached in Redis → Response returned to user
4. Flight added to trips → User profile updated
5. Real-time tracking initiated → Status updates via webhooks
```

### AI Query Flow

```
1. User asks question → Voice or text input
2. Request sent to API → Rate limiting checked
3. Context gathered → User preferences, flight info, airport data
4. Query sent to AI service → Groq (primary) or OpenAI (fallback)
5. Response cached → Returned to user
6. Conversation history saved → For context in future queries
```

## Mobile Architecture

### Capacitor Integration

The application uses Capacitor 7 to package the React web app as native iOS and Android applications.

#### iOS
- **Native Plugins**:
  - Apple Sign-In (@capacitor-community/apple-sign-in)
  - Push Notifications
  - Camera for boarding pass scanning
  - Geolocation
  - Status Bar and Splash Screen
- **Configuration**: Xcode project with native configurations
- **Deployment**: App Store Connect

#### Android
- **Native Plugins**:
  - Google Sign-In (native implementation)
  - Push Notifications (Firebase)
  - Camera for boarding pass scanning
  - Geolocation
  - Status Bar and Splash Screen
- **Configuration**: Gradle build with Google Services
- **Deployment**: Google Play Console

### Platform Detection

The application detects the platform and uses appropriate authentication methods:

```typescript
// Web: Browser-based OAuth
// iOS: Native Apple Sign-In plugin
// Android: Native Google Sign-In
```

## API Architecture

### Serverless Functions Structure

The API is organized into 13+ serverless function files:

- `api/ai.ts` - AI assistant endpoints
- `api/auth/auth.ts` - Authentication endpoints
- `api/social/index.ts` - Social features (connections, messaging)
- `api/flights-unified.ts` - Flight search and management
- `api/stripe.ts` - Payment processing and subscriptions
- `api/consolidated.ts` - Utility endpoints (users, airports, etc.)
- Additional specialized endpoints

### Request/Response Pattern

```typescript
// Standard API request flow
Request → CORS Check → Authentication → Validation → Business Logic → Response
```

### Caching Strategy

- **Redis Cache**: Frequently accessed data (airport info, flight data)
- **Request Deduplication**: Prevent duplicate simultaneous requests
- **Cache TTL**: Time-based expiration for different data types
- **Invalidation**: Smart cache invalidation on data updates

## Security Architecture

### Authentication Security

- **JWT Tokens**: Secure, signed tokens with expiration
- **HTTP-only Cookies**: XSS protection
- **CSRF Protection**: SameSite cookie attributes
- **Rate Limiting**: API endpoint protection (per user/IP)
- **Input Validation**: Comprehensive request validation with Zod

### Data Security

- **Encryption**: TLS/SSL for all communications
- **Database Security**: 
  - Row-level security policies
  - Encrypted connections (SSL)
  - Parameterized queries (SQL injection prevention)
- **Firestore Security**: Comprehensive security rules
- **Privacy Controls**: User-controlled data visibility

### API Security

- **CORS Configuration**: Proper cross-origin policies
- **Authentication Middleware**: All protected endpoints validated
- **Error Handling**: Secure error messages (no sensitive data leak)
- **Audit Logging**: Comprehensive audit trail for sensitive operations

## Performance Architecture

### Frontend Performance

- **Code Splitting**: Route-based and vendor chunk splitting
- **Lazy Loading**: Component lazy loading
- **Caching**: React Query for API response caching
- **PWA**: Service worker for offline functionality
- **Optimization**: 
  - Vite for fast builds
  - Tree shaking for smaller bundles
  - Asset optimization

### Backend Performance

- **Serverless Functions**: Automatic scaling with Vercel
- **Connection Pooling**: Database connection optimization
- **Caching**: Redis for frequently accessed data
- **CDN**: Vercel Edge Network for static assets
- **Request Optimization**: Request deduplication and batching

### Database Performance

- **Indexing**: Optimized database indexes on frequently queried columns
- **Query Optimization**: Efficient query patterns and prepared statements
- **Connection Pooling**: Neon's built-in connection pooling
- **Read Replicas**: Available for read-heavy operations

## Deployment Architecture

### Development Environment

- **Local Development**: 
  - Frontend: Vite dev server (port 3000)
  - API: Vercel dev server (port 3001)
  - Database: Local PostgreSQL or remote Neon dev instance
- **Hot Reload**: Fast refresh for both frontend and API changes

### Production Environment

- **Frontend**: Vercel Edge Network (global CDN)
- **API**: Vercel Serverless Functions (auto-scaling)
- **Database**: Neon PostgreSQL (production instance)
- **Real-time**: Firebase Firestore (production project)
- **CDN**: Static assets served from Vercel Edge Network

### CI/CD Pipeline

- **Version Control**: Git with GitHub
- **Automated Testing**: Jest and React Testing Library
- **Deployment**: Automatic Vercel deployment on push to main
- **Environment Variables**: Secure management via Vercel dashboard
- **Monitoring**: Sentry error tracking and Vercel Analytics

## Scalability Considerations

### Horizontal Scaling

- **Serverless Architecture**: Automatic scaling with Vercel
- **Database Scaling**: Neon's serverless Postgres scales automatically
- **Firestore Scaling**: Automatic scaling with Firebase
- **CDN Scaling**: Global edge network for static assets

### Vertical Scaling

- **Function Memory**: Configurable memory allocation per function
- **Database Resources**: Scalable Neon instances
- **Caching Layer**: Redis for performance optimization
- **Monitoring**: Performance monitoring and alerting

## Monitoring and Observability

### Application Monitoring

- **Error Tracking**: Sentry for error monitoring and alerts
- **Performance Monitoring**: Vercel Analytics for performance metrics
- **Custom Logging**: Application-specific logging for debugging
- **Real-time Metrics**: Response times, error rates, throughput

### Infrastructure Monitoring

- **Database Monitoring**: Query performance and connection usage
- **Firebase Monitoring**: Real-time usage and errors via Firebase Console
- **API Monitoring**: Endpoint performance and error rates via Vercel
- **User Analytics**: User behavior and engagement tracking

## Future Architecture Considerations

### Short-term Improvements

- API consolidation (reduce from 13+ to 3-4 main files)
- Enhanced caching strategies
- Advanced monitoring and alerting
- Performance optimizations

### Medium-term Improvements

- Advanced AI capabilities (agentic AI workflows)
- Enhanced ML predictions
- Real-time analytics
- Advanced security features

### Long-term Considerations

- Multi-region deployment
- Advanced AI orchestration (LangGraph/LangChain)
- Microservices architecture consideration
- Enterprise features and APIs


**Last Updated**: January 2025  
**Version**: 1.0.3
