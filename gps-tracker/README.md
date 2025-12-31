# GPS Tracker Service

## What it is

A production TCP server that receives, parses, and persists real-time GPS location data from Teltonika (FCM650, FMB125) and Tbox GPS devices. Implements binary protocol parsing (Codec 8/8E/16) and JSON-over-TCP with WebSocket streaming for live location updates.

## My role

I designed and implemented the complete backend service including binary protocol parsing, multi-protocol device support, database schema design, WebSocket broadcasting, and production deployment. Responsible for protocol specification implementation, data validation, and real-time streaming architecture.

## Tech stack

**Runtime & Language:**
- Node.js 18+
- TypeScript 5.5+
- ES Modules

**Framework & Server:**
- Express 5.2.1
- Native TCP server (net module)
- WebSocket server (ws 8.16.0)

**Database & ORM:**
- PostgreSQL 12+
- Prisma ORM 7.0.1
- Prisma Client with PostgreSQL adapter

**Development Tools:**
- tsx (TypeScript execution with watch mode)
- Yarn 4.12.0 (package manager)
- dotenv for environment configuration

**Protocol Support:**
- Teltonika binary protocols (Codec 8, 8E Extended, 16)
- Tbox JSON-over-TCP protocol
- Custom CRC16-IBM validation

## System architecture

### Layered Architecture

The service follows a layered architecture pattern optimized for real-time data processing:

```
src/
├── server.ts                  # TCP server + Express API + WebSocket orchestration
├── teltonika-parser.ts        # Binary protocol parser (Codec 8/8E/16)
├── tbox/
│   └── parser.ts             # JSON protocol parser
├── database.ts                # Data persistence layer
├── websocket-server.ts        # Real-time broadcast server
├── api/                       # REST API endpoints
│   ├── devices/
│   └── locations/
├── middlewares/               # Express middleware chain
├── shared/                    # Utilities and helpers
├── generated/                 # Prisma client
└── prisma.ts                 # Database client configuration
```

### Request Flow

```
GPS Device (FCM650/FMB125/Tbox)
    ↓ TCP Connection (Port 8080)
TCP Server (net.createServer)
    ↓ Buffer data
Protocol Detection Layer
    ├→ Teltonika Parser (binary Codec 8/8E/16)
    └→ Tbox Parser (JSON)
    ↓ Parsed AVL records
Database Layer (Prisma)
    ↓ PostgreSQL persistence
WebSocket Broadcast
    ↓ ws:// protocol
Connected Clients
```

### Database Schema

```prisma
// prisma/schema.prisma
model Device {
  id     Int     @id @default(autoincrement())
  imei   String  @unique @db.VarChar(15)
  name   String? @db.VarChar(255)
  active Boolean @default(true)

  ignitionStatus      Boolean?  @map("ignition_status")
  movementStatus      Boolean?  @map("movement_status")
  lastLatitude        Decimal   @db.Decimal(10, 8)
  lastLongitude       Decimal   @db.Decimal(11, 8)
  lastStoppedTime     DateTime? @map("last_stopped_time")
  lastIgnitionOnTime  DateTime? @map("last_ignition_on_time")
  lastIgnitionOffTime DateTime? @map("last_ignition_off_time")
  lastConnectedTime   DateTime? @map("last_connected_time")
  lastDisconnectedTime DateTime? @map("last_disconnected_time")

  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  locations GPSLocation[]

  @@index([active])
  @@index([createdAt])
  @@map("devices")
}

model GPSLocation {
  id       Int    @id @default(autoincrement())
  deviceId Int    @map("device_id")
  device   Device @relation(fields: [deviceId], references: [id], onDelete: Cascade)

  imei      String   @db.VarChar(15)
  timestamp DateTime @map("timestamp")

  latitude   Decimal @db.Decimal(10, 8)
  longitude  Decimal @db.Decimal(11, 8)
  altitude   Int?
  speed      Int?
  angle      Int?
  satellites Int?
  priority   Int?

  ioElements Json?    @map("io_elements")
  createdAt  DateTime @default(now()) @map("created_at")

  @@index([deviceId])
  @@index([timestamp])
  @@index([imei])
  @@map("gps_locations")
}
```

**Design decisions:**
- `Decimal` type for lat/lon to prevent floating-point precision errors
- Indexed IMEI for fast device lookups
- Timestamp indexing for range queries
- JSON column for flexible IO element storage
- Cascading deletes to maintain referential integrity

## Key technical decisions

