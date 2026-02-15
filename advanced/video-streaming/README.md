# Video Streaming Platform

<p align="center">
  <img src="https://images.unsplash.com/photo-1574375927938-d5a98e8ffe85?w=600&q=80" alt="Video Streaming" width="600">
</p>

## Overview

**Difficulty:** Advanced  
**Estimated Time:** 25–35 hours  
**Prerequisites:** Node.js, FFmpeg, AWS S3, Redis, WebSockets, React/Vue  

A **Video Streaming Platform** is a full-featured video hosting service similar to YouTube or Vimeo that allows users to:

- Upload and watch videos with adaptive quality
- Experience smooth playback across different network speeds
- Discover content through AI-powered recommendations
- Interact with content (likes, comments, subscriptions)
- Monetize through subscriptions or ads

This project covers video processing, adaptive streaming protocols (HLS/DASH), CDN distribution, recommendation algorithms, and scalable architecture design.

---

## Features

### Must Have

- User authentication and profiles
- Video upload (drag-and-drop, progress tracking)
- Automatic video transcoding to multiple resolutions (360p, 480p, 720p, 1080p)
- Adaptive streaming with HLS (HTTP Live Streaming)
- Custom video player with controls (play, pause, volume, quality selector, fullscreen)
- Video metadata (title, description, tags, thumbnail)
- Video library and search functionality
- View counter and basic analytics
- Comments system
- Like/dislike functionality
- Responsive design for mobile and desktop

### Nice to Have

- Subscription system with payment integration
- Channel creation and management
- Video playlists
- AI-based recommendation engine
- Live streaming with RTMP
- Watch history and continue watching
- Subtitles/closed captions support (WebVTT)
- Video trimming and editing tools
- CDN integration for global distribution
- DRM and content protection
- Advanced analytics dashboard (watch time, retention, demographics)
- Video monetization (ads, sponsorships)
- Social features (share, embed)
- Notifications system
- Multi-language support
- Dark mode

---

## Learning Objectives

By building this project, you will learn:

- Video encoding and transcoding with **FFmpeg**
- Adaptive streaming protocols (**HLS**, **DASH**)
- Cloud storage integration (**AWS S3**, **Google Cloud Storage**)
- CDN implementation for video delivery
- Background job processing with **queues** (Bull, BullMQ)
- Video processing pipelines
- Recommendation algorithms (collaborative filtering, content-based)
- Real-time features with **WebSockets**
- Video player customization
- Handling large file uploads
- Database design for video metadata
- Scalability patterns for high-traffic applications
- DRM and content protection
- Analytics and metrics tracking

---

## Project Concepts

This project simulates a production-grade video platform and covers:

- **Video Processing Pipeline:** Upload → Validate → Transcode → Store → Serve
- **Adaptive Bitrate Streaming:** Automatically adjust quality based on network conditions
- **Microservices Architecture:** Separate services for upload, processing, streaming, recommendations
- **Asynchronous Processing:** Use job queues for time-consuming operations
- **Content Delivery:** Use CDN to reduce latency and bandwidth costs
- **Scalability:** Design for millions of videos and concurrent users

---

## Step-by-Step Implementation

### Step 1: Project Setup and Architecture

Plan the system architecture:

```
┌─────────────┐
│   Client    │ (React/Vue video player)
└──────┬──────┘
       │
       ├─────────────────────────────────────┐
       │                                     │
┌──────▼──────┐                     ┌───────▼──────┐
│   API       │                     │    CDN       │
│  Gateway    │                     │  (CloudFront)│
└──────┬──────┘                     └───────┬──────┘
       │                                     │
       ├─────────┬──────────┬────────────────┘
       │         │          │
┌──────▼─────┐ ┌▼─────────┐ ┌▼──────────────┐
│  Upload    │ │Processing│ │   Streaming   │
│  Service   │ │ Service  │ │   Service     │
└──────┬─────┘ └┬─────────┘ └┬──────────────┘
       │        │            │
       │   ┌────▼────────────▼───┐
       │   │   Job Queue (Redis)│
       │   └────────────────────┬┘
       │                        │
┌──────▼────────────────────────▼─┐
│     Database (PostgreSQL)       │
└─────────────────────────────────┘
       │
┌──────▼──────────────────────────┐
│   Object Storage (AWS S3)       │
│  - Raw Videos                   │
│  - Transcoded Videos (HLS)      │
│  - Thumbnails                   │
└─────────────────────────────────┘
```

