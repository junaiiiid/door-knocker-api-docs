# Door Knocker Role-Based Authentication API

Complete role-based authentication system built with Supabase Edge Functions and Supabase Auth.

## Overview

This API provides a secure authentication system with three user roles:
- **ADMIN**: Full system access, can manage all users and create other ADMINs
- **MARKETER**: Marketing-specific permissions
- **TECHNICIAN**: Technical operations permissions (default role)

## Files

- **index.html** - Swagger UI documentation
- **openapi.yaml** - OpenAPI 3.0 specification (updated for role-based auth)
- **README.md** - This file

## Architecture

### Database Schema
- **profiles table**: Stores user roles and profile information (linked to auth.users)
- **Row Level Security (RLS)**: Enforces role-based access control
- **Enum type**: `user_role` for ADMIN, MARKETER, TECHNICIAN

### API Endpoints
1. **POST /signUp** - Register new users with role assignment
2. **POST /login** - Authenticate users and get JWT token
3. **GET /getUser** - Fetch user profile and role information
4. **GET /getAppContent** - Get role-based application content and menu configuration
5. **POST /logout** - Invalidate user session
6. **POST /forgotPassword** - Request password reset email
7. **POST /resetPassword** - Reset password with token

## Setup Instructions

### 1. Prerequisites
- Supabase project created
- Supabase CLI installed: `npm install -g supabase`
- Project linked to Supabase

### 2. Apply Database Migration

Run the SQL migration to create the profiles table and RLS policies:

```bash
# Push migrations to your Supabase project
supabase db push
```

Or manually apply in Supabase Dashboard > SQL Editor:
```sql
-- Copy contents of: supabase/migrations/20250113000001_create_profiles_table.sql
```

### 3. Deploy Edge Functions

Deploy all authentication functions:

```bash
# Deploy all functions
supabase functions deploy signUp
supabase functions deploy login
supabase functions deploy getUser
supabase functions deploy getAppContent
supabase functions deploy logout
supabase functions deploy forgotPassword
supabase functions deploy resetPassword
```

### 4. Create First ADMIN User

Since creating ADMIN users requires ADMIN authentication, manually create the first ADMIN:

**Option A: Via Supabase Dashboard**
1. Go to Authentication > Users
2. Create a new user with email/password
3. Copy the user's UUID
4. Go to Table Editor > profiles
5. Insert: `id` = UUID, `role` = ADMIN, `full_name` = "Admin User"

**Option B: Via SQL**
```sql
-- First create the auth user in Dashboard, then:
INSERT INTO profiles (id, role, full_name)
VALUES ('USER_UUID_HERE', 'ADMIN', 'Admin User');
```

Now this user can create other ADMIN users via the API.

## API Usage Examples

### 1. Sign Up (Create Account)

**Default Role (TECHNICIAN)**
```bash
curl -X POST https://YOUR-PROJECT.supabase.co/functions/v1/signUp \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "tech@example.com",
    "password": "SecurePass123!",
    "fullName": "John Technician"
  }'
```

**Create ADMIN User (Requires ADMIN Token)**
```bash
curl -X POST https://YOUR-PROJECT.supabase.co/functions/v1/signUp \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_ADMIN_JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@example.com",
    "password": "SecurePass123!",
    "fullName": "Admin User",
    "role": "ADMIN"
  }'
```

### 2. Login

```bash
curl -X POST https://YOUR-PROJECT.supabase.co/functions/v1/login \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "tech@example.com",
    "password": "SecurePass123!"
  }'
```

Response:
```json
{
  "status": "success",
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "role": "TECHNICIAN",
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "tech@example.com",
    "fullName": "John Technician"
  }
}
```

### 3. Get User Profile

**Get Own Profile**
```bash
curl -X GET https://YOUR-PROJECT.supabase.co/functions/v1/getUser \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

Response:
```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "tech@example.com",
    "role": "TECHNICIAN",
    "full_name": "John Technician",
    "created_at": "2025-01-13T10:00:00.000Z"
  }
}
```

**Get Another User's Profile (ADMIN only)**
```bash
curl -X GET "https://YOUR-PROJECT.supabase.co/functions/v1/getUser?user_id=660e8400-e29b-41d4-a716-446655440001" \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_ADMIN_JWT_TOKEN"
```

### 4. Logout

```bash
curl -X POST https://YOUR-PROJECT.supabase.co/functions/v1/logout \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### 5. Forgot Password

```bash
curl -X POST https://YOUR-PROJECT.supabase.co/functions/v1/forgotPassword \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{"email": "tech@example.com"}'
```

