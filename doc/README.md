# Onyx Store - NextJS + AWS Amplify Gen 2 Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [AWS Amplify Gen 2 Features](#aws-amplify-gen-2-features)
4. [Technology Stack](#technology-stack)
5. [Project Structure](#project-structure)
6. [Data Models & Schema](#data-models--schema)
7. [Authentication & Authorization](#authentication--authorization)
8. [Lambda Functions](#lambda-functions)
9. [Storage Configuration](#storage-configuration)
10. [Frontend Components](#frontend-components)
11. [Development Setup](#development-setup)
12. [Deployment](#deployment)
13. [API Reference](#api-reference)

## Project Overview

Onyx Store is a modern e-commerce application built with NextJS and AWS Amplify Gen 2, demonstrating comprehensive cloud-native features including authentication, real-time data, file storage, serverless functions, and AI-powered sentiment analysis.

### Key Features
- **User Authentication**: Email-based sign up/sign in with Cognito
- **Product Management**: CRUD operations for products with image uploads
- **Review System**: User reviews with AI sentiment analysis via AWS Comprehend
- **Admin Dashboard**: Analytics showing review sentiments and product/review statistics
- **Payment Integration**: Stripe integration for product pricing
- **Real-time Updates**: GraphQL subscriptions for live data
- **File Storage**: S3 integration for product images
- **Serverless Processing**: Lambda functions for data aggregation and external API integration

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │    │   AWS Amplify    │    │   External      │
│   (NextJS)      │◄──►│   Gen 2          │◄──►│   Services      │
│                 │    │                  │    │   (Stripe)      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │
                       ┌────────▼────────┐
                       │   AWS Services  │
                       │                 │
                       │ • Cognito       │
                       │ • DynamoDB      │
                       │ • AppSync       │
                       │ • Lambda        │
                       │ • S3            │
                       │ • Comprehend    │
                       │ • SSM           │
                       └─────────────────┘
```

## AWS Amplify Gen 2 Features

### Core Amplify Gen 2 Capabilities Used

1. **Fullstack TypeScript Development**
   - Code-first backend definition with TypeScript
   - End-to-end type safety from backend to frontend
   - Unified developer experience with shared types
   - IntelliSense support for all resources

2. **Modern Data Layer (AppSync GraphQL)**
   - Declarative data modeling with relationships
   - Real-time subscriptions out-of-the-box
   - Custom business logic with JavaScript resolvers
   - Automatic API generation from schema

3. **Zero-Config Authentication (Amazon Cognito)**
   - Multi-factor authentication support
   - Social sign-in providers
   - Advanced security features (password policies, account recovery)
   - Seamless integration with authorization rules

4. **Per-Developer Cloud Sandbox**
   - Isolated development environments
   - Real AWS resources for local development
   - Hot reloading with cloud backend
   - Branch-based deployment workflows

5. **File Storage (Amazon S3)**
   - Path-based access control
   - Pre-signed URL generation
   - Integration with frontend components
   - Automatic CDN distribution

### Gen 2 Architecture Principles

- **Resource-based Organization**: Each feature (auth, data, storage) in separate resource files
- **Convention over Configuration**: Sensible defaults with customization options
- **Local-first Development**: Develop against real cloud resources locally
- **GitOps Integration**: Infrastructure and application code in same repository
- **Production-ready Defaults**: Security and performance best practices built-in

## Technology Stack

### Frontend
- **NextJS 14.2.4**: React framework with app router
- **React 18**: UI library with hooks
- **TypeScript**: Type safety
- **Tailwind CSS**: Utility-first styling
- **React Hook Form**: Form validation and management

### Backend (AWS Amplify Gen 2)
- **AWS AppSync**: Managed GraphQL API
- **Amazon Cognito**: User authentication and authorization
- **Amazon DynamoDB**: NoSQL database
- **AWS Lambda**: Serverless functions
- **Amazon S3**: Object storage for images
- **AWS Comprehend**: AI sentiment analysis
- **AWS Systems Manager**: Parameter store for secrets

### Third-Party Integrations
- **Stripe**: Payment processing
- **Lodash**: Utility functions
- **Recharts**: Data visualization for admin dashboard

### Development Tools
- **AWS CDK**: Infrastructure as code
- **ESBuild**: Fast JavaScript bundler
- **ESLint**: Code linting
- **PostCSS**: CSS processing

## Project Structure

```
amplify-base-gen2/
├── amplify/                     # Amplify Gen 2 backend configuration
│   ├── auth/                   # Authentication configuration
│   │   ├── post-confirmation/  # Post-confirmation Lambda trigger
│   │   └── resource.ts         # Auth resource definition
│   ├── data/                   # Data layer configuration
│   │   ├── resolvers/          # Custom GraphQL resolvers
│   │   └── resource.ts         # Data schema definition
│   ├── functions/              # Lambda functions
│   │   ├── detect-review-sentiment/
│   │   ├── keep-aggregates/
│   │   ├── stripe-product/
│   │   ├── update-sentiment-counts/
│   │   └── update-statuses/
│   ├── storage/                # S3 storage configuration
│   ├── backend.ts              # Main backend definition
│   └── backend.utils.ts        # Backend utilities
├── src/                        # Frontend source code
│   ├── app/                    # NextJS app router pages
│   ├── components/             # React components
│   ├── actions/                # Server actions
│   ├── context/                # React context providers
│   ├── utils/                  # Utility functions
│   └── types.ts                # TypeScript type definitions
├── doc/                        # Project documentation
└── public/                     # Static assets
```

## Data Models & Schema

### Core Models

#### Product Model
```typescript
Product: {
  name: string (required)
  description: string (required)
  price: integer (required) // In cents
  images: ProductImage[] // One-to-many relationship
  mainImageS3Key: string
  isActive: boolean
  isArchived: boolean
  stripeProductId: string
  stripePriceId: string
  reviews: Review[] // One-to-many relationship
  status: ProductStatusType
}
```

#### Review Model
```typescript
Review: {
  title: string (required)
  rating: integer (required)
  content: string (required)
  productId: string
  userId: string
  sentiment: string // AI-generated
  sentimentScores: {
    mixed: float
    negative: float
    neutral: float
    positive: float
  }
  languageCode: string
  sentimentProcessed: boolean
  status: ReviewStatusType
}
```

#### UserProfile Model
```typescript
UserProfile: {
  userId: string (required)
  username: string (required)
  email: string // Field-level auth
  preferredUsername: string
  birthdate: date
  givenName: string
  middleName: string
  familyName: string
  profilePicture: string
  profileOwner: string // Owner-based auth
  reviews: Review[]
}
```

### Aggregation Models

#### SentimentCounts
Tracks sentiment distribution across all reviews
```typescript
SentimentCounts: {
  sentiment: string (required) // PRIMARY KEY
  count: integer (required)
  createdAt: datetime
  updatedAt: datetime
}
```

#### GeneralAggregates
Tracks general statistics
```typescript
GeneralAggregates: {
  entityType: string (required) // PRIMARY KEY (Product, Review, User)
  count: integer (required)
  createdAt: datetime
  updatedAt: datetime
}
```

### Enums

```typescript
ProductStatusType: "PENDING" | "ACTIVE" | "ARCHIVED"
ReviewStatusType: "PENDING" | "ACTIVE" | "ARCHIVED" | "REJECTED" | "FLAGGED" | "UNDER_REVIEW" | "APPROVED"
```

## Authentication & Authorization

### User Groups
- **Admins**: Full access to all resources
- **Users**: Standard authenticated users (auto-assigned)
- **Guests**: Unauthenticated read access to products

### Authorization Patterns

#### Guest Access
```typescript
allow.guest().to(["read"]) // Products and ProductImages
```

#### Owner-Based Access
```typescript
allow.owner().to(["read", "create", "update"]) // Reviews
allow.ownerDefinedIn("profileOwner") // UserProfile ownership
```

#### Group-Based Access
```typescript
allow.group("Admins") // Full admin access
```

#### Field-Level Authorization
```typescript
email: a.string().authorization((allow) => [
  allow.group("Admins"),
  allow.ownerDefinedIn("profileOwner").to(["read"])
])
```

### Post-Confirmation Trigger
Automatically creates UserProfile when new user signs up:
- Extracts user attributes from Cognito
- Creates corresponding UserProfile record in DynamoDB
- Links profile to user's Cognito ID

## Lambda Functions

### 1. Stripe Product Function (`stripe-product/handler.ts`)
**Trigger**: DynamoDB Product table events
**Purpose**: Stripe integration for product and pricing management
- Creates Stripe products when new products are added
- Updates Stripe prices when product pricing changes
- Stores Stripe IDs back in DynamoDB

### 2. Detect Review Sentiment (`detect-review-sentiment/handler.ts`)
**Trigger**: DynamoDB Review table events
**Purpose**: AI sentiment analysis of reviews
- Uses AWS Comprehend to analyze review content
- Detects language and sentiment scores
- Updates review record with sentiment data

### 3. Update Sentiment Counts (`update-sentiment-counts/handler.ts`)
**Trigger**: DynamoDB Review table events
**Purpose**: Maintains sentiment aggregations
- Tracks counts by sentiment type (POSITIVE, NEGATIVE, NEUTRAL, MIXED)
- Updates SentimentCounts table for admin dashboard

### 4. Keep Aggregates (`keep-aggregates/handler.ts`)
**Trigger**: DynamoDB Product and Review table events
**Purpose**: Maintains general statistics
- Tracks total counts of products and reviews
- Updates GeneralAggregates table

### 5. Update Statuses (`update-statuses/handler.ts`)
**Trigger**: DynamoDB Product and Review table events
**Purpose**: Maintains status distribution statistics
- Tracks counts by status for products and reviews
- Updates ProductStatuses and ReviewStatuses tables

### Lambda Configuration Features
- **Event Sources**: DynamoDB streams for real-time processing
- **IAM Policies**: Custom policies for service access
- **Environment Variables**: Automatic table name injection
- **Runtime**: Node.js 20.x with TypeScript
- **Build**: ESBuild for optimal performance

## Storage Configuration

### S3 Bucket: `onyxStoreNextGen2Bucket`
**Path-based access control**:
```typescript
"product-images/*": [
  allow.guest.to(["read"]),
  allow.authenticated.to(["read"]),
  allow.groups(["Admins"]).to(["read", "write", "delete"])
]
```

### Image Management
- **Upload**: Admin-only through StorageManager component
- **Access**: Public read access for product display
- **Metadata**: Alt text and S3 keys stored in ProductImage model
- **Main Image**: Configurable main image per product

## Frontend Components

### Core Components

#### Authentication
- **AuthenticatorClient**: Amplify UI authenticator wrapper
- **ClientSetupContextProvider**: Amplify configuration setup

#### Product Management
- **ProductForm**: Shared form for create/edit operations
- **ProductCreate**: New product creation
- **ProductEdit**: Product modification
- **ProductItem**: Product display component
- **ProductItemControls**: Admin controls for products

#### Review System
- **ReviewForm**: Shared form for review operations
- **ReviewCreate**: New review creation
- **ReviewEdit**: Review modification
- **ReviewItem**: Review display component

#### Admin Features
- **ProductReport**: Product analytics
- **SentimentReport**: Review sentiment visualization
- **AdminContext**: Admin permission context

#### Media Handling
- **Image**: Optimized image display
- **ImageCarousel**: Multiple image display
- **ImageUploader**: S3 upload interface

### Page Structure

#### Public Pages
- `/`: Product listing homepage
- `/products/[id]`: Product detail page
- `/signin`: Authentication page

#### User Pages
- `/user/[id]`: User profile page
- `/reviews/product/[productId]/new`: Create review
- `/reviews/[reviewId]/edit`: Edit review

#### Admin Pages
- `/admin/dashboard`: Analytics dashboard
- `/admin/product-create`: Create new product
- `/admin/product-edit/[id]`: Edit existing product

## Development Setup

### Prerequisites
- Node.js 18.17+ (LTS recommended)
- AWS CLI v2 with configured credentials
- Modern terminal with internet connection

### Installation
```bash
# Install project dependencies
npm install

# Install Amplify Gen 2 CLI
npm install -g @aws-amplify/backend-cli
```

### Amplify Gen 2 Setup
```bash
# Start per-developer cloud sandbox
npx ampx sandbox

# Or start with auto-reload on changes
npx ampx sandbox --watch

# Start development server (in new terminal)
npm run dev
```

### Environment Configuration
- **Cloud Sandbox**: Automatic isolated AWS environment per developer
- **Configuration**: Auto-generated `amplify_outputs.json`
- **Type Safety**: Automatic TypeScript type generation
- **Hot Reloading**: Backend changes deploy automatically in watch mode

## Deployment

### Development (Cloud Sandbox)
```bash
# Personal development environment
npx ampx sandbox

# With hot reloading
npx ampx sandbox --watch

# Clean up sandbox
npx ampx sandbox delete
```

### Production Deployment (Amplify Hosting)
1. **Connect Repository**: Link to GitHub/GitLab/Bitbucket via Amplify Console
2. **Branch-based Deployments**: Automatic staging and production environments
3. **Build Configuration**: Zero-config setup for Next.js + Amplify Gen 2
4. **Environment Promotion**: Seamless promotion from staging to production

### Modern Build Process
```yaml
# amplify.yml (auto-generated)
version: 1
backend:
  phases:
    build:
      commands:
        - npm ci --cache .npm --prefer-offline
        - npx ampx generate outputs --app-id $AWS_APP_ID --branch $AWS_BRANCH --format=json
frontend:
  phases:
    preBuild:
      commands:
        - npm ci --cache .npm --prefer-offline
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: .next
    files:
      - '**/*'
  cache:
    paths:
      - .next/cache/**/*
      - .npm/**/*
```

## API Reference

### TypeScript Client API

#### Type-Safe Data Operations
```typescript
// Auto-generated client with full type safety
import { generateClient } from 'aws-amplify/data';
import type { Schema } from '../amplify/data/resource';

const client = generateClient<Schema>();

// List products with type safety
const { data: products } = await client.models.Product.list({
  filter: { status: { eq: 'ACTIVE' } }
});

// Get product with relationships
const { data: product } = await client.models.Product.get(
  { id: productId },
  {
    selectionSet: ['id', 'name', 'price', 'images.*', 'reviews.*']
  }
);

// Real-time subscriptions
const subscription = client.models.Product.observeQuery().subscribe({
  next: ({ items, isSynced }) => {
    console.log('Products updated:', items);
  }
});
```

#### Server-Side Operations (Next.js)
```typescript
// Server actions with Amplify Gen 2
import { cookies } from 'next/headers';
import { generateServerClientUsingCookies } from '@aws-amplify/adapter-nextjs/data';
import { Schema } from '@/amplify/data/resource';
import config from '@/amplify_outputs.json';

// Server-side data fetching
export async function getProducts() {
  const client = generateServerClientUsingCookies<Schema>({
    config,
    cookies,
  });
  
  const { data } = await client.models.Product.list();
  return data;
}

// Server action for mutations
export async function createReview(formData: FormData) {
  const client = generateServerClientUsingCookies<Schema>({
    config,
    cookies,
  });
  
  await client.models.Review.create({
    title: formData.get('title') as string,
    rating: parseInt(formData.get('rating') as string),
    content: formData.get('content') as string,
    productId: formData.get('productId') as string,
  });
}
```

#### Admin Queries
```graphql
# Get sentiment counts
query CountReviewSentiments {
  countReviewSentiments {
    POSITIVE
    NEGATIVE
    NEUTRAL
    MIXED
  }
}

# Archive product
mutation ArchiveProduct($productId: ID!, $archive: Boolean!) {
  archiveProduct(productId: $productId, archive: $archive) {
    id
    status
    isArchived
  }
}
```

### Custom Resolvers

#### Archive Product (`resolvers/archiveProduct.js`)
Custom mutation for soft-deleting products
- Sets `isArchived` flag
- Updates product status
- Admin-only access

#### Count Sentiments (`resolvers/countSentiments.js`)
Custom query for sentiment aggregation
- Returns real-time sentiment counts
- Used in admin dashboard
- Efficient aggregation logic

This documentation provides a comprehensive overview of the Onyx Store project built with NextJS and AWS Amplify Gen 2, covering all major features, architecture decisions, and implementation details.