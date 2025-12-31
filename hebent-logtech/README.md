# Hebent Logtech - GPS Tracking Backend Integration

## What it is

Backend service integration for Hebent Logistics providing real-time GPS tracking capabilities for their truck fleet. This project demonstrates the GPS-Tracker service (detailed in the [gps-tracker](../gps-tracker/) directory) deployed in a production logistics environment.

## My role

I designed, developed, and deployed the complete GPS tracking backend infrastructure for Hebent Logistics. Responsible for TCP server implementation, binary protocol parsing (Teltonika devices), database design, WebSocket real-time streaming, and production deployment with monitoring.

## Tech stack

**Service Architecture:**
- GPS-Tracker service (see [../gps-tracker/README.md](../gps-tracker/README.md) for full technical details)
- Node.js 18+ with TypeScript 5.5+
- Express 5.2.1 for REST API
- WebSocket server (ws 8.16.0) for real-time updates

**Database:**
- PostgreSQL 12+ with Prisma ORM 7.0.1
- Connection pooling for high-throughput GPS data

**GPS Devices:**
- Teltonika FCM650, FMB125 (binary Codec 8/8E/16 protocol)
- Tbox devices (JSON-over-TCP protocol)

**Infrastructure:**
- Docker containerization
- PM2 process management
- Nginx reverse proxy
- Firewall rules for device connections

## System architecture

The GPS tracking system operates as a standalone backend service integrated with Hebent's logistics platform:

```
Teltonika/Tbox GPS Devices
    ↓ TCP Connection (Port 8080)
GPS-Tracker Service
    ├→ Binary Protocol Parser
    ├→ IMEI Authentication
    ├→ PostgreSQL Persistence
    └→ WebSocket Broadcasting
         ↓
Hebent Web Dashboard (real-time location updates)
Hebent Mobile App
Third-party Integrations
```

### Integration Points

**Data Flow:**
1. GPS devices connect via TCP and authenticate with IMEI
2. Location data parsed and validated
3. Persisted to PostgreSQL with device state tracking
4. Broadcast to connected WebSocket clients
5. REST API provides historical location queries

**Key Features:**
- Real-time location tracking for entire truck fleet
- Device state management (ignition on/off, movement detection)
- Historical route playback
- Geofencing capabilities
- Trip distance calculations

## Key technical decisions

**Standalone service architecture:**
Deployed as independent microservice rather than embedded in main logistics platform. Enables scaling GPS infrastructure independently and provides isolation for high-throughput TCP connections.

**Multi-protocol support:**
Handled both Teltonika binary devices and Tbox JSON devices on same port. Allowed Hebent to use existing device hardware while transitioning to newer models without infrastructure changes.

**PostgreSQL with proper indexing:**
Designed schema with indexes on `device_id`, `timestamp`, and `imei` for efficient historical queries. Critical for dashboard features like "show truck routes for last 7 days".

**WebSocket for real-time updates:**
Enabled live tracking dashboard where dispatchers see truck movements without page refresh. Lower latency than polling REST API every N seconds.

## Notable challenges solved

**Production-grade reliability:**
Challenge: GPS tracking is mission-critical - any downtime means dispatchers can't locate trucks. Solution: Implemented connection pooling, automatic reconnection logic for devices, database connection health checks, and PM2 auto-restart on crashes. Achieved 99.9% uptime.

**Device connection management:**
Challenge: Trucks in remote areas have intermittent connectivity, causing frequent reconnections. Solution: Stateless connection handling with device registry. Each connection independently authenticates via IMEI and processes data without session state.

**High-throughput data ingestion:**
Challenge: 100+ trucks sending location updates every 30 seconds = 200 inserts/minute. Solution: Batch inserts using Prisma `createMany`, database connection pooling (20 connections), and async processing to prevent blocking TCP server.

**Accurate device state tracking:**
Challenge: Determining if truck is moving, stopped, or has ignition on/off based on IO elements. Solution: Implemented state machine tracking last ignition time, movement detection via speed threshold, and stopped time calculation for idle alerts.

## Code highlights

The complete technical implementation is documented in [gps-tracker/README.md](../gps-tracker/README.md), including:

- Binary protocol parsing for Teltonika Codec 8/8E/16
- CRC16-IBM validation implementation
- PostgreSQL schema with DECIMAL coordinates
- WebSocket broadcasting with connection lifecycle management
- REST API for historical location queries

### Hebent-Specific Customizations

**Device Registration:**
- Custom IMEI whitelist validation for Hebent's truck fleet
- Integration with Hebent's truck database for automatic device-truck association

**Dashboard Integration:**
- WebSocket endpoint consumed by Hebent's React dashboard
- REST API endpoints for trip history and route playback

**Alerting:**
- Webhook notifications when trucks enter/exit geofenced areas
- Ignition state change alerts for unauthorized vehicle usage

## Deployment & environment

**Production Setup:**
- Deployed on Hebent's VPS infrastructure
- Docker container with Node.js 18 Alpine
- PostgreSQL 12 on managed database instance
- Nginx reverse proxy with SSL termination

**Networking:**
- TCP port 8080 exposed for GPS device connections (IP whitelist applied)
- HTTP port 3000 for REST API (internal network only)
- WebSocket port 8081 for dashboard clients (SSL/TLS)

**Monitoring:**
- Custom logging for protocol parsing errors
- Database query performance monitoring with slow query log
- Device connection count metrics
- Alert on device disconnection > 10 minutes

**Scalability:**
Current deployment handles 100+ concurrent device connections. Stateless design allows horizontal scaling if fleet grows beyond 500 trucks.

## Public links

Private commercial project for Hebent Logistics. Full technical specification available in [gps-tracker/README.md](../gps-tracker/README.md).
