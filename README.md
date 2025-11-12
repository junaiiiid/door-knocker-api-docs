# API Documentation

This directory contains the API documentation for the Door Knocker Authentication System.

## Files

- **swagger.html** - Standalone Swagger UI documentation (single HTML file)
- **openapi.yaml** - OpenAPI 3.0 specification (for CI/CD and tooling)
- **README.md** - This file

## Viewing the Documentation

### Option 1: Open Locally

Simply open `swagger.html` in your web browser:

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

- The documentation is public - don't include sensitive data
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