### 6. Get App Content (Role-based Menu Configuration)

```bash
curl -X GET https://YOUR-PROJECT.supabase.co/functions/v1/getAppContent \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**ADMIN User Response:**
```json
{
  "data": {
    "menu_items": [
      {
        "name": "search_bar",
        "title": "Search",
        "description": "Search Job, Referrals & Campaigns",
        "id": 0,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "dashboard",
        "title": "Dashboard",
        "description": "Create, Navigate, Lead & Iterate",
        "id": 1,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "referrals",
        "title": "Referrals",
        "description": "Add Referrals, start Campaigns & Get Leads",
        "id": 2,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "campaigns",
        "title": "Campaigns",
        "description": "Manage Your Campaigns",
        "id": 3,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "templates",
        "title": "Templates",
        "description": "Manage Your Templates",
        "id": 4,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "targeting",
        "title": "Targeting",
        "description": "",
        "id": 5,
        "enabled": true,
        "extended": true,
        "extended_options": [
          {
            "name": "targeting_zones",
            "title": "Targeting Zones",
            "description": "Targeting > Targeting Zones",
            "id": 51,
            "enabled": true,
            "extended": false,
            "extended_options": []
          }
        ]
      },
      {
        "name": "analytics",
        "title": "Analytics",
        "description": "",
        "id": 6,
        "enabled": true,
        "extended": true,
        "extended_options": [
          {
            "name": "overview",
            "title": "Overview",
            "description": "Analytics > Overview",
            "id": 61,
            "enabled": true,
            "extended": false,
            "extended_options": []
          }
        ]
      },
      {
        "name": "notifications",
        "title": "Notifications",
        "description": "",
        "id": 8,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "settings",
        "title": "Settings",
        "description": "Manage Your Account Settings",
        "id": 9,
        "enabled": true,
        "extended": false,
        "extended_options": []
      }
    ]
  }
}
```

**MARKETER User Response** (Limited features):
```json
{
  "data": {
    "menu_items": [
      {
        "name": "referrals",
        "title": "Referrals",
        "description": "Add Referrals, start Campaigns & Get Leads",
        "id": 2,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "campaigns",
        "title": "Campaigns",
        "description": "Manage Your Campaigns",
        "id": 3,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "targeting",
        "title": "Targeting",
        "description": "",
        "id": 5,
        "enabled": true,
        "extended": true,
        "extended_options": []
      },
      {
        "name": "analytics",
        "title": "Analytics", 
        "description": "",
        "id": 6,
        "enabled": true,
        "extended": true,
        "extended_options": []
      },
      {
        "name": "notifications",
        "title": "Notifications",
        "description": "",
        "id": 8,
        "enabled": true,
        "extended": false,
        "extended_options": []
      }
    ]
  }
}
```

**TECHNICIAN User Response** (Referrals only):
```json
{
  "data": {
    "menu_items": [
      {
        "name": "referrals",
        "title": "Referrals",
        "description": "Add Referrals & Manage Jobs",
        "id": 2,
        "enabled": true,
        "extended": false,
        "extended_options": []
      },
      {
        "name": "notifications",
        "title": "Notifications",
        "description": "",
        "id": 8,
        "enabled": true,
        "extended": false,
        "extended_options": []
      }
    ]
  }
}
```

### 7. Reset Password

```bash
curl -X POST https://YOUR-PROJECT.supabase.co/functions/v1/resetPassword \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "access_token": "TOKEN_FROM_EMAIL",
    "newPassword": "NewSecurePass123!"
  }'