Install dependencies:

```bash
npm init -y
npm install express multer bull ffmpeg-static fluent-ffmpeg
npm install aws-sdk redis socket.io mongoose jsonwebtoken bcryptjs
npm install dotenv cors helmet express-rate-limit
npm install @aws-sdk/client-s3 @aws-sdk/lib-storage
```

### Step 2: Video Upload Service

Create the upload endpoint with multipart handling:

```javascript
// services/uploadService.js
const multer = require('multer');
const path = require('path');
const fs = require('fs').promises;
const { S3Client } = require('@aws-sdk/client-s3');
const { Upload } = require('@aws-sdk/lib-storage');
const crypto = require('crypto');

// Configure S3
const s3Client = new S3Client({
    region: process.env.AWS_REGION,
    credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
    }
});

// Configure Multer for temporary storage
const upload = multer({
    dest: 'temp/uploads/',
    limits: {
        fileSize: 5 * 1024 * 1024 * 1024 // 5GB limit
    },
    fileFilter: (req, file, cb) => {
        const allowedFormats = ['.mp4', '.mov', '.avi', '.mkv', '.webm'];
        const ext = path.extname(file.originalname).toLowerCase();
        
        if (allowedFormats.includes(ext)) {
            cb(null, true);
        } else {
            cb(new Error('Invalid video format. Allowed: MP4, MOV, AVI, MKV, WebM'));
        }
    }
});

class UploadService {
    async uploadVideo(file, metadata, userId) {
        try {
            // Generate unique video ID
            const videoId = crypto.randomBytes(16).toString('hex');
            
            // Upload raw video to S3
            const rawKey = `raw/${videoId}/${file.originalname}`;
            const uploadParams = {
                Bucket: process.env.S3_BUCKET,
                Key: rawKey,
                Body: require('fs').createReadStream(file.path),
                ContentType: file.mimetype
            };
            
            const s3Upload = new Upload({
                client: s3Client,
                params: uploadParams
            });
            
            // Track upload progress
            s3Upload.on('httpUploadProgress', (progress) => {
                const percentage = (progress.loaded / progress.total) * 100;
                console.log(`Upload progress: ${percentage.toFixed(2)}%`);
                // Emit progress via WebSocket to client
            });
            
            await s3Upload.done();
            
            // Clean up temporary file
            await fs.unlink(file.path);
            
            // Save video metadata to database
            const video = await this.createVideoRecord({
                videoId,
                userId,
                title: metadata.title,
                description: metadata.description,
                tags: metadata.tags,
                rawVideoUrl: `s3://${process.env.S3_BUCKET}/${rawKey}`,
                status: 'processing',
                uploadedAt: new Date()
            });
            
            // Add to processing queue
            await this.queueVideoProcessing(videoId);
            
            return {
                videoId,
                status: 'processing',
                message: 'Video uploaded successfully and queued for processing'
            };
            
        } catch (error) {
            console.error('Upload error:', error);
            throw error;
        }
    }
    
    async createVideoRecord(data) {
        const Video = require('../models/Video');
        return await Video.create(data);
    }
    
    async queueVideoProcessing(videoId) {
        const { videoProcessingQueue } = require('./queueService');
        await videoProcessingQueue.add('transcode', { videoId });
    }
}

module.exports = new UploadService();
```

### Step 3: Video Processing Service with FFmpeg

Implement video transcoding to multiple resolutions:

```javascript
// services/processingService.js
const ffmpeg = require('fluent-ffmpeg');
const ffmpegStatic = require('ffmpeg-static');
const path = require('path');
const fs = require('fs').promises;
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');

ffmpeg.setFfmpegPath(ffmpegStatic);

