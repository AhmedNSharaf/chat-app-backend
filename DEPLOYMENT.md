# Deployment Guide

This guide will help you deploy the chat backend to either Vercel or Render.

---

# Deploying to Vercel

## Prerequisites

1. A Vercel account (sign up at https://vercel.com)
2. A MongoDB Atlas account for your database (https://www.mongodb.com/cloud/atlas)
3. Vercel CLI installed (optional): `npm i -g vercel`

## Important Note About Socket.IO on Vercel

⚠️ **WARNING**: Vercel's serverless functions have limitations with WebSocket connections (Socket.IO). For production use with real-time features, consider:
- Using Vercel with an external WebSocket service (like Pusher, Ably)
- Deploying to Render, Railway, or other platforms that support long-running processes
- Using Vercel only for the REST API and hosting Socket.IO separately

## Step 1: Set up MongoDB Atlas

1. Create a free MongoDB cluster at https://www.mongodb.com/cloud/atlas
2. Create a database user with password
3. Whitelist all IP addresses (0.0.0.0/0) in Network Access
4. Get your connection string (it will look like: `mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/chatapp?retryWrites=true&w=majority`)

## Step 2: Deploy to Vercel

### Option A: Using Vercel Dashboard (Recommended)

1. Push your code to GitHub/GitLab/Bitbucket
2. Go to https://vercel.com/dashboard
3. Click "Add New..." → "Project"
4. Import your repository
5. Configure the project:
   - **Framework Preset**: Other
   - **Root Directory**: ./
   - **Build Command**: (leave default)
   - **Output Directory**: (leave default)

6. Add Environment Variables:
   - `MONGODB_URI`: Your MongoDB Atlas connection string
   - `JWT_SECRET`: A secure random string (use a password generator)
   - `NODE_ENV`: production

7. Click "Deploy"

### Option B: Using Vercel CLI

1. Install Vercel CLI if you haven't already:
   ```bash
   npm i -g vercel
   ```

2. Login to Vercel:
   ```bash
   vercel login
   ```

3. Deploy from your project directory:
   ```bash
   vercel
   ```

4. Follow the prompts and add environment variables when asked

5. For production deployment:
   ```bash
   vercel --prod
   ```

## Step 3: Environment Variables

Make sure to set these environment variables in Vercel:

- `MONGODB_URI`: Your MongoDB connection string from Atlas
- `JWT_SECRET`: A secure random string for JWT tokens (min 32 characters recommended)
- `NODE_ENV`: production

## Step 4: Update CORS Settings

After deployment, update the CORS settings in [backend/server.js](backend/server.js) to only allow your frontend domain:

```javascript
const io = socketIo(server, {
  cors: {
    origin: "https://your-frontend-domain.com",
    methods: ["GET", "POST"]
  }
});

app.use(cors({
  origin: "https://your-frontend-domain.com"
}));
```

## Step 5: Test Your Deployment

Once deployed, your API will be available at: `https://your-project-name.vercel.app`

Test the endpoints:
- Health check: `https://your-project-name.vercel.app/api/auth/register`
- Check Vercel deployment logs for any errors

## Limitations on Vercel

- **Serverless Functions**: 10-second execution limit on Hobby plan
- **WebSockets**: Limited support for persistent connections
- **File Uploads**: The `uploads` folder will not persist between deployments
  - Consider using cloud storage (AWS S3, Cloudinary, Vercel Blob)

## File Upload Solution for Vercel

If you need file uploads on Vercel, use Vercel Blob or another cloud storage:

1. Install Vercel Blob SDK:
   ```bash
   npm install @vercel/blob
   ```

2. Update your upload routes to use Vercel Blob instead of local storage

---

# Deploying to Render

This guide will help you deploy the chat backend to Render.

## Prerequisites

1. A Render account (sign up at https://render.com)
2. A MongoDB Atlas account for your database (https://www.mongodb.com/cloud/atlas)
3. Your code pushed to a Git repository (GitHub, GitLab, or Bitbucket)

## Step 1: Set up MongoDB Atlas

1. Create a free MongoDB cluster at https://www.mongodb.com/cloud/atlas
2. Create a database user with password
3. Whitelist all IP addresses (0.0.0.0/0) in Network Access
4. Get your connection string (it will look like: `mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/chatapp?retryWrites=true&w=majority`)

## Step 2: Deploy to Render

### Option A: Using Blueprint (render.yaml) - Recommended

1. Push your code to GitHub/GitLab/Bitbucket
2. Go to https://render.com/dashboard
3. Click "New" → "Blueprint"
4. Connect your repository
5. Render will automatically detect the `render.yaml` file
6. Add the required environment variable:
   - `MONGODB_URI`: Your MongoDB Atlas connection string
7. Click "Apply" to deploy

### Option B: Manual Setup

1. Push your code to GitHub/GitLab/Bitbucket
2. Go to https://render.com/dashboard
3. Click "New" → "Web Service"
4. Connect your repository
5. Configure the service:
   - **Name**: chat-app-backend
   - **Runtime**: Node
   - **Build Command**: `cd backend && npm install`
   - **Start Command**: `cd backend && npm start`
   - **Plan**: Free

6. Add Environment Variables:
   - `NODE_ENV`: production
   - `MONGODB_URI`: Your MongoDB Atlas connection string
   - `JWT_SECRET`: A secure random string (Render can auto-generate this)
   - `PORT`: 10000 (Render's default)

7. Click "Create Web Service"

## Step 3: Environment Variables

Make sure to set these environment variables in Render:

- `MONGODB_URI`: Your MongoDB connection string from Atlas
- `JWT_SECRET`: A secure random string for JWT tokens
- `NODE_ENV`: production
- `PORT`: 10000 (automatically set by Render)

## Step 4: Update CORS Settings (for Production)

After deployment, update the CORS settings in `backend/server.js` to only allow your frontend domain:

```javascript
const io = socketIo(server, {
  cors: {
    origin: "https://your-frontend-domain.com",
    methods: ["GET", "POST"]
  }
});

app.use(cors({
  origin: "https://your-frontend-domain.com"
}));
```

## Step 5: Test Your Deployment

Once deployed, your API will be available at: `https://your-app-name.onrender.com`

Test the health of your API:
- Check logs in Render dashboard
- Test authentication endpoints: `https://your-app-name.onrender.com/api/auth/register`

## Important Notes

- Free tier services on Render spin down after 15 minutes of inactivity
- First request after spin-down may take 30-60 seconds
- For production, consider upgrading to a paid plan for always-on service
- Make sure to never commit `.env` files to your repository
- The `uploads` folder will not persist on Render's free tier - consider using cloud storage (AWS S3, Cloudinary) for file uploads

## Troubleshooting

### Service won't start
- Check logs in Render dashboard
- Verify all environment variables are set correctly
- Ensure MongoDB connection string is correct

### Database connection errors
- Verify MongoDB Atlas is allowing connections from all IPs (0.0.0.0/0)
- Check that database user credentials are correct
- Ensure the database name in the connection string is correct

### CORS errors
- Make sure CORS is configured to allow your frontend domain
- Check that the origin in socket.io configuration matches your frontend

## Monitoring

- Check logs regularly in Render dashboard
- Monitor MongoDB Atlas for database performance
- Set up alerts for errors and downtime