```

## Security Features

- **Role-Based Access Control**: Only ADMINs can create other ADMINs
- **Organization-Based Content**: App content scoped to user's organization
- **Dynamic Menu Generation**: Menu items filtered by user role
- **Row Level Security**: Database-level access control
- **Password Security**: Managed by Supabase Auth (bcrypt)
- **JWT Tokens**: Secure session management
- **Email Enumeration Prevention**: Forgot password always returns success

## Role-Based Features Access

| Feature | ADMIN | MARKETER | TECHNICIAN |
|---------|-------|----------|------------|
| Search | ✅ | ✅ | ✅ (Limited) |
| Dashboard | ✅ | ✅ | ✅ |
| Referrals | ✅ | ✅ | ✅ |
| Campaigns | ✅ | ✅ | ❌ |
| Templates | ✅ | ❌ | ❌ |
| Targeting | ✅ | ✅ | ❌ |
| Analytics | ✅ | ✅ | ❌ |
| Quick Actions | ✅ Full | ✅ Limited | ✅ Referrals Only |
| Notifications | ✅ | ✅ | ✅ |
| Settings | ✅ | ❌ | ❌ |
| Integrations | ✅ | ❌ | ❌ |

## Error Codes

| Code | Description |
|------|-------------|
| INVALID_INPUT | Missing or invalid parameters |
| INVALID_EMAIL | Invalid email format |
| WEAK_PASSWORD | Password < 8 characters |
| INVALID_ROLE | Invalid role specified |
| EMAIL_EXISTS | Email already registered |
| INVALID_CREDENTIALS | Wrong email/password |
| UNAUTHORIZED | Missing/invalid auth |
| FORBIDDEN | Insufficient permissions |
| INVALID_TOKEN | Token invalid/expired |
| USER_NOT_FOUND | Requested user does not exist |
| PROFILE_NOT_FOUND | User profile not found |
| NO_ORGANIZATION | User not associated with organization |
| CONTENT_FETCH_FAILED | Failed to fetch app content |
| CONTENT_NOT_FOUND | App content not found for user role |

## Enhanced Address Fields

The address-related APIs (`/getAddressesFromZone`, `/verifyAddresses`, `/getAddressZoneById`, etc.) now return enhanced address objects with additional tracking and metadata fields:

### New Address Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `propertyType` | string | Type of property based on building classification | "Single Family Home", "Office Building" |
| `distanceFromCenter` | number | Distance from search center point in meters | 847 |
| `targeting_zone_name` | string | Human-readable name of the targeting zone | "Mountain View, CA Area" |
| `campaigns_used_in` | string[] | Array of campaign names that have used this address | `["Summer 2024 Campaign"]` |
| `zoneType` | string | Type of zone search performed | "radius(1.0km)" or "point(50 addresses)" |
| `postcards_sent` | number | Number of postcards sent to this address | 0 |
| `first_post_card_sent_date` | string\|null | ISO date when first postcard was sent | "2024-01-15T10:30:00.000Z" |
| `status` | enum | Verification status of address | "Valid", "Duplicate", "Opt-out", "UnVerified" |

### Address Status Values

- **UnVerified**: Default state, address not yet verified
- **Valid**: Address verified through PostGrid API (`/verifyAddresses`)
- **Duplicate**: Address already exists in another zone  
- **Opt-out**: Address manually marked as opted out

### Enhanced Response Example

```json
{
  "status": "success",
  "message": "Found 115 addresses",
  "addresses": [
    {
      "lat": 37.4176671,
      "long": -122.0928876,
      "address": "901, North Rengstorff Avenue",
      "residential": true,
      "building_type": "residential",
      "osm_id": "166790593",
      "propertyType": "Residential Building",
      "distanceFromCenter": 847,
      "targeting_zone_name": "Mountain View, CA Area",
      "campaigns_used_in": [],
      "zoneType": "radius(1.0km)",
      "postcards_sent": 0,
      "first_post_card_sent_date": null,
      "status": "UnVerified"
    }
  ]
}
```

**Note**: All request body formats remain unchanged - these are response-only enhancements.

## Get All Addresses API

The `/getAllAddresses` endpoint retrieves all addresses from all location zones in the user's organization with optional filtering capabilities.

### Endpoint

- **URL**: `/getAllAddresses`
- **Methods**: GET, POST
- **Authentication**: Required (Bearer Token)

### Features

- Returns addresses from all zones in the organization
- Optional filtering to show only non-Valid addresses (exclusions)
- Automatic deduplication based on lat/lng coordinates
- Includes zone metadata for each address
- Detailed status breakdown and metadata

### Request Methods

**GET Request** - Returns all addresses (default behavior):
```bash
curl -X GET https://YOUR-PROJECT.supabase.co/functions/v1/getAllAddresses \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**POST Request** - Supports filtering with request body:
```bash
curl -X POST https://YOUR-PROJECT.supabase.co/functions/v1/getAllAddresses \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "showOnlyExclusions": true
  }'
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `showOnlyExclusions` | boolean | false | If true, returns only addresses with status other than 'Valid' (UnVerified, Duplicate, Opt-out) |

### Response Format

```json
{
  "status": "success",
  "message": "Found 150 unique addresses from 3 zones",
  "metadata": {
    "total_zones": 3,
    "total_addresses_before_dedup": 152,
    "total_addresses_after_dedup": 150,
    "addresses_returned": 150,
    "showOnlyExclusions": false,
    "status_breakdown": {
      "Valid": 120,
      "UnVerified": 25,
      "Duplicate": 3,
      "Opt-out": 2
    },
    "processingTimeMs": 150
  },
  "addresses": [
    {
      "lat": 37.4176671,
      "long": -122.0928876,
      "address": "901, North Rengstorff Avenue",
      "residential": true,
      "building_type": "residential",
      "osm_id": "166790593",
      "propertyType": "Residential Building",
      "distanceFromCenter": 847,
      "targeting_zone_name": "Mountain View Zone",
      "campaigns_used_in": ["Summer 2024 Campaign"],
      "zoneType": "radius(1.0km)",
      "postcards_sent": 0,
      "first_post_card_sent_date": null,
      "status": "Valid",
      "zone_id": "c9abd41e-56a2-448e-942c-a0b37f796972",
      "zone_name": "Mountain View Zone",
      "campaign_id": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6"
    }
  ]
}
```

### Additional Address Fields

Each address in the response includes zone-specific metadata:

| Field | Type | Description |
|-------|------|-------------|
| `zone_id` | string (uuid) | ID of the zone this address belongs to |
| `zone_name` | string | Name of the zone this address belongs to |
| `campaign_id` | string (uuid) \| null | Campaign ID associated with the zone |

### Filtering Logic

- **showOnlyExclusions = false** (default): Returns all addresses regardless of status
- **showOnlyExclusions = true**: Returns only addresses with non-Valid status:
  - UnVerified
  - Duplicate
  - Opt-out

### Example: Get Only Exclusions

**Request:**
```bash
curl -X POST https://YOUR-PROJECT.supabase.co/functions/v1/getAllAddresses \
  -H "apikey: YOUR_ANON_KEY" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "showOnlyExclusions": true
  }'