const s3Client = new S3Client({
    region: process.env.AWS_REGION,
    credentials: {
        accessKeyId: process.env.AWS_ACCESS_KEY_ID,
        secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
    }
});

class ProcessingService {
    constructor() {
        this.resolutions = [
            { name: '360p', width: 640, height: 360, bitrate: '800k' },
            { name: '480p', width: 854, height: 480, bitrate: '1400k' },
            { name: '720p', width: 1280, height: 720, bitrate: '2800k' },
            { name: '1080p', width: 1920, height: 1080, bitrate: '5000k' }
        ];
    }
    
    async processVideo(videoId) {
        try {
            console.log(`Starting processing for video: ${videoId}`);
            
            // Get video record from database
            const video = await this.getVideoRecord(videoId);
            
            // Download raw video from S3 to temp directory
            const tempDir = path.join(__dirname, '../temp/processing', videoId);
            await fs.mkdir(tempDir, { recursive: true });
            
            const rawVideoPath = path.join(tempDir, 'raw.mp4');
            await this.downloadFromS3(video.rawVideoUrl, rawVideoPath);
            
            // Get video metadata
            const metadata = await this.getVideoMetadata(rawVideoPath);
            console.log('Video metadata:', metadata);
            
            // Generate thumbnail
            const thumbnailPath = await this.generateThumbnail(rawVideoPath, tempDir);
            await this.uploadThumbnail(videoId, thumbnailPath);
            
            // Transcode to multiple resolutions with HLS
            await this.transcodeToHLS(rawVideoPath, tempDir, videoId);
            
            // Update video status in database
            await this.updateVideoStatus(videoId, 'ready');
            
            // Clean up temp files
            await fs.rm(tempDir, { recursive: true });
            
            console.log(`Processing completed for video: ${videoId}`);
            
        } catch (error) {
            console.error('Processing error:', error);
            await this.updateVideoStatus(videoId, 'failed', error.message);
            throw error;
        }
    }
    
    async transcodeToHLS(inputPath, outputDir, videoId) {
        const hlsDir = path.join(outputDir, 'hls');
        await fs.mkdir(hlsDir, { recursive: true });
        
        // Process each resolution
        for (const resolution of this.resolutions) {
            await this.transcodeResolution(inputPath, hlsDir, resolution);
        }
        
        // Create master playlist
        await this.createMasterPlaylist(hlsDir, videoId);
        
        // Upload all HLS files to S3
        await this.uploadHLSToS3(hlsDir, videoId);
    }
    
    transcodeResolution(inputPath, outputDir, resolution) {
        return new Promise((resolve, reject) => {
            const outputPath = path.join(outputDir, `${resolution.name}`);
            
            ffmpeg(inputPath)
                .outputOptions([
                    `-vf scale=${resolution.width}:${resolution.height}`,
                    `-c:v libx264`,
                    `-b:v ${resolution.bitrate}`,
                    `-c:a aac`,
                    `-b:a 128k`,
                    `-hls_time 10`,
                    `-hls_playlist_type vod`,
                    `-hls_segment_filename ${outputPath}_%03d.ts`
                ])
                .output(`${outputPath}.m3u8`)
                .on('start', (cmd) => {
                    console.log(`Starting transcode for ${resolution.name}:`, cmd);
                })
                .on('progress', (progress) => {
                    console.log(`${resolution.name} progress: ${progress.percent}%`);
                })
                .on('end', () => {
                    console.log(`${resolution.name} transcode complete`);
                    resolve();
                })
                .on('error', (err) => {
                    console.error(`${resolution.name} transcode error:`, err);
                    reject(err);
                })
                .run();
        });
    }
    
    async createMasterPlaylist(hlsDir, videoId) {
        const masterPlaylist = `#EXTM3U
#EXT-X-VERSION:3

#EXT-X-STREAM-INF:BANDWIDTH=800000,RESOLUTION=640x360
360p.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=1400000,RESOLUTION=854x480
480p.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=2800000,RESOLUTION=1280x720
720p.m3u8

#EXT-X-STREAM-INF:BANDWIDTH=5000000,RESOLUTION=1920x1080
1080p.m3u8
`;
        
        const masterPath = path.join(hlsDir, 'master.m3u8');
        await fs.writeFile(masterPath, masterPlaylist);
    }
    