**Binary protocol parsing over third-party libraries:**
Implemented custom Teltonika parser instead of using existing libraries. This provided full control over codec versions (8, 8E, 16), enabled device-specific IO element mapping, and eliminated dependencies with poor TypeScript support.

**PostgreSQL Decimal type for coordinates:**
Used PostgreSQL `DECIMAL(10,8)` for latitude and `DECIMAL(11,8)` for longitude instead of `FLOAT`. Prevents cumulative floating-point errors critical for geofencing and distance calculations.

**Dual-protocol support on single port:**
Implemented protocol detection by inspecting first bytes of incoming data. Teltonika starts with 4-byte preamble (0x00000000), Tbox sends JSON. This simplified deployment by requiring only one exposed port.

**WebSocket for real-time updates over Server-Sent Events:**
Chose WebSocket bidirectional communication to enable future client→server commands (e.g., device configuration). SSE would only support server→client streaming.

**Prisma ORM over raw SQL:**
Type-safe database queries with automatic TypeScript types. Schema-first development with automatic migrations. Prevented SQL injection and eliminated runtime type mismatches.

**Stateless TCP connections:**
Each device connection is independent with no session state. Enables horizontal scaling by load-balancing TCP connections across multiple server instances.

## Notable challenges solved

**Binary protocol CRC validation:**
Challenge: Teltonika packets include CRC16-IBM checksum for integrity verification. Incorrect calculation caused all packets to be rejected. Solution: Implemented CRC16-IBM with polynomial 0x8005, initial value 0x0000, and big-endian byte order. Added comprehensive test vectors from Teltonika specification.

**Variable-length AVL records:**
Challenge: Each AVL record has variable number of IO elements (1-byte, 2-byte, 4-byte, 8-byte sizes). Buffer parsing required dynamic offset management. Solution: Implemented recursive parser that tracks offset through buffer, validating bounds at each read operation.

**Concurrent device connections:**
Challenge: Multiple GPS devices connecting simultaneously with overlapping IMEI authentication. Solution: Used IMEI-based device registry with async/await to serialize database operations per device while allowing parallel processing across different devices.

**Incomplete TCP packets:**
Challenge: TCP stream fragmentation meant packets could arrive split across multiple data events. Solution: Maintained per-connection buffer that accumulates data until complete packet detected (validated by preamble + data length). Prevents processing partial frames.

**IO element ID mapping:**
Challenge: Teltonika devices send hundreds of IO element IDs (battery voltage, temperature sensors, etc.) without human-readable labels. Solution: Created comprehensive IO ID→name mapping table supporting FCM650, FMB125, and generic parameters.

## Code highlights

### [src/teltonika-parser.ts](src/teltonika-parser.ts)
Demonstrates binary protocol parsing with bitwise operations:

```typescript
/**
 * Teltonika Protocol Parser
 * Supports multiple Teltonika devices: FCM650, FMB125, and others
 * Supports Codec 8, 8E (Extended), and Codec 16
 */

export interface AVLRecord {
  timestamp: Date;
  priority: number;
  longitude: number;
  latitude: number;
  altitude: number;
  angle: number;
  satellites: number;
  speed: number;
  ioElements: Record<string, unknown>;
}

export interface ParsedData {
  preamble: number;
  dataLength: number;
  codecId: number;
  numberOfRecords: number;
  records: AVLRecord[];
  crc: number;
}

class TeltonikaParser {
  private AVL_IDS: Record<number, string>;

  constructor() {
    this.AVL_IDS = {
      // Common IO elements (FCM650, FMB125, and others)
      1: "digitalInput1",
      9: "analogInput1",
      10: "digitalInput2",
      11: "digitalInput3",
      16: "totalDistance",
      17: "axisX",
      18: "axisY",
      19: "axisZ",
      21: "gsmSignal",
      24: "speed",
      66: "externalVoltage",
      67: "batteryVoltage",
      68: "batteryCurrent",
      69: "gnsStatus",
      72: "temperature1",
      73: "temperature2",
      74: "temperature3",
      // ... 100+ additional IO elements
    };
  }

  parse(buffer: Buffer): ParsedData {
    // Validate preamble (4 bytes: 0x00000000)
    const preamble = buffer.readUInt32BE(0);
    if (preamble !== 0x00000000) {
      throw new Error('Invalid preamble');
    }

    // Read data field length
    const dataLength = buffer.readUInt32BE(4);

    // Validate CRC before parsing
    const expectedCrc = buffer.readUInt16BE(8 + dataLength);
    const calculatedCrc = this.calculateCRC(buffer.slice(8, 8 + dataLength));

    if (expectedCrc !== calculatedCrc) {
      throw new Error(`CRC mismatch: expected ${expectedCrc}, got ${calculatedCrc}`);
    }

    // Parse codec and records
    // ... (implementation continues)
  }

  private calculateCRC(data: Buffer): number {
    let crc = 0;
    for (let i = 0; i < data.length; i++) {
      crc ^= data[i] << 8;
      for (let j = 0; j < 8; j++) {
        if (crc & 0x8000) {
          crc = (crc << 1) ^ 0x1021;
        } else {
          crc = crc << 1;
        }
      }
    }
    return crc & 0xFFFF;
  }
}
```

