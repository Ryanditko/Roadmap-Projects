# Online Course Platform (LMS)

<p align="center">
  <img src="https://images.unsplash.com/photo-1501504905252-473c47e087f8?w=600&q=80" alt="Online Learning" width="600">
</p>

## Overview

**Difficulty:** Advanced  
**Estimated Time:** 40–50 hours  
**Prerequisites:** Node.js/Python, React, Video Streaming, Payment APIs, PDF Generation, WebSockets, MongoDB/PostgreSQL  

An **Online Course Platform (LMS - Learning Management System)** is a comprehensive e-learning solution that allows instructors to create, manage, and sell courses while students can enroll, learn, and track their progress. The platform features:

- Course creation with modules, lessons, and resources
- Video hosting and streaming with adaptive quality
- Progress tracking and completion certificates
- Interactive quizzes and assessments with automatic grading
- Payment integration (one-time purchase or subscriptions)
- Student dashboard with enrolled courses and progress
- Instructor dashboard with analytics and earnings
- Discussion forums and Q&A
- Gamification with badges, points, and leaderboards
- Live classes and webinars

This project covers video streaming, content delivery, payment processing, certificate generation, quiz engines, gamification systems, and building a scalable LMS similar to Udemy, Coursera, or Teachable.

---

## Features

### Must Have

- User roles (student, instructor, admin)
- Course creation with hierarchy:
  - Course → Modules → Lessons → Resources
- Video upload and streaming
- Video player with basic controls (play, pause, seek, volume)
- Progress tracking (lessons completed, course completion %)
- Enrollment system
- Basic quiz functionality (multiple choice)
- Certificate generation upon course completion
- Payment integration (Stripe or PayPal)
- Student dashboard (enrolled courses, progress)
- Instructor dashboard (courses, students, earnings)
- Search and filtering courses
- Course ratings and reviews
- Responsive design

### Nice to Have

- Advanced video player features:
  - Playback speed control (0.5x, 1x, 1.5x, 2x)
  - Quality selection (360p, 480p, 720p, 1080p)
  - Subtitles/captions (multiple languages)
  - Picture-in-picture mode
  - Keyboard shortcuts
- Advanced quiz types:
  - Multiple choice, true/false, short answer, essay
  - Timed quizzes
  - Question randomization
  - Partial credit scoring
- Assignments with file submissions
- Course preview (free sample lessons)
- Drip content (scheduled lesson releases)
- Course bundles and packages
- Subscription model (monthly access to all courses)
- Coupon and discount codes
- Affiliate program
- Live classes with Zoom/Jitsi integration
- Discussion forums with threads and replies
- Q&A section for lessons
- Notes and bookmarks
- Certificate customization (templates, branding)
- Gamification:
  - Badges for achievements
  - Points for activities
  - Leaderboards
  - Streaks and milestones
- Email notifications (enrollment, completion, announcements)
- Course announcements
- Instructor messaging
- Mobile app (React Native)
- Offline mode (download lessons)
- Analytics dashboard (completion rates, engagement, revenue)
- Course recommendations (based on interests)
- Multi-language support (i18n)
- Accessibility (WCAG compliance, screen readers)

---

## Learning Objectives

By building this project, you will learn:

- **Video hosting and streaming** (AWS S3, Cloudflare Stream, Vimeo API)
- **HLS adaptive streaming** for multiple quality levels
- **Payment processing** with Stripe/PayPal (checkout, webhooks)
- **PDF generation** with PDFKit or Puppeteer
- **Quiz engines** with automatic grading
- **Progress tracking** algorithms (completion percentage)
- **Certificate generation** with dynamic data
- **File uploads** and storage (AWS S3, Cloudinary)
- **Video transcoding** with FFmpeg
- **Content security** (signed URLs, DRM)
- **Subscription billing** (recurring payments)
- **Webhook handling** for payment events
- **Email automation** (SendGrid, AWS SES)
- **Real-time features** with WebSockets (live classes, notifications)
- **Analytics** and reporting

