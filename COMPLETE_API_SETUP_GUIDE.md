# Human Design Guide Processing API - Complete Setup Guide

## What This Does

This is a simple API that accepts your template and data, processes it properly (with all formatting preserved), and returns the completed document. You'll call it from Make.com instead of trying to process the document within Make.

## Deployment Options (Pick One)

### Option 1: Render.com (Recommended - Free Tier Available)

1. Go to https://render.com and sign up
2. Click "New +" → "Web Service"
3. Connect your GitHub (or upload these files to a new repo)
4. Configure:
   - **Environment**: Python 3
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `gunicorn api_server:app`
   - **Plan**: Free
5. Click "Create Web Service"
6. Wait for deployment (2-3 minutes)
7. Copy your service URL (e.g., `https://your-service.onrender.com`)

### Option 2: Railway.app (Also Free Tier)

1. Go to https://railway.app
2. Click "Start a New Project" → "Deploy from GitHub"
3. Upload files and configure Python environment
4. Railway will auto-detect and deploy
5. Copy your service URL

### Option 3: Google Cloud Run (More Advanced)

If you need higher reliability/speed, you can deploy to Google Cloud Run. Let me know if you want those instructions.

## Files to Deploy

Upload these 3 files to your hosting service:

1. `api_server.py` (the Flask app)
2. `requirements.txt` (Python dependencies)
3. `Procfile` (optional, for some hosts) containing: `web: gunicorn api_server:app`

## How to Use in Make.com

Once deployed, here's your new Make.com workflow:

### NEW WORKFLOW STRUCTURE:

1. **Webhook** (Module 6) ✓ Keep as is
2. **Text Parsers** (Modules 15-23) ✓ Keep as is
3. **Google Drive: Download Template** - NEW
4. **HTTP: Make a Request** - NEW - Calls your API
5. **Google Drive: Upload File** - NEW
6. **Google Drive: Share Link** ✓ Reconnect
7. **ActiveCampaign** ✓ Keep as is
8. **Slack** ✓ Keep as is

### STEP-BY-STEP MAKE.COM SETUP:

#### Step 1: Upload Your Template to Google Drive

1. Upload `Human_Design_Guide-_Legacy_Phase_1__2_.docx` to your Google Drive
2. Note the File ID from the URL

#### Step 2: Add Google Drive Download Module

After module 23, add:
- **Google Drive > Download a File**
- File ID: [Your template file ID]
- This gives you the template data

#### Step 3: Add HTTP Module to Call API

After the download module, add:
- **HTTP > Make a Request**
- URL: `https://your-service.onrender.com/process-guide`
- Method: POST
- Headers:
  - `Content-Type`: `application/json`
- Body (JSON):

```json
{
  "template_data": "{{base64(previous_module.data)}}",
  "replacements": {
    "{{name}}": "{{6.name}}",
    "{{type}}": "{{6.type}}",
    "{{profile}}": "{{6.profile}}",
    "{{authority}}": "{{6.authority}}",
    "{{section1}}": "{{15.$1}}",
    "{{section2}}": "{{16.$1}}",
    "{{section3}}": "{{17.$1}}",
    "{{section4}}": "{{18.$1}}",
    "{{section5}}": "{{19.$1}}",
    "{{section6}}": "{{20.$1}}",
    "{{section7}}": "{{21.$1}}",
    "{{section8}}": "{{22.$1}}",
    "{{section9}}": "{{23.$1}}"
  }
}
```

#### Step 4: Add Google Drive Upload Module

After the HTTP module:
- **Google Drive > Upload a File**
- File Name: `{{6.name}} - Legacy Guide.docx`
- Data: `{{http_module.data}}`
- Parent Folder: [Your folder ID]
- Convert to Google Docs: Yes

#### Step 5: Reconnect Existing Modules

- Update Share Link module to use the new upload module's file ID
- Keep ActiveCampaign and Slack as they are

## Testing

1. Send a test webhook
2. Check each module's output
3. Verify the final Google Doc has:
   - All content with proper formatting
   - Multiple paragraphs preserved
   - Images and branding intact

## Troubleshooting

**API returns 400 error**
- Check that base64 encoding is working in Make
- Verify JSON structure is correct

**API returns 500 error**
- Check API logs in Render/Railway dashboard
- Verify template file is valid .docx

**Paragraphs not formatted correctly**
- The API preserves Word's paragraph structure
- Each `\n\n` in your text becomes a new paragraph

## Cost

- Render.com Free Tier: 750 hours/month (plenty for your use)
- Railway Free Tier: $5 credit/month
- Both are free for moderate usage

## Support

If you run into issues:
1. Check the API health endpoint: `https://your-service.onrender.com/health`
2. View logs in your hosting dashboard
3. Test the API directly with Postman or curl first

## Example curl test:

```bash
curl -X POST https://your-service.onrender.com/process-guide \
  -H "Content-Type: application/json" \
  -d '{
    "template_url": "https://example.com/template.docx",
    "replacements": {
      "{{name}}": "Test Name",
      "{{section1}}": "Test content"
    }
  }' \
  --output test_output.docx
```

This approach is MUCH cleaner than trying to process documents within Make.com!