    async uploadHLSToS3(hlsDir, videoId) {
        const files = await fs.readdir(hlsDir, { recursive: true });
        
        for (const file of files) {
            const filePath = path.join(hlsDir, file);
            const stat = await fs.stat(filePath);
            
            if (stat.isFile()) {
                const key = `videos/${videoId}/hls/${file}`;
                const fileContent = await fs.readFile(filePath);
                
                await s3Client.send(new PutObjectCommand({
                    Bucket: process.env.S3_BUCKET,
                    Key: key,
                    Body: fileContent,
                    ContentType: file.endsWith('.m3u8') ? 'application/vnd.apple.mpegurl' : 'video/MP2T'
                }));
                
                console.log(`Uploaded: ${key}`);
            }
        }
    }
    
    generateThumbnail(videoPath, outputDir) {
        return new Promise((resolve, reject) => {
            const thumbnailPath = path.join(outputDir, 'thumbnail.jpg');
            
            ffmpeg(videoPath)
                .screenshots({
                    timestamps: ['10%'],
                    filename: 'thumbnail.jpg',
                    folder: outputDir,
                    size: '1280x720'
                })
                .on('end', () => resolve(thumbnailPath))
                .on('error', reject);
        });
    }
    
    getVideoMetadata(videoPath) {
        return new Promise((resolve, reject) => {
            ffmpeg.ffprobe(videoPath, (err, metadata) => {
                if (err) reject(err);
                else resolve(metadata);
            });
        });
    }
    
    async downloadFromS3(s3Url, localPath) {
        // Implementation for downloading from S3
        // Parse s3:// URL and use GetObjectCommand
    }
    
    async uploadThumbnail(videoId, thumbnailPath) {
        const fileContent = await fs.readFile(thumbnailPath);
        const key = `videos/${videoId}/thumbnail.jpg`;
        
        await s3Client.send(new PutObjectCommand({
            Bucket: process.env.S3_BUCKET,
            Key: key,
            Body: fileContent,
            ContentType: 'image/jpeg'
        }));
    }
    
    async getVideoRecord(videoId) {
        const Video = require('../models/Video');
        return await Video.findOne({ videoId });
    }
    
    async updateVideoStatus(videoId, status, error = null) {
        const Video = require('../models/Video');
        return await Video.updateOne(
            { videoId },
            { status, processingError: error, processedAt: new Date() }
        );
    }
}

