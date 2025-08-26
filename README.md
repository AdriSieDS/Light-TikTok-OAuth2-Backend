# 🚀 Light weight TikTok OAuth2 Server 
[![Node.js](https://img.shields.io/badge/Node.js-18.x-green.svg)](https://nodejs.org/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

created with ❤️ by: [CyberBlueCollarBrandon](https://linktr.ee/CyberBlueCollarBrandon)

A simple, secure, local TikTok OAuth2 server with PKCE support and encrypted token storage. Perfect for integrating TikTok authentication flow into your local workflow.
I created this mostly to workaround the issue of n8n TikTok OAuth2 flow not working and no n8n TikTok node. This project proxy the auth and token management flow locally. 

## ✨ Features

- 🔒 **Proxy Tiktok OAuth2 flow with PKCE Support and token management** - Implements TikTok's required PKCE. Auto token refresh. Secure AES-256 encryption for persistent token storage
- 📱 **Expandable TikTok API Integration** - Add you own API endpoint based on [TikTok for Developer Documentation](https://developers.tiktok.com/doc/overview)

## 📖 Usage

1. **Complete one-time Authentication (OAuth2 Flow)**: Visit `/auth/login`
2. **Use API**: Call `/video/upload` to upload your video. 
3. **Expand**: Making your own API calls using cached tokens. 

### API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/login` | GET | One time OAuth2 flow (thru TikTok) |
| `/auth/callback` | GET | OAuth2 callback handler |
| `/creator-info` | GET | Get TikTok creator information |
| `/user/info` | GET | Get user information with specified fields |
| `/video/direct-post` | POST | Upload video directly to TikTok |
| `/video/upload` | POST | Upload video to TikTok inbox for user edit |
| `/video/status` | GET | Check video upload status |
| `/shutdown` | POST | Gracefully shutdown server |
| `/health` | GET | Server health check |


### Video Upload

#### Example Workflow

When running with Docker, set the `VIDEO_DIR` variable in your `.env` file. That directory on your host is mounted into the container at `/videos`. Use paths within `/videos` when referencing files.

```bash
# One-time OAuth2 authentication
curl http://localhost:7777/auth/login

# Then you can get creator info with: 
curl -X GET http://localhost:7777/creator-info

#  Or upload a video with:
curl -X POST http://localhost:7777/video/upload \
  -H "Content-Type: application/json" \
  -d '{
    "file_path": "/videos/my_video.mp4",
  }'

# and check upload status at (replace with actual publish_id):
curl "http://localhost:7777/video/status?publish_id=abc123def456"
```

#### Behind The Hood

This backend implements video upload function following TikTok's two-step process:

1. **Initialize Upload**: The server calls TikTok's initialization endpoint with video metadata
2. **Upload File**: The video file is uploaded to TikTok's designated URL
3. **Status Tracking**: Use the returned `publish_id` to track upload progress

(Video upload works fine by direct video posting only supports private posting. Reason [here](https://community.n8n.io/t/http-request-node-not-sending-authorization-header-despite-selecting-connected-oauth2-credential-tiktok-api/99963/4) and [here](https://developers.tiktok.com/doc/content-sharing-guidelines#:~:text=Not%20acceptable%3A%20A%20utility%20tool%20to%20help%20upload%20contents%20to%20the%20account(s)%20you%20or%20your%20team%20manages.%20%E2%9D%8C))


### Example API Usage

```javascript
// Get creator info
const response = await fetch('http://localhost:7777/creator-info');
const creatorData = await response.json();
console.log(creatorData);

// Get user info with specific fields
const userInfoResponse = await fetch('http://localhost:7777/user/info?fields=open_id,union_id,avatar_url,display_name,bio_description');
const userData = await userInfoResponse.json();
console.log(userData);

// Upload a video
const uploadResponse = await fetch('http://localhost:7777/video/upload', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    file_path: '/videos/my_video.mp4',
  })
});
const uploadData = await uploadResponse.json();
console.log(uploadData);

// Check video upload status
const statusResponse = await fetch('http://localhost:7777/video/status?publish_id=abc123def456');
const statusData = await statusResponse.json();
console.log(statusData);
```



## 🚀 Quick Start

### 1. Clone & Install

```bash
git clone https://github.com/yourusername/tiktok-oauth2-server.git
cd tiktok-oauth2-server
npm install
```

### 2. Configure TikTok App

Get your **Client Key** and **Client Secret** by setup at [TikTok for Developers](https://developers.tiktok.com/) 


### 3. Setup Environment

```bash
cp env.example .env
```
Then edit `.env` with your credentials:

| Variable | Required | Description |
|----------|----------|-------------|
| `TIKTOK_CLIENT_KEY` | ✅ | Your TikTok app client key |
| `TIKTOK_CLIENT_SECRET` | ✅ | Your TikTok app client secret |
| `TIKTOK_REDIRECT_URI` | ✅ | OAuth2 redirect URI. Where TikTok will send redirect info (eg. http://localhost:7777/auth/callback) |
| `PORT` | ❌ | Server port (default: 7777) |
| `ENCRYPTION_KEY` | ❌ | Encryption key for token storage |
| `VIDEO_DIR` | ❌ | Host directory with videos to mount at `/videos` in the container |


### 4. Start the Server

```bash
# Development mode
npm run dev

# Production mode
npm start
```

Visit `http://localhost:[port]/auth/login` to start the OAuth2 flow!

### Run with Docker Compose

Set your TikTok credentials and optional `VIDEO_DIR` in a `.env` file or directly under the `environment` section of `docker-compose.yml`, then start the service:

```bash
docker-compose up --build
```

The directory specified in `VIDEO_DIR` will be available inside the container at `/videos`. Use that path when providing `file_path` values to the upload endpoint.


### Server Management

#### Shutdown Endpoints

```bash
# Support graceful shutdown thru API (for easy integration in n8n workflow)
curl -X POST http://localhost:7777/shutdown

# Health check
curl http://localhost:7777/health
```

The shutdown endpoint attempts to gracefully terminate the server and any parent processes (like nodemon). It sends a success response before initiating shutdown.


## 🔧 Configuration

### Scopes

The server requests these TikTok scopes:
- `user.info.basic` - Basic user information
- `user.info.profile` - User profile data
- `user.info.stats` - User statistics
- `video.publish` - Video publishing permissions
- `video.upload` - Video upload permissions


## 📁 Project Structure

```
tiktok-oauth2-server/
├── index.js              # Main server file
├── tokenStorage.js       # Encrypted token storage
├── setup.js              # Setup helper (optional)
├── package.json          # Dependencies and scripts
├── env.example           # Environment template
├── README.md             # This file
├── LICENSE               # MIT License
├── .gitignore            # Git ignore rules
└── tokens.encrypted.json # Encrypted tokens (auto-created)
```

## 🆘 Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| "Invalid redirect URI" | Check TikTok app settings |
| "Code verifier invalid" | Ensure PKCE is properly implemented |
| "Token expired" | Server should auto-refresh unless refresh token is expired (valid for a year) |
| "Encryption failed" | Check ENCRYPTION_KEY |


---

⭐ **Star this repository if it helped you!** 