# Classified Ads App - Deployment & Setup Guide

This guide covers how to set up, deploy, and host your React Native / Node.js Classified Ads mobile application using free services.

## 1. Local Development Commands

To run this application locally:

```bash
# Install all dependencies
npm install

# Push database schema (if running on a new PostgreSQL instance)
npm run db:push

# Start both frontend and backend concurrently
npm run dev
```

The application uses `concurrently` (via `tsx`) to start the Vite React development server and the Express backend simultaneously on ports 5000/5001.

## 2. Environment Variables (.env)

Your `.env` file should contain the following for production:

```env
# PostgreSQL connection string (Provided by Replit or your hosting provider)
DATABASE_URL=postgres://user:password@host:port/dbname

# JWT Secret for authentication
JWT_SECRET=your_super_secret_key_here

# (Optional) Node Environment
NODE_ENV=production

# Replit Specific (handled automatically if on Replit)
VITE_API_URL=/api
```

## 3. DevOps & Hosting Architecture (Free Tier)

### Backend Hosting: Render (Free Tier)
1. Push your code to a GitHub repository.
2. Sign up on [Render.com](https://render.com).
3. Click **New +** -> **Web Service**.
4. Connect your GitHub repository.
5. Setup Settings:
   - Environment: `Node`
   - Build Command: `npm install && npm run build`
   - Start Command: `npm run start` (make sure you add a `start` script to package.json pointing to `NODE_ENV=production node dist/index.js` or similar compiled output)
6. Add your Environment Variables (from above) in Render's dashboard.
7. Deploy.

*Note: Render's free tier spins down after 15 minutes of inactivity. The first request after a spin-down might take 30-50 seconds.*

### Database: PostgreSQL (or MongoDB Atlas)
*While you requested MongoDB Atlas, Replit's environment pairs natively with PostgreSQL (and this project uses Drizzle ORM + PostgreSQL as standard for our environment). If you strictly need MongoDB, you can swap the database logic in `server/storage.ts` using `mongoose`.*

If using PostgreSQL outside Replit (e.g., Supabase Free Tier or Neon):
1. Create a free PostgreSQL database.
2. Copy the connection string.
3. Paste it as `DATABASE_URL` in Render.

### Image Hosting: Cloudinary
Currently, images are stored via Base64 for simplicity in this MVP. To integrate Cloudinary:
1. Sign up on Cloudinary.
2. Install their SDK (`npm install cloudinary multer`).
3. Update `server/routes.ts` to accept `multipart/form-data` uploads and forward them to Cloudinary's API.

## 4. GitHub Actions CI/CD Workflow

Create `.github/workflows/deploy.yml` in your repository to automatically deploy to Render on push:

```yaml
name: Deploy to Render

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Render Deploy
        # Note: You need to get your Render Deploy Hook URL from your web service settings
        run: |
          curl -X POST ${{ secrets.RENDER_DEPLOY_HOOK_URL }}
```

Make sure to add `RENDER_DEPLOY_HOOK_URL` to your GitHub Repository Secrets.

## 5. Mobile App Hosting & Expo EAS Build (APK)

Since the frontend is built using standard React for this specific environment (to ensure it runs smoothly on the web as a mobile-first PWA), you can easily wrap this React code into a React Native Expo app, or use it via a WebView.

If building a native React Native Expo app with this backend:
1. Initialize an Expo app: `npx create-expo-app MyAdsApp`
2. Install EAS CLI: `npm install -g eas-cli`
3. Log in: `eas login`
4. Configure EAS: `eas build:configure`
5. To generate a free APK for Android, modify your `eas.json`:

```json
{
  "build": {
    "preview": {
      "android": {
        "buildType": "apk"
      }
    },
    "production": {}
  }
}
```

6. Run the build command:
```bash
eas build -p android --profile preview
```

7. Once the build finishes, Expo will provide a URL to download your `.apk` file for free.

## 6. Hosting Links Structure Example

- **Frontend (Web/PWA):** `https://your-app-name.vercel.app` or `https://your-app.replit.app`
- **Backend API:** `https://your-backend-api.onrender.com`
- **Database:** `postgres://...`
- **Images:** `https://res.cloudinary.com/your-cloud-name/...`
