# Cloudflare Pages Setup Guide

This guide explains how to deploy your DecapCMS-Webstudio project to Cloudflare Pages while maintaining compatibility with Netlify.

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Deployment Steps](#deployment-steps)
- [Configuring Cloudflare Access](#configuring-cloudflare-access)
- [Environment Variables](#environment-variables)
- [Authentication Setup](#authentication-setup)
- [Embedding the CMS](#embedding-the-cms)
- [Troubleshooting](#troubleshooting)

## Overview

This project now supports both Netlify and Cloudflare Pages deployments. The CMS has been configured to live at `/admin/cms/` to support iframe embedding and Cloudflare Access protection.

### Key Features:
- **Dual platform support**: Works on both Netlify and Cloudflare Pages
- **Iframe-ready CMS**: Located at `/admin/cms/` for seamless embedding
- **Cloudflare Access compatible**: Entire `/admin/*` path can be protected
- **Backwards compatibility**: Legacy `/admin/` path still works

## Prerequisites

Before deploying to Cloudflare Pages, ensure you have:

1. A Cloudflare account with Pages enabled
2. Your repository connected to GitHub
3. (Optional) Cloudflare Access configured for your domain

## Deployment Steps

### 1. Connect Your Repository

1. Log in to your [Cloudflare dashboard](https://dash.cloudflare.com/)
2. Select **Pages** from the "add" dropdown button in the top right menu
3. Select **Import an existing Git repository**
4. Authorize Cloudflare to access your GitHub account
5. Select your forked repository and choose **Begin setup**
6. Choose the branch to deploy (e.g., `main` or your customer branch)

### 2. Configure Build Settings

Use these build settings for Cloudflare Pages:

- **Build command**: `npm run build`
- **Build output directory**: `public`
- **Node.js version**: 16 or higher

### 3. Set Environment Variables

In the Cloudflare Pages project settings, add these environment variables:

```env
# Repository configuration
GITHUB_BRANCH=main

# Site information
SITE_TITLE="My Awesome Blog"
SITE_URL=https://your-project.pages.dev

# Media and content settings
MEDIA_FOLDER=images/uploads
PUBLIC_FOLDER=/images/uploads
CONTENT_FOLDER=content/blog

# CMS configuration
PUBLISH_MODE=simple
```

**Important**: Replace `your-project.pages.dev` with your actual Cloudflare Pages URL.

### 4. Deploy

Click **Save and Deploy**. Cloudflare Pages will build and deploy your site.

## Configuring Cloudflare Access

To protect the `/admin/*` path with Cloudflare Access:

### 1. Enable Cloudflare Access

1. In your Cloudflare dashboard, go to **Zero Trust**
2. Navigate to **Access** → **Applications**
3. Click **Add an application**
4. Choose **Self-hosted**

### 2. Configure the Application

- **Application name**: Blog CMS Admin
- **Session duration**: 24 hours (or your preference)
- **Application domain**: `your-domain.com`
- **Path**: `/admin/*`

### 3. Set Access Policies

Create policies to control who can access the CMS:

**Example Policy - Email-based**:
- **Policy name**: Admin Access
- **Action**: Allow
- **Include**: Emails ending in `@yourdomain.com`

**Example Policy - Individual Users**:
- **Policy name**: CMS Editors
- **Action**: Allow
- **Include**: Specific email addresses

### 4. Configure Authentication Methods

Choose your preferred authentication methods:
- Google
- GitHub
- Email OTP
- Or any other supported provider

### 5. Advanced Settings (Optional)

In most cases, you can skip the advanced settings and use the defaults. However, here's when you might need them:

**CORS Settings**: Skip unless you're embedding the CMS from a different domain
- Default settings work fine for same-domain iframe embedding

**Cookie Settings**: Skip for standard setups
- Cloudflare Access handles authentication cookies automatically

**Browser Rendering Settings**: Skip
- Only needed for SSH/VNC terminal access, not applicable for web CMS

**401 Response Settings**: Skip
- Default behavior works well for web applications

Simply click **Save** to proceed with the default settings.

## Authentication Setup

### Single Sign-On with Cloudflare Access (Recommended)

This setup provides a seamless experience - users authenticate once with Cloudflare Access and they're in. No second login screen!

#### How It Works

1. User visits `/admin/cms/`
2. Cloudflare Access prompts for authentication (email, Google, etc.)
3. Once authenticated, they access the CMS directly - **no second login**

#### Setting Up Single Sign-On

1. **Deploy the Git Gateway Worker**

   Deploy the provided `workers/simple-git-gateway.js` as a Cloudflare Worker:
   
   ```bash
   # In your Cloudflare dashboard:
   1. Go to Workers & Pages > Create > Create Worker
   2. Name it (e.g., "cms-git-gateway")
   3. Paste the worker code from workers/simple-git-gateway.js
   4. Add environment variables:
      - GITHUB_TOKEN: Your GitHub personal access token
      - GITHUB_REPO: "owner/repo" format
   5. Deploy the worker
   ```

2. **Update Your Environment Variables**

   Add to your Cloudflare Pages environment variables:
   ```env
   CMS_BACKEND=git-gateway
   GIT_GATEWAY_URL=https://your-worker.workers.dev
   ```

3. **Configure the CMS**

   The config.yml will use these environment variables:
   ```yaml
   backend:
     name: git-gateway
     branch: ${GITHUB_BRANCH}
     identity_url: ${GIT_GATEWAY_URL}
     gateway_url: ${GIT_GATEWAY_URL}
   ```

This creates a truly seamless experience where Cloudflare Access is the only authentication layer.

### Alternative: Netlify for Authentication Only

If you prefer not to deploy a worker, you can still use Netlify purely as an authentication provider while hosting on Cloudflare Pages. This will show a second login screen but requires no custom code.

### Environment Variables for Config

The config files use environment variables that are processed during build:

- `${GITHUB_BRANCH}` - The branch to edit
- `${SITE_URL}` - Your site's URL
- `${MEDIA_FOLDER}` - Where to store media files
- `${PUBLIC_FOLDER}` - Public path for media
- `${PUBLISH_MODE}` - Publishing workflow mode

These are automatically replaced during the build process by the `prebuild` script.

## Embedding the CMS

The CMS is now available at `/admin/cms/` and optimized for iframe embedding.

### In Webstudio

Your Webstudio page at `/admin/edit-blogs` should include:

```html
<iframe 
    src="/admin/cms/"
    style="width: 100%; height: 100vh; border: none;"
    title="Blog Content Management System">
</iframe>
```

### Example Implementation

See `/admin/edit-blogs-example.html` for a complete example of how to:
- Embed the CMS in an iframe
- Handle dynamic height adjustments
- Communicate between parent and iframe

### PostMessage API

The CMS sends these messages to the parent window:

```javascript
// Height updates
{
  type: 'cms-height',
  height: 1200 // in pixels
}

// Authentication status
{
  type: 'cms-auth',
  authenticated: true/false
}
```

## Troubleshooting

### Build Failures

If your build fails on Cloudflare Pages:

1. Check the build logs for specific errors
2. Ensure all environment variables are set correctly
3. Verify Node.js version compatibility

### CMS Not Loading

If the CMS doesn't load at `/admin/cms/`:

1. Check browser console for errors
2. Verify the `_redirects` file is in the `public` directory
3. Ensure JavaScript is enabled
4. Check Cloudflare Access policies if enabled

### Authentication Issues

For authentication problems:

1. Verify your Git provider credentials
2. Check Cloudflare Access configuration
3. Ensure cookies are enabled for your domain
4. Try accessing the CMS directly (not in iframe) for testing

### Content Not Updating

If content changes aren't appearing:

1. Check if the build was triggered
2. Verify branch protection rules
3. Ensure proper Git permissions
4. Clear Cloudflare cache if needed

### Iframe Issues

For iframe embedding problems:

1. Check X-Frame-Options headers
2. Verify Content Security Policy
3. Ensure same-origin policy compliance
4. Test the CMS URL directly first

## Platform Differences

### Netlify Features
- Netlify Identity (built-in authentication)
- Serverless functions for dynamic config
- Automatic Git Gateway setup

### Cloudflare Pages Features
- Cloudflare Access for authentication
- Global CDN with better performance
- Built-in DDoS protection
- No function size limits

### What Works on Both
- Static site generation
- Git-based content management
- Media uploads
- All CMS features

### What Requires Adaptation
- Authentication method
- Serverless functions (use Workers on Cloudflare)
- Environment variable names (some differences)

## Recommended Setup for Customer Sites

For the best customer experience with single sign-on:

1. **Deploy the Git Gateway Worker** (one-time setup)
   - Copy `workers/simple-git-gateway.js` to a new Cloudflare Worker
   - Add your GitHub token and repo as environment variables
   - Deploy the worker

2. **Configure Cloudflare Pages** with these environment variables:
   ```env
   GITHUB_BRANCH=main
   SITE_URL=https://your-site.pages.dev
   GIT_GATEWAY_URL=https://your-worker.workers.dev
   MEDIA_FOLDER=images/uploads
   PUBLIC_FOLDER=/images/uploads
   CONTENT_FOLDER=content/blog
   PUBLISH_MODE=simple
   ```

3. **Set up Cloudflare Access**
   - Protect `/admin/*` path
   - Use email-based authentication
   - Add your customers' emails to the access list

This gives you:
- ✅ **Single sign-on** - One login via Cloudflare Access
- ✅ **No GitHub accounts needed** - Customers use their email
- ✅ **Fast global CDN** - Cloudflare's network
- ✅ **Secure** - Zero Trust protection
- ✅ **Simple** - No complex identity providers

## Best Practices

1. **Test locally first**: Use `npm run dev` to test changes
2. **Use preview deployments**: Both platforms offer preview URLs
3. **Monitor build times**: Optimize if builds take too long
4. **Cache static assets**: Configure appropriate cache headers
5. **Regular backups**: Keep your content backed up in Git
6. **Dual-platform compatibility**: Always test changes on both Netlify and Cloudflare Pages

## Next Steps

After successful deployment:

1. Test the CMS at `/admin/cms/`
2. Configure Cloudflare Access if needed
3. Update your Webstudio integration
4. Train content editors on the new URL
5. Monitor performance and usage

For more help, see:
- [Cloudflare Pages Documentation](https://developers.cloudflare.com/pages/)
- [Decap CMS Documentation](https://decapcms.org/docs/)
- [Troubleshooting Guide](./troubleshooting.md)