```

**Response:**
```json
{
  "status": "success",
  "message": "Found 30 addresses with non-Valid status (30 total exclusions)",
  "metadata": {
    "total_zones": 3,
    "total_addresses_before_dedup": 152,
    "total_addresses_after_dedup": 150,
    "addresses_returned": 30,
    "showOnlyExclusions": true,
    "status_breakdown": {
      "Valid": 0,
      "UnVerified": 25,
      "Duplicate": 3,
      "Opt-out": 2
    },
    "processingTimeMs": 120
  },
  "addresses": [...]
}
```

### Business Rules

1. **Organization Scope**: Only returns addresses from zones in the user's organization
2. **Deduplication**: Addresses are automatically deduplicated based on lat/lng coordinates
3. **Zone Metadata**: Each address includes information about the zone it belongs to
4. **Status Tracking**: Provides detailed breakdown of address statuses
5. **Performance**: Returns processing time for monitoring

### Error Responses

| Status Code | Error | Description |
|-------------|-------|-------------|
| 401 | Unauthorized | Missing or invalid authentication token |
| 403 | Forbidden | User not in organization |
| 500 | Internal Server Error | Database or processing error |

### Use Cases

1. **Global Address Review**: View all addresses across all campaigns and zones
2. **Exclusion Management**: Filter to see only problematic addresses (duplicates, unverified, opt-outs)
3. **Data Quality**: Monitor address verification status across organization
4. **Compliance**: Track opt-outs across all zones
5. **Analytics**: Analyze address distribution and status across organization

## Viewing the Documentation

### Option 1: Open Locally

Simply open the Swagger UI in your browser:

```bash
# macOS
open docs/swagger.html

# Linux
xdg-open docs/swagger.html

# Windows
start docs/swagger.html
```

### Option 2: Serve with Python

```bash
# Python 3
cd docs
python3 -m http.server 8000

# Then visit: http://localhost:8000/swagger.html
```

### Option 3: Serve with Node.js

```bash
# Install http-server globally
npm install -g http-server

# Serve the docs
cd docs
http-server -p 8000

# Then visit: http://localhost:8000/swagger.html
```

### Option 4: Deploy to Hosting

The `swagger.html` file is a completely standalone file that can be deployed to any static hosting service:

- **Firebase Hosting** (recommended, see GitHub Actions example below)
- **Netlify**
- **Vercel**
- **GitHub Pages**
- **AWS S3**
- **Cloudflare Pages**

## Using the OpenAPI Spec

The `openapi.yaml` file can be used with various tools:

### Generate Client SDKs

```bash
# Install OpenAPI Generator
npm install @openapitools/openapi-generator-cli -g

# Generate TypeScript client
openapi-generator-cli generate -i docs/openapi.yaml -g typescript-axios -o client/typescript