---

## Project Concepts

This project simulates a complete LMS and covers:

- **Course Hierarchy:** Courses → Modules → Lessons → Resources
- **Video Delivery:** HLS streaming with adaptive bitrate
- **Enrollment:** Purchase, enroll, access control
- **Progress Tracking:** Lesson completion, quiz scores, overall progress
- **Certification:** Auto-generate certificates upon 100% completion
- **Payments:** One-time purchase, subscriptions, refunds
- **Gamification:** Reward system to increase engagement
- **Analytics:** Track student engagement, completion rates, revenue

---

## Step-by-Step Implementation

### Step 1: Course Structure and Data Models

Define the course hierarchy:

```javascript
// models/Course.js
const mongoose = require('mongoose');

const resourceSchema = new mongoose.Schema({
    title: String,
    type: {
        type: String,
        enum: ['video', 'pdf', 'quiz', 'assignment', 'text', 'link']
    },
    videoUrl: String,
    duration: Number, // in seconds
    pdfUrl: String,
    content: String, // for text content
    externalLink: String,
    isPreview: { type: Boolean, default: false }, // Free preview
    order: Number
});

const lessonSchema = new mongoose.Schema({
    title: { type: String, required: true },
    description: String,
    resources: [resourceSchema],
    duration: Number, // Total duration of all resources
    order: Number
});

const moduleSchema = new mongoose.Schema({
    title: { type: String, required: true },
    description: String,
    lessons: [lessonSchema],
    order: Number
});

const courseSchema = new mongoose.Schema({
    title: { type: String, required: true },
    slug: { type: String, required: true, unique: true },
    description: String,
    shortDescription: String,
    thumbnail: String,
    previewVideo: String,
    
    instructor: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true
    },
    
    category: String,
    level: {
        type: String,
        enum: ['beginner', 'intermediate', 'advanced']
    },
    
    modules: [moduleSchema],
    
    price: { type: Number, required: true, default: 0 },
    currency: { type: String, default: 'USD' },
    discountPrice: Number,
    
    language: { type: String, default: 'en' },
    subtitles: [String], // Available subtitle languages
    
    requirements: [String],
    whatYouWillLearn: [String],
    targetAudience: [String],
    
    isPublished: { type: Boolean, default: false },
    publishedAt: Date,
    
    enrollmentCount: { type: Number, default: 0 },
    averageRating: { type: Number, default: 0 },
    ratingCount: { type: Number, default: 0 },
    
    certificateTemplate: String,
    
    createdAt: { type: Date, default: Date.now },
    updatedAt: { type: Date, default: Date.now }
});

// Calculate total duration
courseSchema.virtual('totalDuration').get(function() {
    return this.modules.reduce((total, module) => {
        return total + module.lessons.reduce((lessonTotal, lesson) => {
            return lessonTotal + (lesson.duration || 0);
        }, 0);
    }, 0);
});

// Calculate total lessons
courseSchema.virtual('totalLessons').get(function() {
    return this.modules.reduce((total, module) => {
        return total + module.lessons.length;
    }, 0);
});

module.exports = mongoose.model('Course', courseSchema);
```

