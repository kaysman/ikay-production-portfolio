# Route and Route - Logistics SaaS Platform

## What it is

A full-stack logistics management platform that connects shippers with truck carriers. Handles quote generation, order management, cargo-to-truck assignment, real-time shipment tracking with interactive maps, and trip phase management (pickup, in-transit, delivered). Built as a monorepo with separate API backend, admin dashboard, client interface, and landing page.

## My role

I architected and developed the complete platform end-to-end as a personal SaaS project. Responsible for full-stack implementation including database design (Prisma schema modeling), REST API with Express, Next.js admin interface with Zustand state management, Zod validation layer, and OpenStreetMap integration. Implemented distance calculations using Haversine formula and designed the complete cargo-shipment-trip workflow engine.

## Tech stack

### Backend (Express API)

**Runtime & Language:**

- Node.js with TypeScript 5+
- ES Modules (type: "module")

**Framework & Architecture:**

- Express 5.0.0-beta.1
- Layered architecture (routes → controllers → services → repositories)
- Repository pattern for data access

**Database & ORM:**

- PostgreSQL with Prisma ORM 7.0.1
- Type-safe query builder
- Automatic migration generation

**Validation & Security:**

- Zod for runtime schema validation
- JWT authentication with bcrypt
- Helmet (security headers)
- CORS configuration
- Rate limiting

**Logging:**

- Pino (structured JSON logging)
- Pino Pretty (development formatting)

### Frontend (Admin Panel)

**Framework:**

- Next.js 15 (App Router)
- React 18.3.0
- TypeScript

**UI & Styling:**

- Radix UI components (Accordion, Dialog, DropdownMenu, Select, Tabs, Toast)
- Tailwind CSS 3.4
- Lucide React icons
- Responsive design patterns

**State Management:**

- Zustand 5.0.8 (lightweight state management)
- React Hook Form with Zod validation

**Maps:**

- React Leaflet (OpenStreetMap integration)
- No API key required
- Custom marker and tile layer configuration

**HTTP Client:**

- Axios for API communication

**Internationalization:**

- next-intl for multi-language support

### Monorepo Structure

**Package Manager:**

- pnpm 10.20.0 with workspace support

**Projects:**

- `logistics-api/` - Express backend
- `frontend-admin/` - Next.js admin dashboard
- `frontend-client/` - Client-facing interface
- `landing-site/` - Marketing site
- `shared/` - Shared TypeScript types and utilities

## System architecture

### Backend Architecture (Layered + Repository Pattern)

```
logistics-api/src/
├── index.ts                    # Server entry point
├── config/                     # Configuration management
├── middlewares/                # Express middleware chain
│   ├── auth.middleware.ts     # JWT verification
│   ├── validation.middleware.ts
│   └── error-handler.ts
├── modules/                    # Feature modules
│   ├── cargos/
│   │   ├── controller/        # HTTP request handlers
│   │   │   ├── order-cargos.controller.ts
│   │   │   └── cargos.base.controller.ts
│   │   ├── service/           # Business logic
│   │   ├── repository/        # Data access layer
│   │   │   ├── order-cargos.repository.ts
│   │   │   └── cargos.base.repository.ts
│   │   ├── types/             # TypeScript interfaces
│   │   ├── order-cargos.routes.ts
│   │   └── quote-cargos.routes.ts
│   ├── fleet/                 # Truck management
│   ├── orders/                # Order processing
│   ├── quotes/                # Quote generation
│   ├── calculation-trips/     # Trip calculations
│   ├── users/                 # User management
│   ├── company-auth/          # Authentication
│   └── ... (14 modules total)
├── shared/                     # Shared utilities
├── types/                      # Global types
├── generated/                  # Prisma client
└── prisma/                     # Database schema
    └── schema.prisma
```

### Request Flow (Backend)

```
HTTP Request
    ↓
Express Router
    ↓
Middleware Chain (auth → validation → rate-limit)
    ↓
Controller (request parsing, response formatting)
    ↓
Service Layer (business logic, transaction coordination)
    ↓
Repository Pattern (Prisma queries, data mapping)
    ↓
PostgreSQL Database
```

### Frontend Architecture (Next.js App Router + Zustand)

```
frontend-admin/
├── app/                        # Next.js App Router
│   ├── layout.tsx             # Root layout
│   ├── page.tsx               # Dashboard
│   ├── orders/                # Order management
│   ├── fleet/                 # Truck management
│   ├── shipments/             # Shipment tracking
│   └── ... (feature pages)
├── components/                 # Reusable UI components
│   ├── sections/              # Page sections
│   ├── ui/                    # Radix UI wrappers
│   └── maps/                  # Leaflet map components
├── store/                      # Zustand stores
│   ├── shipments-store.ts
│   ├── orders-store.ts
│   └── fleet-store.ts
├── lib/                        # Utilities
│   ├── api-client.ts          # Axios configuration
│   └── utils.ts               # Helper functions
└── types/                      # TypeScript definitions
```

