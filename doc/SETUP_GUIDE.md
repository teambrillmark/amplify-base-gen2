# Setup and Development Guide

## Prerequisites

### System Requirements
- **Node.js**: Version 18.17 or higher (LTS recommended)
- **npm**: Version 9 or higher (included with Node.js)
- **Git**: For version control
- **AWS Account**: With appropriate permissions for Amplify Gen 2
- **Stripe Account**: For payment integration (optional for basic features)

### Amplify Gen 2 Requirements
- AWS CLI v2 configured with credentials
- Modern terminal/command line interface
- Internet connection for AWS resource deployment

### AWS CLI Configuration
```bash
# Install AWS CLI (if not already installed)
# For macOS: brew install awscli
# For Windows: Download from https://aws.amazon.com/cli/
# For Linux: Use package manager

# Configure AWS credentials
aws configure
# AWS Access Key ID: [Your Access Key]
# AWS Secret Access Key: [Your Secret Key]
# Default region name: [e.g., us-east-1]
# Default output format: json
```

### Amplify Gen 2 CLI Installation
```bash
# Install Amplify Gen 2 CLI globally
npm install -g @aws-amplify/backend-cli

# No additional configuration needed - uses AWS CLI credentials
# Gen 2 doesn't require 'amplify configure' like Gen 1
```

## Project Setup

### 1. Clone Repository
```bash
git clone <repository-url>
cd amplify-base-gen2
```

### 2. Install Dependencies
```bash
# Install project dependencies
npm install

# Backend dependencies are managed automatically by Amplify Gen 2
# No need to manually install amplify backend dependencies
```

### 3. Environment Configuration

#### AWS Configuration
Ensure your AWS credentials have the following permissions:
- CloudFormation (full access)
- IAM (create roles and policies)
- DynamoDB (full access)
- S3 (full access)
- Lambda (full access)
- AppSync (full access)
- Cognito (full access)
- Comprehend (full access)
- Systems Manager Parameter Store (read/write)