```javascript
// models/Enrollment.js
const mongoose = require('mongoose');

const progressSchema = new mongoose.Schema({
    moduleId: mongoose.Schema.Types.ObjectId,
    lessonId: mongoose.Schema.Types.ObjectId,
    resourceId: mongoose.Schema.Types.ObjectId,
    completed: { type: Boolean, default: false },
    completedAt: Date,
    lastWatched: Number, // Video progress in seconds
    quizScore: Number,
    quizAttempts: Number
});

const enrollmentSchema = new mongoose.Schema({
    user: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true
    },
    course: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Course',
        required: true
    },
    
    enrolledAt: { type: Date, default: Date.now },
    completedAt: Date,
    
    progress: [progressSchema],
    completionPercentage: { type: Number, default: 0 },
    
    certificateIssued: { type: Boolean, default: false },
    certificateUrl: String,
    
    lastAccessedAt: Date,
    totalWatchTime: { type: Number, default: 0 }, // in seconds
    
    paymentId: String, // Stripe payment ID
    amountPaid: Number,
    currency: String
});

// Calculate completion percentage
enrollmentSchema.methods.calculateProgress = function() {
    const totalResources = this.progress.length;
    const completedResources = this.progress.filter(p => p.completed).length;
    
    this.completionPercentage = totalResources > 0 
        ? Math.round((completedResources / totalResources) * 100)
        : 0;
    
    // Check if course is completed
    if (this.completionPercentage === 100 && !this.completedAt) {
        this.completedAt = new Date();
    }
    
    return this.completionPercentage;
};

module.exports = mongoose.model('Enrollment', enrollmentSchema);
```

### Step 2: Video Streaming Service

Implement video upload and streaming:

```javascript
// services/videoService.js
const AWS = require('aws-sdk');
const ffmpeg = require('fluent-ffmpeg');
const fs = require('fs');
const path = require('path');

const s3 = new AWS.S3({
    accessKeyId: process.env.AWS_ACCESS_KEY,
    secretAccessKey: process.env.AWS_SECRET_KEY
});

class VideoService {
    async uploadVideo(file, courseId, lessonId) {
        try {
            // Generate unique filename
            const timestamp = Date.now();
            const filename = `${courseId}/${lessonId}/${timestamp}_${file.originalname}`;
            
            // Upload original video to S3
            const uploadParams = {
                Bucket: process.env.AWS_S3_BUCKET,
                Key: `videos/original/${filename}`,
                Body: file.buffer,
                ContentType: file.mimetype,
                ACL: 'private'
            };
            
            await s3.upload(uploadParams).promise();
            
            // Transcode video to multiple qualities
            const transcodePromises = [360, 480, 720, 1080].map(quality =>
                this.transcodeVideo(file.buffer, filename, quality)
            );
            
            await Promise.all(transcodePromises);
            
            // Generate HLS playlist
            const playlistUrl = await this.generateHLSPlaylist(filename);
            
            return {
                originalUrl: `videos/original/${filename}`,
                playlistUrl: playlistUrl,
                qualities: [360, 480, 720, 1080]
            };
            
        } catch (error) {
            console.error('Error uploading video:', error);
            throw error;
        }
    }
    
    async transcodeVideo(videoBuffer, filename, quality) {
        const tempInput = `/tmp/${Date.now()}_input.mp4`;
        const tempOutput = `/tmp/${Date.now()}_output_${quality}p.mp4`;
        
        // Write buffer to temp file
        fs.writeFileSync(tempInput, videoBuffer);
        
        return new Promise((resolve, reject) => {
            ffmpeg(tempInput)
                .outputOptions([
                    `-vf scale=-2:${quality}`,
                    '-c:v libx264',
                    '-crf 23',
                    '-preset medium',
                    '-c:a aac',
                    '-b:a 128k'
                ])
                .output(tempOutput)
                .on('end', async () => {
                    // Upload transcoded video to S3
                    const transcodedBuffer = fs.readFileSync(tempOutput);
                    
                    const uploadParams = {
                        Bucket: process.env.AWS_S3_BUCKET,
                        Key: `videos/${quality}p/${filename}`,
                        Body: transcodedBuffer,
                        ContentType: 'video/mp4',
                        ACL: 'private'
                    };
                    
                    await s3.upload(uploadParams).promise();
                    
                    // Clean up temp files
                    fs.unlinkSync(tempInput);
                    fs.unlinkSync(tempOutput);
                    
                    resolve();
                })
                .on('error', (err) => {
                    console.error(`Error transcoding video to ${quality}p:`, err);
                    reject(err);
                })
                .run();
        });
    }
    
    async generateHLSPlaylist(filename) {
        // Generate HLS master playlist
        const qualities = [360, 480, 720, 1080];
        const bandwidths = {
            360: 800000,
            480: 1400000,
            720: 2800000,
            1080: 5000000
        };
        
        let masterPlaylist = '#EXTM3U\n#EXT-X-VERSION:3\n';
        
        qualities.forEach(quality => {
            masterPlaylist += `#EXT-X-STREAM-INF:BANDWIDTH=${bandwidths[quality]},RESOLUTION=${quality === 360 ? '640x360' : quality === 480 ? '854x480' : quality === 720 ? '1280x720' : '1920x1080'}\n`;
            masterPlaylist += `${quality}p/playlist.m3u8\n`;
        });
        
        // Upload master playlist to S3
        const playlistKey = `videos/playlists/${filename.replace('.mp4', '.m3u8')}`;
        
        const uploadParams = {
            Bucket: process.env.AWS_S3_BUCKET,
            Key: playlistKey,
            Body: masterPlaylist,
            ContentType: 'application/vnd.apple.mpegurl',
            ACL: 'private'
        };
        
        await s3.upload(uploadParams).promise();
        
        return playlistKey;
    }
    
    async getSignedUrl(key, expiresIn = 3600) {
        // Generate signed URL for private video access
        const params = {
            Bucket: process.env.AWS_S3_BUCKET,
            Key: key,
            Expires: expiresIn
        };
        
        return s3.getSignedUrl('getObject', params);
    }
}