### Database Schema (Key Models)

```prisma
model Cargo {
  id          String   @id @default(cuid())
  orderId     String
  description String
  weight      Decimal
  volume      Decimal
  pickupLat   Decimal  @db.Decimal(10, 8)
  pickupLng   Decimal  @db.Decimal(11, 8)
  deliveryLat Decimal  @db.Decimal(10, 8)
  deliveryLng Decimal  @db.Decimal(11, 8)
  status      CargoStatus

  order       Order    @relation(fields: [orderId], references: [id])
  shipments   Shipment[]

  @@index([orderId])
  @@index([status])
}

model Truck {
  id          String   @id @default(cuid())
  fleetId     String
  plateNumber String   @unique
  capacity    Decimal
  currentLat  Decimal? @db.Decimal(10, 8)
  currentLng  Decimal? @db.Decimal(11, 8)
  status      TruckStatus

  fleet       Fleet    @relation(fields: [fleetId], references: [id])
  trips       Trip[]

  @@index([fleetId])
  @@index([status])
}

model Trip {
  id          String   @id @default(cuid())
  shipmentId  String
  truckId     String
  phase       TripPhase // PICKUP, IN_TRANSIT, DELIVERED
  startTime   DateTime?
  endTime     DateTime?
  distance    Decimal?

  shipment    Shipment @relation(fields: [shipmentId], references: [id])
  truck       Truck    @relation(fields: [truckId], references: [id])

  @@index([shipmentId])
  @@index([truckId])
  @@index([phase])
}
```

## Key technical decisions

**Repository pattern over direct Prisma usage:**
Abstracted data access through repository classes. Enabled switching database implementations, centralized query logic, and simplified testing with mock repositories. All service-layer code depends on repository interfaces, not Prisma client directly.

**Zod for runtime validation:**
Express doesn't provide type safety for request bodies. Zod schemas validate incoming data at runtime and infer TypeScript types at compile time. Single source of truth for both validation and typing prevents drift between runtime checks and type definitions.

**Zustand over Redux:**
Chose Zustand for frontend state management due to minimal boilerplate, zero context provider nesting, and selector-based subscriptions. Admin dashboard has complex state (active shipment, selected trucks, map bounds) that Redux would handle with excessive ceremony.

**OpenStreetMap over Google Maps:**
No API key requirements, no usage limits, no billing concerns. React Leaflet provides same feature set (markers, polylines, custom tiles) as Google Maps. Self-hosted tile servers possible for production if needed.

**pnpm monorepo over separate repos:**
Shared TypeScript types between frontend and backend through `shared/` package. Single dependency installation, atomic cross-package changes, and consistent versioning. Prevents API contract drift.

**Decimal for coordinates and weights:**
PostgreSQL DECIMAL type prevents floating-point rounding errors critical for:

- Geofencing (is truck within 500m of pickup?)
- Distance calculations (Haversine formula)
- Weight/volume calculations (cargo capacity validation)

**Express 5 beta over Express 4:**
Native Promise support in middleware eliminates async wrapper functions. Improved error handling with async/await. TypeScript definitions significantly better than Express 4.

## Notable challenges solved

**Cargo-to-truck assignment algorithm:**
Challenge: Matching available trucks to cargo based on capacity, current location, and schedule. Solution: Implemented scoring algorithm considering truck capacity utilization, distance from pickup point (Haversine), and driver availability. Prevents overloading and optimizes route efficiency.

**Real-time trip phase management:**
Challenge: Tracking shipment phases (pickup → in-transit → delivered) with state transitions. Solution: State machine pattern with Prisma transactions ensuring atomic phase updates. Phase transitions trigger WebSocket events for real-time dashboard updates.

**Multi-cargo shipment consolidation:**
Challenge: Multiple cargos assigned to single truck need consolidated route planning. Solution: Designed normalized database schema separating Cargo, Shipment, and Trip entities. Many-to-many relationship through Shipment junction table with ordered pickup/delivery sequence.

**Type-safe API contracts:**
Challenge: Frontend and backend TypeScript types drifting causing runtime errors. Solution: Shared `@logistics/types` package in monorepo workspace. Zod schemas export inferred types used by both API and UI. Schema changes break compilation in both projects.

**Distance calculation accuracy:**
Challenge: Simple lat/lng subtraction gives incorrect distances due to Earth curvature. Solution: Haversine formula implementation accounting for spherical geometry. Critical for accurate ETA calculations and geofencing (e.g., "truck within 1km of delivery point").