module.exports = new ProcessingService();
```

### Step 4: Custom Video Player

Build a feature-rich HTML5 video player with HLS support:

```html
<!-- client/VideoPlayer.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Video Player</title>
    <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
    <style>
        .video-container {
            position: relative;
            width: 100%;
            max-width: 1280px;
            margin: 0 auto;
            background: #000;
        }
        
        .video-container video {
            width: 100%;
            display: block;
        }
        
        .video-controls {
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            background: linear-gradient(transparent, rgba(0,0,0,0.8));
            padding: 20px;
            opacity: 0;
            transition: opacity 0.3s;
        }
        
        .video-container:hover .video-controls {
            opacity: 1;
        }
        
        .controls-row {
            display: flex;
            align-items: center;
            gap: 15px;
        }
        
        .progress-bar {
            flex: 1;
            height: 5px;
            background: rgba(255,255,255,0.3);
            border-radius: 5px;
            cursor: pointer;
            position: relative;
        }
        
        .progress-filled {
            height: 100%;
            background: #ff0000;
            border-radius: 5px;
            transition: width 0.1s;
        }
        
        .control-btn {
            background: none;
            border: none;
            color: white;
            font-size: 20px;
            cursor: pointer;
            padding: 5px 10px;
        }
        
        .time-display {
            color: white;
            font-size: 14px;
        }
        
        .quality-selector {
            background: rgba(0,0,0,0.8);
            border: 1px solid rgba(255,255,255,0.3);
            color: white;
            padding: 5px 10px;
            border-radius: 5px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="video-container">
        <video id="video" preload="metadata"></video>
        
        <div class="video-controls">
            <div class="progress-bar" id="progressBar">
                <div class="progress-filled" id="progressFilled"></div>
            </div>
            
            <div class="controls-row">
                <button class="control-btn" id="playBtn">▶</button>
                <span class="time-display" id="timeDisplay">0:00 / 0:00</span>
                <button class="control-btn" id="muteBtn">🔊</button>
                <input type="range" id="volumeSlider" min="0" max="100" value="100" style="width: 80px">
                <select class="quality-selector" id="qualitySelector">
                    <option value="auto">Auto</option>
                </select>
                <button class="control-btn" id="fullscreenBtn">⛶</button>
            </div>
        </div>
    </div>
    
    <script>
        class VideoPlayer {
            constructor(videoElement, hlsUrl) {
                this.video = videoElement;
                this.hlsUrl = hlsUrl;
                this.hls = null;
                
                this.setupHLS();
                this.setupControls();
                this.setupAnalytics();
            }
            
            setupHLS() {
                if (Hls.isSupported()) {
                    this.hls = new Hls({
                        enableWorker: true,
                        lowLatencyMode: false,
                        backBufferLength: 90
                    });
                    
                    this.hls.loadSource(this.hlsUrl);
                    this.hls.attachMedia(this.video);
                    
                    this.hls.on(Hls.Events.MANIFEST_PARSED, () => {
                        console.log('HLS manifest loaded');
                        this.populateQualitySelector();
                    });
                    
                    this.hls.on(Hls.Events.ERROR, (event, data) => {
                        console.error('HLS error:', data);
                        if (data.fatal) {
                            this.handleFatalError(data);
                        }
                    });
                    
                } else if (this.video.canPlayType('application/vnd.apple.mpegurl')) {
                    // Native HLS support (Safari)
                    this.video.src = this.hlsUrl;
                }
            }
            
            setupControls() {
                const playBtn = document.getElementById('playBtn');
                const muteBtn = document.getElementById('muteBtn');
                const volumeSlider = document.getElementById('volumeSlider');
                const progressBar = document.getElementById('progressBar');
                const progressFilled = document.getElementById('progressFilled');
                const timeDisplay = document.getElementById('timeDisplay');
                const fullscreenBtn = document.getElementById('fullscreenBtn');
                const qualitySelector = document.getElementById('qualitySelector');
                
                // Play/Pause
                playBtn.addEventListener('click', () => {
                    if (this.video.paused) {
                        this.video.play();
                        playBtn.textContent = '⏸';
                    } else {
                        this.video.pause();
                        playBtn.textContent = '▶';
                    }
                });
                
                // Progress bar
                this.video.addEventListener('timeupdate', () => {
                    const percent = (this.video.currentTime / this.video.duration) * 100;
                    progressFilled.style.width = percent + '%';
                    
                    timeDisplay.textContent = 
                        `${this.formatTime(this.video.currentTime)} / ${this.formatTime(this.video.duration)}`;
                });
                
                progressBar.addEventListener('click', (e) => {
                    const rect = progressBar.getBoundingClientRect();
                    const percent = (e.clientX - rect.left) / rect.width;
                    this.video.currentTime = percent * this.video.duration;
                });
                
                // Volume
                volumeSlider.addEventListener('input', (e) => {
                    this.video.volume = e.target.value / 100;
                    this.video.muted = e.target.value == 0;
                });
                
                muteBtn.addEventListener('click', () => {
                    this.video.muted = !this.video.muted;
                    muteBtn.textContent = this.video.muted ? '🔇' : '🔊';
                });
                
                // Fullscreen
                fullscreenBtn.addEventListener('click', () => {
                    if (document.fullscreenElement) {
                        document.exitFullscreen();
                    } else {
                        document.querySelector('.video-container').requestFullscreen();
                    }
                });
                
                // Quality selector
                qualitySelector.addEventListener('change', (e) => {
                    if (e.target.value === 'auto') {
                        this.hls.currentLevel = -1; // Auto
                    } else {
                        this.hls.currentLevel = parseInt(e.target.value);
                    }
                });
            }
            
            populateQualitySelector() {
                const selector = document.getElementById('qualitySelector');
                
                this.hls.levels.forEach((level, index) => {
                    const option = document.createElement('option');
                    option.value = index;
                    option.textContent = `${level.height}p`;
                    selector.appendChild(option);
                });
            }
            
            setupAnalytics() {
                let watchTime = 0;
                let lastTime = 0;
                
                this.video.addEventListener('timeupdate', () => {
                    if (this.video.currentTime > lastTime) {
                        watchTime += this.video.currentTime - lastTime;
                    }
                    lastTime = this.video.currentTime;
                    
                    // Send analytics every 30 seconds
                    if (Math.floor(watchTime) % 30 === 0) {
                        this.sendAnalytics({
                            videoId: this.videoId,
                            watchTime: watchTime,
                            currentTime: this.video.currentTime,
                            quality: this.hls.currentLevel
                        });
                    }
                });
            }
            
            formatTime(seconds) {
                const mins = Math.floor(seconds / 60);
                const secs = Math.floor(seconds % 60);
                return `${mins}:${secs.toString().padStart(2, '0')}`;
            }
            
            async sendAnalytics(data) {
                try {
                    await fetch('/api/analytics/watch', {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(data)
                    });
                } catch (error) {
                    console.error('Analytics error:', error);
                }
            }
            
            handleFatalError(data) {
                switch (data.type) {
                    case Hls.ErrorTypes.NETWORK_ERROR:
                        console.error('Network error, trying to recover...');
                        this.hls.startLoad();
                        break;
                    case Hls.ErrorTypes.MEDIA_ERROR:
                        console.error('Media error, trying to recover...');
                        this.hls.recoverMediaError();
                        break;
                    default:
                        console.error('Fatal error, cannot recover');
                        this.hls.destroy();
                        break;
                }
            }
        }
        
        // Initialize player
        const videoUrl = 'https://cdn.example.com/videos/VIDEO_ID/hls/master.m3u8';
        const player = new VideoPlayer(document.getElementById('video'), videoUrl);
    </script>
</body>
</html>
```

### Step 5: Recommendation Engine

Implement AI-based video recommendations:

```javascript
// services/recommendationService.js
class RecommendationService {
    constructor() {
        this.weights = {
            viewHistory: 0.3,
            likes: 0.2,
            tags: 0.2,
            channel: 0.15,
            trending: 0.15
        };
    }
    
    async getRecommendations(userId, limit = 20) {
        try {
            // Get user's watch history
            const watchHistory = await this.getUserWatchHistory(userId);
            
            // Get user's liked videos
            const likedVideos = await this.getUserLikedVideos(userId);
            
            // Get user's subscribed channels
            const subscriptions = await this.getUserSubscriptions(userId);
            
            // Calculate recommendations
            const recommendations = await this.calculateRecommendations({
                watchHistory,
                likedVideos,
                subscriptions,
                limit
            });
            
            return recommendations;
            
        } catch (error) {
            console.error('Recommendation error:', error);
            return await this.getTrendingVideos(limit);
        }
    }
    
    async calculateRecommendations(data) {
        const { watchHistory, likedVideos, subscriptions, limit } = data;
        
        // Extract tags from watched videos
        const userTags = this.extractTags(watchHistory);
        
        // Find similar videos based on collaborative filtering
        const collaborative = await this.collaborativeFiltering(watchHistory);
        
        // Find similar videos based on content
        const contentBased = await this.contentBasedFiltering(userTags);
        
        // Get videos from subscribed channels
        const fromSubscriptions = await this.getVideosFromChannels(subscriptions);
        
        // Get trending videos
        const trending = await this.getTrendingVideos(limit);
        
        // Combine and score
        const scored = this.combineAndScore({
            collaborative,
            contentBased,
            fromSubscriptions,
            trending
        });
        
        // Remove already watched
        const filtered = scored.filter(v => 
            !watchHistory.find(w => w.videoId === v.videoId)
        );
        
        // Sort by score and return top N
        return filtered
            .sort((a, b) => b.score - a.score)
            .slice(0, limit);
    }
    
    async collaborativeFiltering(watchHistory) {
        // Find users with similar watch history
        const similarUsers = await this.findSimilarUsers(watchHistory);
        
        // Get videos watched by similar users
        const videos = [];
        for (const user of similarUsers) {
            const userVideos = await this.getUserWatchHistory(user.userId);
            videos.push(...userVideos);
        }
        
        // Count occurrences and return most common
        const counts = {};
        videos.forEach(v => {
            counts[v.videoId] = (counts[v.videoId] || 0) + 1;
        });
        
        return Object.entries(counts)
            .map(([videoId, count]) => ({ videoId, score: count }))
            .sort((a, b) => b.score - a.score)
            .slice(0, 50);
    }
    
    async contentBasedFiltering(userTags) {
        // Find videos with similar tags
        const Video = require('../models/Video');
        
        const videos = await Video.find({
            tags: { $in: userTags },
            status: 'ready'
        })
        .limit(50)
        .lean();
        
        return videos.map(v => ({
            videoId: v.videoId,
            score: this.calculateTagSimilarity(v.tags, userTags)
        }));
    }
    
    extractTags(videos) {
        const tags = new Set();
        videos.forEach(v => {
            v.tags?.forEach(tag => tags.add(tag));
        });
        return Array.from(tags);
    }
    
    calculateTagSimilarity(tags1, tags2) {
        const intersection = tags1.filter(t => tags2.includes(t));
        const union = [...new Set([...tags1, ...tags2])];
        return intersection.length / union.length; // Jaccard similarity
    }
    
    combineAndScore(sources) {
        const videos = new Map();
        
        // Add scores from each source
        Object.entries(sources).forEach(([source, items]) => {
            const weight = this.weights[source] || 0.1;
            
            items.forEach(item => {
                const existing = videos.get(item.videoId) || { videoId: item.videoId, score: 0 };
                existing.score += item.score * weight;
                videos.set(item.videoId, existing);
            });
        });
        
        return Array.from(videos.values());
    }
    
    async getUserWatchHistory(userId) {
        const WatchHistory = require('../models/WatchHistory');
        return await WatchHistory.find({ userId }).sort({ watchedAt: -1 }).limit(100);
    }
    
    async getUserLikedVideos(userId) {
        const Like = require('../models/Like');
        return await Like.find({ userId, type: 'like' });
    }
    
    async getUserSubscriptions(userId) {
        const Subscription = require('../models/Subscription');
        return await Subscription.find({ userId });
    }
    
    async getTrendingVideos(limit) {
        const Video = require('../models/Video');
        return await Video.find({ status: 'ready' })
            .sort({ views: -1, createdAt: -1 })
            .limit(limit);
    }
    
    async getVideosFromChannels(subscriptions) {
        const Video = require('../models/Video');
        const channelIds = subscriptions.map(s => s.channelId);
        
        return await Video.find({
            userId: { $in: channelIds },
            status: 'ready'
        })
        .sort({ createdAt: -1 })
        .limit(50);
    }
    
    async findSimilarUsers(watchHistory) {
        // Implementation of user similarity calculation
        // Using cosine similarity or Pearson correlation
        return [];
    }
}

module.exports = new RecommendationService();
```

## Technology Stack

### Backend

- **Node.js + Express** - API server
- **FFmpeg** - Video transcoding
- **Bull/BullMQ** - Job queue for async processing
- **Redis** - Queue storage and caching
- **PostgreSQL/MongoDB** - Video metadata storage
- **AWS S3** - Video storage
- **AWS CloudFront** - CDN for video delivery

### Frontend

- **React/Vue** - UI framework
- **HLS.js** - HLS playback support
- **Video.js** - Alternative video player
- **Socket.io** - Real-time features

### Infrastructure

- **Docker** - Containerization
- **Kubernetes** - Orchestration
- **Nginx** - Reverse proxy
- **Elasticsearch** - Video search
- **Prometheus + Grafana** - Monitoring

## Testing Checklist

- [ ] Video upload works with progress tracking
- [ ] Videos transcode to all target resolutions
- [ ] HLS playback works across browsers
- [ ] Quality switching is seamless
- [ ] Player controls work correctly
- [ ] Analytics are recorded accurately
- [ ] Recommendations are relevant
- [ ] Search returns correct results
- [ ] Comments and likes persist
- [ ] Subscriptions work correctly
- [ ] Mobile responsive design
- [ ] CDN delivers videos efficiently

## Additional Challenges

### Easy Extensions

- Add video categories and genres
- Implement playlist creation
- Add watch later functionality
- Create video sharing links
- Implement theater mode

### Medium Extensions

- Add live streaming with RTMP
- Implement subtitles/closed captions
- Create video trimming tool
- Add collaborative playlists
- Build channel analytics dashboard
- Implement video monetization
- Add content moderation tools

### Hard Extensions

- Implement DRM with Widevine/FairPlay
- Add 360° video support
- Create automatic subtitle generation with ML
- Build content ID system (copyright detection)
- Implement adaptive bitrate with DASH
- Add interactive video features (polls, cards)
- Create video recommendation ML model from scratch
- Build distributed transcoding with Kubernetes

## Common Pitfalls

1. **Storage Costs:** Videos consume massive storage. Implement lifecycle policies to delete raw videos after transcoding and use compression.

2. **Transcoding Time:** Large videos take hours to process. Use multiple worker processes and show realistic ETAs to users.

3. **Bandwidth Costs:** Video streaming is expensive. Use CDN caching aggressively and consider peer-to-peer options.

4. **Browser Compatibility:** Not all browsers support HLS natively. Always include HLS.js as fallback.

5. **Buffering Issues:** Optimize segment duration (10s is standard) and implement adaptive bitrate switching properly.

## Resources

### Documentation

- [HLS Specification](https://datatracker.ietf.org/doc/html/rfc8216)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)
- [HLS.js GitHub](https://github.com/video-dev/hls.js/)
- [AWS S3 Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html)

### Tutorials

- [Building a Video Platform](https://www.youtube.com/results?search_query=build+video+streaming+platform)
- [FFmpeg Tutorial](https://github.com/leandromoreira/ffmpeg-libav-tutorial)
- [Adaptive Streaming Guide](https://developer.mozilla.org/en-US/docs/Web/Media/Streaming)

### Tools

- [HandBrake](https://handbrake.fr/) - Video transcoding
- [Video.js](https://videojs.com/) - HTML5 video player
- [Mux](https://www.mux.com/) - Video infrastructure platform

## Submission Checklist

- [ ] Video upload with chunked transfer
- [ ] Multi-resolution transcoding
- [ ] HLS adaptive streaming
- [ ] Custom video player with full controls
- [ ] User authentication and profiles
- [ ] Search functionality
- [ ] Recommendation system
- [ ] Comments and likes
- [ ] Analytics dashboard
- [ ] Responsive design
- [ ] CDN integration
- [ ] Comprehensive documentation

## Portfolio Tips

- **Show Scale:** Mention handling X hours of video, Y concurrent streams
- **Highlight Performance:** CDN usage reduced latency by X%, costs by Y%
- **Demonstrate ML:** Explain recommendation algorithm with accuracy metrics
- **Compare to Industry:** "Built similar architecture to YouTube/Netflix using..."
- **Show Monitoring:** Include screenshots of analytics dashboards with real metrics
- **Security:** Explain DRM implementation, signed URLs, access control

---

This project is part of the [Coding Projects Library](https://github.com/Ryanditko/biblioteca-de-projetos) and is licensed under the MIT License.
- Subtitles and multiple audio tracks
- Playlists and series
- Comments with timestamps
- Watch parties (watch together)
- Chromecast/AirPlay support
- Offline download
- Parental controls
- Advanced analytics
- Ad monetization

---