module.exports = new VideoService();
```

### Step 3: Quiz Engine

Build a quiz system with automatic grading:

```javascript
// models/Quiz.js
const mongoose = require('mongoose');

const questionSchema = new mongoose.Schema({
    type: {
        type: String,
        enum: ['multiple_choice', 'true_false', 'short_answer', 'essay'],
        required: true
    },
    question: { type: String, required: true },
    options: [String], // For multiple choice
    correctAnswer: mongoose.Schema.Types.Mixed, // String or array of strings
    points: { type: Number, default: 1 },
    explanation: String,
    order: Number
});

const quizSchema = new mongoose.Schema({
    title: { type: String, required: true },
    description: String,
    
    course: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Course',
        required: true
    },
    lesson: {
        type: mongoose.Schema.Types.ObjectId,
        required: true
    },
    
    questions: [questionSchema],
    
    passingScore: { type: Number, default: 70 }, // Percentage
    timeLimit: Number, // in minutes
    attemptsAllowed: { type: Number, default: -1 }, // -1 = unlimited
    randomizeQuestions: { type: Boolean, default: false },
    showCorrectAnswers: { type: Boolean, default: true },
    
    createdAt: { type: Date, default: Date.now }
});

module.exports = mongoose.model('Quiz', quizSchema);
```

```javascript
// models/QuizAttempt.js
const mongoose = require('mongoose');

const answerSchema = new mongoose.Schema({
    questionId: mongoose.Schema.Types.ObjectId,
    answer: mongoose.Schema.Types.Mixed,
    isCorrect: Boolean,
    pointsEarned: Number
});

const quizAttemptSchema = new mongoose.Schema({
    user: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true
    },
    quiz: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Quiz',
        required: true
    },
    
    answers: [answerSchema],
    
    score: Number, // Percentage
    pointsEarned: Number,
    totalPoints: Number,
    
    passed: Boolean,
    
    startedAt: { type: Date, default: Date.now },
    completedAt: Date,
    timeSpent: Number // in seconds
});

// Calculate score
quizAttemptSchema.methods.calculateScore = function(quiz) {
    let pointsEarned = 0;
    let totalPoints = 0;
    
    this.answers.forEach((answer, index) => {
        const question = quiz.questions.id(answer.questionId);
        totalPoints += question.points;
        
        if (answer.isCorrect) {
            pointsEarned += question.points;
        }
    });
    
    this.pointsEarned = pointsEarned;
    this.totalPoints = totalPoints;
    this.score = totalPoints > 0 ? Math.round((pointsEarned / totalPoints) * 100) : 0;
    this.passed = this.score >= quiz.passingScore;
    
    return this.score;
};