**Nested resource routing:**
Challenge: REST API for nested resources like `/orders/:orderId/cargos/:cargoId` with proper authorization. Solution: Base controller pattern with `getCompanyId()`, `getParentId()`, `getCargoId()` extractors. Validates user has access to parent order before accessing child cargo.

## Code highlights

### [logistics-api/src/modules/cargos/controller/order-cargos.controller.ts](logistics-api/src/modules/cargos/controller/order-cargos.controller.ts)

Demonstrates layered architecture with inheritance-based controller pattern:

```typescript
import type { NextFunction, Request, Response } from "express";
import { orderCargosService } from "../service/order-cargos.service.js";
import { CargosBaseController } from "./cargos.base.controller.js";
import type {
  CargoFilter,
  OrderCargoQueryDTO,
} from "../types/order-cargo.types.js";

class OrderCargosController extends CargosBaseController {
  async list(req: Request, res: Response, next: NextFunction) {
    try {
      const companyId = this.getCompanyId(req);
      const orderId = this.getParentId(req, "orderId");
      const filter = (req.query as OrderCargoQueryDTO).filter as
        | CargoFilter
        | undefined;
      const result = await orderCargosService.list(companyId, orderId, filter);
      return this.handleList(res, result);
    } catch (error) {
      return this.handleError(next, error);
    }
  }

  async get(req: Request, res: Response, next: NextFunction) {
    try {
      const companyId = this.getCompanyId(req);
      const orderId = this.getParentId(req, "orderId");
      const cargoId = this.getCargoId(req);
      const result = await orderCargosService.get(companyId, orderId, cargoId);
      return this.handleGet(res, result);
    } catch (error) {
      return this.handleError(next, error);
    }
  }

  async create(req: Request, res: Response, next: NextFunction) {
    try {
      const companyId = this.getCompanyId(req);
      const orderId = this.getParentId(req, "orderId");
      const userId = this.getUserId(req);
      const result = await orderCargosService.create(
        companyId,
        orderId,
        userId,
        req.body
      );
      return this.handleCreate(res, result);
    } catch (error) {
      return this.handleError(next, error);
    }
  }
}

export const orderCargosController = new OrderCargosController();
```

**Why this matters:** Base controller pattern eliminates repetitive authorization logic. `getCompanyId()` ensures multi-tenant isolation - users can only access their company's data. Error handling centralized through `handleError()`.

### [logistics-api/src/modules/cargos/repository/](logistics-api/src/modules/cargos/repository/)

Repository pattern abstracts Prisma queries. Base repository provides common CRUD operations, specific repositories extend for custom queries. Enables testing with mock repositories.

### [frontend-admin/ - Next.js App Router](frontend-admin/)

Demonstrates modern Next.js patterns:

- Server Components for initial data fetching
- Client Components for interactive maps
- Zustand for global state (selected shipment, active filters)
- React Hook Form + Zod for form validation

### [prisma/schema.prisma](prisma/schema.prisma)

Normalized schema for logistics domain:

- `Cargo` → `Shipment` → `Trip` relationship
- `Fleet` → `Truck` hierarchy
- `Order` → `Quote` workflow
- Proper indexing on foreign keys and status columns

## Deployment & environment

**Backend Deployment:**

- Node.js server on VPS (DigitalOcean/AWS/Hetzner)
- PM2 process manager for auto-restart
- PostgreSQL on managed database instance (connection pooling)
- Nginx reverse proxy with SSL (Let's Encrypt)

**Frontend Deployment:**

- Next.js static export (`next build && next export`)
- Deployed to Vercel/Netlify or served via Nginx
- Environment variables for API base URL

**Database:**

```bash
# Prisma migrations
pnpm --filter logistics-api prisma:migrate

# Development
pnpm --filter logistics-api prisma:studio
```

**Environment Variables:**

```bash
# Backend
DATABASE_URL=postgresql://user:password@host:5432/logistics_db
JWT_SECRET=<secure-random-string>
PORT=5000
NODE_ENV=production

# Frontend
NEXT_PUBLIC_API_URL=https://api.routeandroute.example.com
```

**Monorepo Commands:**

```bash
# Install all dependencies
pnpm install

# Start API dev server
pnpm --filter logistics-api dev

# Start admin frontend
pnpm --filter frontend-admin dev

# Build all projects
pnpm -r build
```

**Production Considerations:**

- Rate limiting to prevent abuse
- JWT token expiration and refresh
- Pino logging to file with rotation
- Database connection pooling (10-20 connections)
- Nginx caching for static assets

## Public links

Personal SaaS project. Demo available upon request for portfolio review.