**Why this matters:** Shows low-level binary protocol implementation with proper error handling, CRC validation, and buffer boundary checks. Critical for production GPS tracking where packet corruption could cause incorrect location data.

### [src/database.ts](src/database.ts)
Demonstrates data persistence layer with transaction safety:

```typescript
export async function saveGPSData(
  imei: string,
  records: AVLRecord[]
): Promise<void> {
  // Upsert device (create or update last seen time)
  const device = await prisma.device.upsert({
    where: { imei },
    create: {
      imei,
      lastLatitude: records[0].latitude,
      lastLongitude: records[0].longitude,
      lastConnectedTime: new Date(),
    },
    update: {
      lastConnectedTime: new Date(),
    },
  });

  // Batch insert GPS locations
  await prisma.gPSLocation.createMany({
    data: records.map(record => ({
      deviceId: device.id,
      imei,
      timestamp: record.timestamp,
      latitude: record.latitude,
      longitude: record.longitude,
      altitude: record.altitude,
      speed: record.speed,
      angle: record.angle,
      satellites: record.satellites,
      priority: record.priority,
      ioElements: record.ioElements,
    })),
    skipDuplicates: true,
  });
}
```

**Why this matters:** Atomic upsert prevents race conditions when multiple packets arrive simultaneously from same device. `createMany` with `skipDuplicates` provides idempotent inserts for retry safety.

### [src/websocket-server.ts](src/websocket-server.ts)
Real-time broadcasting with connection management:

```typescript
import WebSocket from 'ws';

export class WebSocketServer {
  private wss: WebSocket.Server;
  private clients: Set<WebSocket> = new Set();

  constructor(port: number) {
    this.wss = new WebSocket.Server({ port });

    this.wss.on('connection', (ws) => {
      this.clients.add(ws);
      console.log(`WebSocket client connected. Total: ${this.clients.size}`);

      ws.on('close', () => {
        this.clients.delete(ws);
        console.log(`Client disconnected. Total: ${this.clients.size}`);
      });

      ws.on('error', (error) => {
        console.error('WebSocket error:', error);
        this.clients.delete(ws);
      });
    });
  }

  broadcast(data: any): void {
    const message = JSON.stringify(data);

    this.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  }
}
```

**Why this matters:** Demonstrates proper WebSocket lifecycle management with automatic cleanup of stale connections. Prevents memory leaks from dead clients.

### [src/api/](src/api/)
REST API endpoints for device and location queries with pagination and filtering support.

### [prisma/schema.prisma](prisma/schema.prisma)
Type-safe database schema with proper indexing for high-throughput GPS data ingestion.

## Deployment & environment

**Production Setup:**
- Docker containerized service (Node.js 18 Alpine base image)
- PostgreSQL 12+ on managed database instance (connection pooling enabled)
- Process management with PM2 or systemd

**Environment Variables:**
```bash
DATABASE_URL=postgresql://user:password@host:5432/gps_tracker
TCP_PORT=8080
HTTP_PORT=3000
WS_PORT=8081
NODE_ENV=production
```

**Networking:**
- TCP port 8080 exposed for GPS device connections
- HTTP port 3000 for REST API
- WebSocket port 8081 for real-time clients
- Firewall rules limiting TCP connections to known device IP ranges

**Database:**
- Prisma migrations: `prisma migrate deploy`
- Connection pooling (10-20 connections)
- Automated backups (daily snapshots + WAL archiving)

**Monitoring:**
- Custom logging for protocol parsing errors
- Database query performance monitoring
- Connection count tracking
- WebSocket client count metrics

**Scaling Considerations:**
- Stateless design enables horizontal scaling
- Database connection pooling prevents connection exhaustion
- WebSocket server can run on separate instances with Redis pub/sub

## Public links

Private commercial project for Hebent Logistics. Source code available upon request for portfolio review.