module.exports = mongoose.model('QuizAttempt', quizAttemptSchema);
```

```javascript
// services/quizService.js
class QuizService {
    gradeQuizAttempt(quiz, userAnswers) {
        const gradedAnswers = userAnswers.map(userAnswer => {
            const question = quiz.questions.id(userAnswer.questionId);
            
            let isCorrect = false;
            
            switch (question.type) {
                case 'multiple_choice':
                case 'true_false':
                    isCorrect = userAnswer.answer === question.correctAnswer;
                    break;
                    
                case 'short_answer':
                    // Case-insensitive comparison
                    isCorrect = userAnswer.answer.trim().toLowerCase() === 
                               question.correctAnswer.trim().toLowerCase();
                    break;
                    
                case 'essay':
                    // Manual grading required
                    isCorrect = null;
                    break;
            }
            
            return {
                questionId: userAnswer.questionId,
                answer: userAnswer.answer,
                isCorrect: isCorrect,
                pointsEarned: isCorrect ? question.points : 0
            };
        });
        
        return gradedAnswers;
    }
}

module.exports = new QuizService();
```

### Step 4: Certificate Generation

Generate PDF certificates:

```javascript
// services/certificateService.js
const PDFDocument = require('pdfkit');
const fs = require('fs');
const AWS = require('aws-sdk');

const s3 = new AWS.S3({
    accessKeyId: process.env.AWS_ACCESS_KEY,
    secretAccessKey: process.env.AWS_SECRET_KEY
});

class CertificateService {
    async generateCertificate(enrollment, course, user) {
        try {
            const doc = new PDFDocument({
                layout: 'landscape',
                size: 'A4'
            });
            
            const filename = `certificate_${enrollment._id}_${Date.now()}.pdf`;
            const tempPath = `/tmp/${filename}`;
            
            // Create write stream
            const stream = fs.createWriteStream(tempPath);
            doc.pipe(stream);
            
            // Certificate design
            // Border
            doc.rect(20, 20, doc.page.width - 40, doc.page.height - 40)
               .lineWidth(3)
               .stroke('#1a73e8');
            
            // Title
            doc.fontSize(40)
               .font('Helvetica-Bold')
               .fillColor('#1a73e8')
               .text('Certificate of Completion', 0, 100, {
                   align: 'center'
               });
            
            // Divider
            doc.moveTo(150, 160)
               .lineTo(doc.page.width - 150, 160)
               .strokeColor('#1a73e8')
               .lineWidth(2)
               .stroke();
            
            // Body text
            doc.fontSize(18)
               .font('Helvetica')
               .fillColor('#333')
               .text('This is to certify that', 0, 200, {
                   align: 'center'
               });
            
            doc.fontSize(32)
               .font('Helvetica-Bold')
               .fillColor('#1a73e8')
               .text(user.name, 0, 240, {
                   align: 'center'
               });
            
            doc.fontSize(18)
               .font('Helvetica')
               .fillColor('#333')
               .text('has successfully completed the course', 0, 300, {
                   align: 'center'
               });
            
            doc.fontSize(28)
               .font('Helvetica-Bold')
               .fillColor('#1a73e8')
               .text(course.title, 0, 340, {
                   align: 'center'
               });
            
            // Date and instructor
            const completionDate = enrollment.completedAt.toLocaleDateString('en-US', {
                year: 'numeric',
                month: 'long',
                day: 'numeric'
            });
            
            doc.fontSize(14)
               .font('Helvetica')
               .fillColor('#666')
               .text(`Completion Date: ${completionDate}`, 0, 420, {
                   align: 'center'
               });
            
            // Certificate ID (for verification)
            doc.fontSize(10)
               .fillColor('#999')
               .text(`Certificate ID: ${enrollment._id}`, 0, doc.page.height - 80, {
                   align: 'center'
               });
            
            doc.end();
            
            // Wait for PDF to finish
            await new Promise((resolve, reject) => {
                stream.on('finish', resolve);
                stream.on('error', reject);
            });
            
            // Upload to S3
            const fileBuffer = fs.readFileSync(tempPath);
            
            const uploadParams = {
                Bucket: process.env.AWS_S3_BUCKET,
                Key: `certificates/${filename}`,
                Body: fileBuffer,
                ContentType: 'application/pdf',
                ACL: 'public-read'
            };
            
            const result = await s3.upload(uploadParams).promise();
            
            // Clean up temp file
            fs.unlinkSync(tempPath);
            
            return result.Location;
            
        } catch (error) {
            console.error('Error generating certificate:', error);
            throw error;
        }
    }
}

