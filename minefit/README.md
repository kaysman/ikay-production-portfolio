# Minefit

[Website](https://minefit.jp/) | [App Store](https://apps.apple.com/us/app/minefit-%E8%87%AA%E5%AE%85%E3%81%A7%E3%83%95%E3%82%A3%E3%83%83%E3%83%88%E3%83%8D%E3%82%B9-%E8%87%AA%E5%AE%85%E3%83%88%E3%83%AC%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0/id1521495780) | [Play Store](https://play.google.com/store/apps/details?id=jp.joyfit.minefit&hl=pt)

![Cover](images/cover.png)

## Overview

We offer a wide variety of live and recorded lessons from professional trainers and instructors with years of experience and knowledge from wellness and fitness industry. Making everyone's training environment anytime, anywhere. minefit will support your healthy and happy daily life.

## Screenshots

![App Screenshot](images/screen.webp)

## Tech Stack

- **Frontend:** Flutter
- **Backend:** Rust
- **Database:** PostgreSQL
- **Video Streaming:** AWS S3 + CloudFront / Firebase Storage
- **Real-time:** WebRTC for live sessions, Socket.io
- **Payment:** Stripe / PayPal integration
- **Analytics:** Firebase Analytics, Crashlytics
- **Push Notifications:** Firebase Cloud Messaging (FCM)

## Platform

- iOS & Android (Cross-platform mobile app)
- Web platform (React/Next.js)

## Features

- **No Registration Required:** Quick start with guest access for immediate workout sessions
- **Live & On-Demand Classes:** Daily live fitness sessions with professional trainers plus 400+ pre-recorded workout videos
- **Gym Integration:** Use at participating JOYFIT gyms with QR code check-in and ticket purchasing
- **Interactive Live Lessons:** Real-time instructor feedback during unlimited live streaming classes
- **Diverse Content Library:** Access to dance, yoga, strength training, and specialized fitness programs
- **Video Recommendations:** Personalized workout suggestions with continuously updated content
- **Premium Lessons:** One-on-one interactive sessions with professional trainers for personalized guidance
- **Class Scheduling:** Browse and reserve upcoming live sessions with countdown timers
- **Offline Access:** Download videos for offline viewing during travel or low connectivity

## Architecture

- **Microservices Architecture:** Separate services for user management, content delivery, live streaming, and payment processing
- **CDN Distribution:** Global content delivery network for low-latency video streaming
- **Scalable Infrastructure:** AWS/GCP cloud hosting with auto-scaling for live session traffic spikes
- **Video Processing Pipeline:** Automated transcoding and optimization for multiple device resolutions
- **Real-time Communication:** WebSocket connections for live class interactivity and instructor feedback
- **Secure Payment Gateway:** PCI-compliant payment processing with subscription management
