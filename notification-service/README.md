# Notification Service

## Overview
Rust gRPC notification microservice for THXCONNECT that stores devices/notifications in Postgres and delivers push messages via Firebase Cloud Messaging. Runs DB migrations on boot, validates FCM tokens, and exposes RPCs to register/deregister devices, fetch notifications, and send user or broadcast pushes.

## Tech Stack
- Rust 2021, Tokio, Tonic/Prost (gRPC)
- Diesel + Postgres + r2d2
- Firebase Cloud Messaging (fcm_v1) with yup-oauth2 service accounts
- tracing + tracing-subscriber for logging
- Dockerfile + Nix flake/devshell for tooling

## Architecture
- Binary wraps CLI crate; CLI boots the service after loading `.env` via `dotenv`.
- Service layer (`service/`) wires gRPC server, FCM clients (validator + sender), and Diesel pool/migrations.
- Repository layer manages device tokens and notifications; model layer maps Diesel schemas to protobuf types.

## Features
- Register, validate, and delete device tokens per user
- Send notification to a specific user with typed metadata; auto-prune invalid tokens
- Broadcast (topic “all”) mass notifications
- List stored notifications with pagination

## API Endpoints
*[Add API documentation]*

## Deployment
*[Add deployment instructions]*