module.exports = new CertificateService();
```

### Step 5: Payment Integration

Implement Stripe payment:

```javascript
// services/paymentService.js
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

class PaymentService {
    async createCheckoutSession(course, user) {
        try {
            const session = await stripe.checkout.sessions.create({
                payment_method_types: ['card'],
                line_items: [
                    {
                        price_data: {
                            currency: course.currency.toLowerCase(),
                            product_data: {
                                name: course.title,
                                description: course.shortDescription,
                                images: [course.thumbnail]
                            },
                            unit_amount: Math.round((course.discountPrice || course.price) * 100)
                        },
                        quantity: 1
                    }
                ],
                mode: 'payment',
                success_url: `${process.env.FRONTEND_URL}/courses/${course.slug}/success?session_id={CHECKOUT_SESSION_ID}`,
                cancel_url: `${process.env.FRONTEND_URL}/courses/${course.slug}`,
                customer_email: user.email,
                metadata: {
                    courseId: course._id.toString(),
                    userId: user._id.toString()
                }
            });
            
            return session;
            
        } catch (error) {
            console.error('Error creating checkout session:', error);
            throw error;
        }
    }
    
    async handleWebhook(event) {
        try {
            switch (event.type) {
                case 'checkout.session.completed':
                    await this.handleSuccessfulPayment(event.data.object);
                    break;
                    
                case 'charge.refunded':
                    await this.handleRefund(event.data.object);
                    break;
                    
                default:
                    console.log(`Unhandled event type: ${event.type}`);
            }
        } catch (error) {
            console.error('Error handling webhook:', error);
            throw error;
        }
    }
    
    async handleSuccessfulPayment(session) {
        const { courseId, userId } = session.metadata;
        
        // Create enrollment
        const enrollment = await Enrollment.create({
            user: userId,
            course: courseId,
            paymentId: session.payment_intent,
            amountPaid: session.amount_total / 100,
            currency: session.currency
        });
        
        // Initialize progress tracking
        const course = await Course.findById(courseId);
        
        course.modules.forEach(module => {
            module.lessons.forEach(lesson => {
                lesson.resources.forEach(resource => {
                    enrollment.progress.push({
                        moduleId: module._id,
                        lessonId: lesson._id,
                        resourceId: resource._id,
                        completed: false
                    });
                });
            });
        });
        
        await enrollment.save();
        
        // Update course enrollment count
        course.enrollmentCount += 1;
        await course.save();
        
        // Send confirmation email
        // await emailService.sendEnrollmentConfirmation(userId, courseId);
    }
    
    async handleRefund(charge) {
        const paymentId = charge.payment_intent;
        
        // Find enrollment and mark as refunded
        const enrollment = await Enrollment.findOne({ paymentId });
        
        if (enrollment) {
            // Remove enrollment or mark as inactive
            await enrollment.remove();
            
            // Update course enrollment count
            await Course.findByIdAndUpdate(
                enrollment.course,
                { $inc: { enrollmentCount: -1 } }
            );
        }
    }
}