#### Stripe Configuration (Optional)
1. Create a Stripe account at [stripe.com](https://stripe.com)
2. Get your secret key from the Stripe dashboard
3. Store the key in AWS Systems Manager Parameter Store:

```bash
aws ssm put-parameter \
  --name "/stripe/STRIPE_SECURE_KEY" \
  --value "sk_test_your_stripe_secret_key" \
  --type "SecureString" \
  --description "Stripe secret key for product integration"
```

## Development Workflow

### 1. Start Sandbox Environment
```bash
# Start Amplify Gen 2 sandbox (deploys backend to AWS)
npx ampx sandbox

# Alternative: Use the watch mode for auto-deployment on changes
npx ampx sandbox --watch

# This will:
# - Deploy your backend to AWS
# - Generate amplify_outputs.json configuration
# - Set up real-time sync with your local code
# - Create an isolated development environment
```

### 2. Start Development Server
```bash
# In a new terminal
npm run dev
```

Your application will be available at `http://localhost:3000`

### 3. Development Commands

#### Backend Operations
```bash
# Start/restart sandbox
npx ampx sandbox

# Start sandbox with watch mode (auto-redeploy on changes)
npx ampx sandbox --watch

# Check sandbox status
npx ampx sandbox status

# Delete sandbox (cleanup)
npx ampx sandbox delete

# Generate outputs for production deployment
npx ampx generate outputs --app-id <app-id> --branch <branch>

# Generate TypeScript types (usually automatic)
npx ampx generate outputs
```

#### Frontend Operations
```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Lint code
npm run lint
```

## Project Structure Deep Dive

### Backend Structure (`/amplify`)
```
amplify/
├── auth/
│   ├── post-confirmation/
│   │   ├── handler.ts          # Post-signup user profile creation
│   │   └── resource.ts         # Lambda resource definition
│   └── resource.ts             # Authentication configuration
├── data/
│   ├── resolvers/
│   │   ├── archiveProduct.js   # Custom product archiving logic
│   │   └── countSentiments.js  # Sentiment aggregation logic
│   └── resource.ts             # GraphQL schema and data models
├── functions/
│   ├── detect-review-sentiment/
│   │   └── handler.ts          # AWS Comprehend integration
│   ├── keep-aggregates/
│   │   └── handler.ts          # General statistics tracking
│   ├── stripe-product/
│   │   └── handler.ts          # Stripe product/price sync
│   ├── update-sentiment-counts/
│   │   └── handler.ts          # Sentiment count aggregation
│   └── update-statuses/
│       └── handler.ts          # Status distribution tracking
├── storage/
│   └── resource.ts             # S3 bucket configuration
├── backend.ts                  # Main backend definition
├── backend.utils.ts            # Build utilities
└── package.json                # Backend dependencies
```

### Frontend Structure (`/src`)
```
src/
├── app/                        # Next.js App Router pages
│   ├── admin/
│   │   ├── dashboard/          # Admin analytics dashboard
│   │   ├── product-create/     # Create new products
│   │   └── product-edit/       # Edit existing products
│   ├── products/[id]/          # Product detail pages
│   ├── reviews/                # Review management
│   ├── signin/                 # Authentication page
│   ├── user/[id]/              # User profile pages
│   ├── layout.tsx              # Root layout with providers
│   └── page.tsx                # Homepage
├── components/
│   ├── AuthenticatorClient.tsx # Authentication wrapper
│   ├── ClientSetupContextProvider.tsx # Amplify setup
│   ├── ProductForm.tsx         # Reusable product form
│   ├── ReviewForm.tsx          # Reusable review form
│   ├── NavBar.tsx              # Navigation component
│   └── [other-components].tsx
├── context/
│   └── AdminContext.tsx        # Admin permission context
├── actions/                    # Server actions
├── utils/
│   ├── amplify-utils.ts        # Server-side Amplify utilities
│   └── util.ts                 # General utilities
├── middleware.ts               # Route protection
└── types.ts                    # TypeScript definitions
```

## Configuration Files

### Amplify Gen 2 Configuration
Amplify Gen 2 automatically generates configuration based on your backend definition. The key files are:

- `amplify_outputs.json` - Auto-generated configuration (replaces aws-exports.js)
- `amplify/backend.ts` - Your backend definition (code-first approach)
- No manual configuration files needed (unlike Gen 1)

### Next.js Configuration
```javascript
// next.config.mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    serverComponentsExternalPackages: ['@aws-sdk/client-s3'],
  },
};

export default nextConfig;
```

### TypeScript Configuration
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

## Development Best Practices

### 1. Schema Changes
When modifying the data schema:
```bash
# 1. Edit amplify/data/resource.ts
# 2. If using watch mode, changes deploy automatically
npx ampx sandbox --watch

# OR manually redeploy
npx ampx sandbox

# 3. Types are generated automatically in amplify_outputs.json
# Frontend will pick up new types on next build/refresh
```

### 2. Function Development
For Lambda function changes:
```bash
# Functions are automatically redeployed with sandbox
npx ampx sandbox

# Use watch mode for faster iteration
npx ampx sandbox --watch

# View function logs in real-time
npx ampx sandbox logs --function <function-name> --follow

# View all recent logs
npx ampx sandbox logs
```

### 3. Authentication Testing
Test different user roles:
1. Create admin user through Cognito console
2. Add user to "Admins" group
3. Test permissions in application

### 4. Storage Testing
Test file uploads:
1. Ensure bucket permissions are correct
2. Test with different file types
3. Verify access controls

## Environment Management

### Development Environment
- **Sandbox**: Isolated development environment
- **Hot Reloading**: Changes deploy automatically
- **Local Testing**: Full feature testing locally

### Staging Environment
```bash
# For hosted environments, use Amplify Console
# Connect your repository and configure branch-based deployments

# For manual deployment to a specific environment
npx ampx generate outputs --app-id <staging-app-id> --branch staging
```

### Production Environment
```bash
# Deploy via Amplify Console for production
# Or generate outputs for production app
npx ampx generate outputs --app-id <prod-app-id> --branch main

# For CI/CD pipelines
npx ampx deploy --branch main
```

## Troubleshooting

### Common Issues

#### 1. Authentication Errors
```bash
# Check AWS credentials
aws sts get-caller-identity

# Reconfigure if needed
aws configure
```

#### 2. Build Failures
```bash
# Clear node modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear Next.js cache
rm -rf .next
npm run build
```

#### 3. Backend Deployment Issues
```bash
# Check sandbox status
npx ampx sandbox status

# View detailed sandbox logs
npx ampx sandbox logs --verbose

# Clean restart sandbox
npx ampx sandbox delete
npx ampx sandbox

# Force regenerate outputs
npx ampx generate outputs --force
```

#### 4. Type Generation Issues
```bash
# Types are auto-generated, but if issues occur:
npx ampx generate outputs --force

# Check amplify_outputs.json exists
ls -la amplify_outputs.json

# Restart development server to pick up new types
npm run dev
```

### Debug Commands
```bash
# View all sandbox logs
npx ampx sandbox logs

# View specific function logs
npx ampx sandbox logs --function <function-name>

# Check resource status
npx ampx sandbox status

# View generated configuration
cat amplify_outputs.json

# Debug deployment issues
npx ampx sandbox logs --verbose
```

## Testing

### Unit Testing Setup
```bash
# Install testing dependencies
npm install --save-dev jest @testing-library/react @testing-library/jest-dom

# Create test configuration
# jest.config.js
```

### Integration Testing
```bash
# Test with real AWS resources
npm run test:integration
```

### End-to-End Testing
```bash
# Install Playwright
npm install --save-dev @playwright/test

# Run E2E tests
npm run test:e2e
```

## Performance Optimization

### Frontend Optimization
- Use Next.js Image component for images
- Implement code splitting
- Optimize bundle size
- Use React.memo for expensive components

### Backend Optimization
- Configure appropriate Lambda memory/timeout
- Use DynamoDB efficiently
- Implement proper caching strategies
- Monitor CloudWatch metrics

## Security Considerations

### 1. Authentication
- Use strong password policies
- Implement MFA for admin users
- Regular security audits

### 2. Authorization
- Follow principle of least privilege
- Test authorization rules thoroughly
- Regular permission reviews

### 3. Data Protection
- Use field-level authorization
- Encrypt sensitive data
- Regular backup strategies

### 4. API Security
- Rate limiting implementation
- Input validation
- CORS configuration

## Monitoring and Logging

### CloudWatch Integration
- Function execution metrics
- Error tracking
- Performance monitoring

### Application Monitoring
```bash
# View function logs in real-time
npx ampx sandbox logs --function detect-review-sentiment --follow

# View all function logs
npx ampx sandbox logs

# Monitor API usage via CloudWatch Console
# Check AppSync GraphQL metrics
# Monitor DynamoDB performance metrics
```

## Additional Gen 2 Features

### Real-time Development
- **Watch Mode**: Automatic redeployment on code changes
- **Instant Feedback**: See backend changes reflected immediately
- **Type Safety**: Auto-generated TypeScript types
- **Local Testing**: Full cloud backend accessible locally

### Modern Developer Experience
- **Code-first Configuration**: Define infrastructure in TypeScript
- **IntelliSense Support**: Full IDE support for configuration
- **Git-friendly**: All configuration in source control
- **Team Collaboration**: Shared backend definitions

### Migration from Gen 1
If migrating from Amplify Gen 1:
1. **Data Migration**: Use DynamoDB backup/restore
2. **Auth Migration**: Export/import Cognito users
3. **Schema Translation**: Convert GraphQL transform to Gen 2 schema
4. **Function Updates**: Adapt Lambda functions to new patterns

This comprehensive setup guide provides everything needed to get the Onyx Store project running locally and deployed to AWS using the latest Amplify Gen 2 features and best practices.