# Generate Python client
openapi-generator-cli generate -i docs/openapi.yaml -g python -o client/python

# Generate Java client
openapi-generator-cli generate -i docs/openapi.yaml -g java -o client/java
```

### Validate the Spec

```bash
# Install Swagger CLI
npm install -g @apidevtools/swagger-cli

# Validate
swagger-cli validate docs/openapi.yaml
```

### Use with Postman

1. Open Postman
2. Click "Import"
3. Select `openapi.yaml`
4. All endpoints will be imported as a collection

## Deployment with GitHub Actions

### Deploy to Firebase Hosting

Create `.github/workflows/deploy-docs.yml`:

```yaml
name: Deploy API Documentation

on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to Firebase Hosting
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: ${{ secrets.GITHUB_TOKEN }}
          firebaseServiceAccount: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
          channelId: live
          projectId: your-firebase-project-id
          target: api-docs
```

Then create `firebase.json` in the root:

```json
{
  "hosting": {
    "target": "api-docs",
    "public": "docs",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**",
      "**/*.md"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/swagger.html"
      }
    ]
  }
}
```

### Deploy to GitHub Pages

Create `.github/workflows/deploy-docs.yml`:

```yaml
name: Deploy API Documentation

on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './docs'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Deploy to Netlify

Create `.github/workflows/deploy-docs.yml`:

```yaml
name: Deploy API Documentation

on:
  push:
    branches:
      - main
    paths:
      - 'docs/**'
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v2
        with:
          publish-dir: './docs'
          production-branch: main
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy API docs from GitHub Actions"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

## Updating the Documentation

When you add or modify API endpoints:

1. Update the OpenAPI spec in `openapi.yaml`
2. Update the embedded spec in `swagger.html` (lines 25-800+)
3. Commit both files
4. The GitHub Action will automatically deploy the changes

### Quick Update Script

You can create a script to keep both files in sync:

```bash
#!/bin/bash
# update-docs.sh

# Convert YAML to JSON and update swagger.html
# This requires yq (https://github.com/mikefarah/yq)

yq eval -o=json docs/openapi.yaml > /tmp/openapi.json

# TODO: Update the spec object in swagger.html with the JSON content
echo "Please manually update the spec object in swagger.html"
echo "Or use a templating tool to automate this"
```

## Interactive Features

The Swagger UI includes:

- **Try it out** - Test API endpoints directly from the browser
- **Code samples** - Generate code snippets in multiple languages
- **Request/Response examples** - View sample payloads
- **Authentication** - Test with Bearer tokens
- **Filter** - Search through endpoints
- **Download** - Export the OpenAPI spec

## Customization

### Change Server URL

Edit the `servers` section in both files:

```yaml
servers:
  - url: https://your-actual-project.supabase.co/functions/v1
    description: Production server
```

### Add Custom Styling

In `swagger.html`, add custom CSS in the `<style>` tag:

```html
<style>
    body {
        margin: 0;
        padding: 0;
    }
    .topbar {
        display: none;
    }
    /* Add your custom styles here */
    .swagger-ui .info .title {
        color: #your-brand-color;
    }
</style>
```

### Add Custom Logo

In `swagger.html`, modify the spec's info section:

```javascript
"info": {
    "title": "Door Knocker Authentication API",
    "x-logo": {
        "url": "https://your-domain.com/logo.png",
        "altText": "Door Knocker Logo"
    },
    // ... rest of info
}
```

## Security Notes

- The documentation is public (don't include sensitive data)
- Don't include real API keys or tokens in examples
- The "Try it out" feature sends real requests to your API
- Consider adding authentication to your docs hosting if needed

## Troubleshooting

### Swagger UI not loading

- Check browser console for errors
- Ensure you're serving over HTTP/HTTPS (not `file://`)
- Verify the OpenAPI spec is valid

### CORS errors when testing

- This is expected when testing from a different origin
- Configure CORS in your Supabase Edge Functions
- Or use a browser extension to disable CORS temporarily

### Spec validation errors

```bash
# Install validator
npm install -g @apidevtools/swagger-cli

# Validate spec
swagger-cli validate docs/openapi.yaml
```

## Additional Resources

- [OpenAPI Specification](https://swagger.io/specification/)
- [Swagger UI Documentation](https://swagger.io/docs/open-source-tools/swagger-ui/)
- [OpenAPI Generator](https://openapi-generator.tech/)
- [Postman OpenAPI Import](https://learning.postman.com/docs/integrations/available-integrations/working-with-openAPI/)