module.exports = new PaymentService();
```

## Technology Stack

### Frontend
- **React** - UI framework
- **TypeScript** - Type safety
- **Redux Toolkit** - State management
- **React Query** - Server state
- **React Player** or **Video.js** - Video player
- **HLS.js** - HLS streaming
- **Material-UI** or **Chakra UI** - Component library
- **React Hook Form** - Form handling
- **Stripe Elements** - Payment UI

### Backend
- **Node.js + Express** - API server
- **MongoDB** with Mongoose - Database
- **Redis** - Caching
- **Bull** - Job queue (video processing)
- **JWT** - Authentication
- **Stripe** - Payment processing
- **SendGrid** or **AWS SES** - Email

### Infrastructure
- **AWS S3** - Video and file storage
- **AWS CloudFront** - CDN for video delivery
- **FFmpeg** - Video transcoding
- **Docker** - Containerization
- **Nginx** - Reverse proxy

## Testing Checklist

- [ ] Course creation with modules and lessons
- [ ] Video upload and streaming works
- [ ] Progress tracking updates correctly
- [ ] Quiz grading is accurate
- [ ] Certificate generation works
- [ ] Payment flow completes successfully
- [ ] Webhooks handle payment events
- [ ] Enrollment access control works
- [ ] Search and filtering courses
- [ ] Ratings and reviews display
- [ ] Responsive design on mobile

## Additional Challenges

### Easy Extensions
- Add course wishlist
- Implement course preview
- Create course announcements
- Add Q&A section for lessons

### Medium Extensions
- Live classes with Zoom/Jitsi
- Discussion forums
- Assignments with file submissions
- Drip content (scheduled releases)
- Course bundles
- Affiliate program

### Hard Extensions
- Subscription model (monthly access)
- Advanced analytics dashboard
- AI-powered course recommendations
- Automated subtitle generation
- Live streaming with WebRTC
- Mobile app (React Native)
- Offline mode (download courses)
- Proctored exams

## Common Pitfalls

1. **Video Storage Costs:** Videos are expensive to store and stream. Use CDN and compression.

2. **Payment Security:** Never store credit card data. Use Stripe Elements and webhooks properly.

3. **Progress Tracking:** Handle edge cases (page refresh, network errors). Save progress frequently.

4. **Video DRM:** Protect paid content with signed URLs and expiration times.

5. **Scalability:** Video transcoding is CPU-intensive. Use background jobs and cloud services.

## Resources

### Documentation
- [Stripe API](https://stripe.com/docs/api)
- [HLS.js](https://github.com/video-dev/hls.js/)
- [PDFKit](https://pdfkit.org/)
- [FFmpeg](https://ffmpeg.org/documentation.html)

### Tutorials
- [Building LMS Platforms](https://www.youtube.com/results?search_query=lms+platform+tutorial)
- [Video Streaming with HLS](https://www.youtube.com/results?search_query=hls+streaming+tutorial)

### Design Inspiration
- [Udemy](https://www.udemy.com/)
- [Coursera](https://www.coursera.org/)
- [Teachable](https://teachable.com/)

## Submission Checklist

- [ ] Course creation and management
- [ ] Video upload and streaming
- [ ] Progress tracking
- [ ] Quiz engine with grading
- [ ] Certificate generation
- [ ] Payment integration
- [ ] Student dashboard
- [ ] Instructor dashboard
- [ ] Search and filtering
- [ ] Ratings and reviews
- [ ] Responsive design
- [ ] Documentation

## Portfolio Tips

- **Show Complete Flow:** Demo from course creation to enrollment to certificate
- **Highlight Scale:** "Supports 10,000+ concurrent students, 500+ hours of video content"
- **Explain Architecture:** System diagram showing video pipeline, payment flow, progress tracking
- **Compare to Industry:** "Built similar to Udemy/Coursera LMS architecture"
- **Show Metrics:** Completion rates, engagement analytics, revenue tracking
- **Technical Depth:** Discuss video transcoding, HLS streaming, payment webhooks, certificate generation

---

This project is part of the [Coding Projects Library](https://github.com/Ryanditko/biblioteca-de-projetos) and is licensed under the MIT License.
