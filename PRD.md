# Product Requirements Document (PRD)
## AssetFlow Pro - Multi-Tenant SaaS Asset Management System

**Version:** 1.0  
**Last Updated:** January 15, 2025  
**Status:** In Development

---

## 1. Executive Summary

AssetFlow Pro is an enterprise-grade, multi-tenant SaaS platform for asset management with strict data isolation, hierarchical user management, ISO-compliant asset tracking, and intelligent notification workflows. The system ensures complete tenant privacy while providing SuperAdmins platform-wide control over branding and subscriptions.

### 1.1 Vision
To provide organizations with a secure, scalable asset management solution that enforces data privacy at the database level while enabling efficient collaboration between planners and operators.

### 1.2 Goals
- Implement complete tenant data isolation using Row-Level Security (RLS)
- Enable hierarchical role-based access control (RBAC) with 5 distinct roles
- Track assets with ISO-compliant fields (maintenance cycles, safety logs, certifications)
- Provide an intelligent notification workflow with merge capabilities
- Support multi-tenant branding and subscription management

---

## 2. System Architecture

### 2.1 Multi-Tenancy Strategy

| Aspect | Implementation |
|--------|----------------|
| **Database Strategy** | Row-Level Security (RLS) with `tenant_id` on all operational tables |
| **Access Control** | API layer enforces "Tenant Scope" on every request |
| **SuperAdmin Privacy** | No `tenant_id` association for Assets/Notifications tables |

### 2.2 Technology Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 18, TypeScript, Vite |
| UI Framework | Tailwind CSS, shadcn/ui |
| State Management | TanStack Query |
| Routing | React Router v6 |
| Backend | Cloudflare Workers + Hono |
| Database | Cloudflare D1 (SQLite) |
| Authentication | Cloudflare Access + Workers Auth |
| Real-time | Cloudflare Durable Objects |
| File Storage | Cloudflare R2 |
| CDN | Cloudflare CDN |
| Edge Compute | Cloudflare Workers |

---

## 3. User Roles & Permissions

### 3.1 Role Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                      SUPERADMIN                              │
│              (Platform-wide, no tenant context)              │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                     TENANT ADMIN                             │
│                  (Level 1 - Full tenant control)             │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│        PLANNER          │     │        OPERATOR         │
│  (Level 2 - Assets &    │     │  (Level 2 - Submit      │
│   Notifications)        │     │   Notifications)        │
└─────────────────────────┘     └─────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                      EXECUTIVE                               │
│                    (View-Only Access)                        │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 RBAC Permission Matrix

| Feature | SuperAdmin | TenantAdmin | Planner | Operator | Executive |
|---------|:----------:|:-----------:|:-------:|:--------:|:---------:|
| Global Branding | ✅ | ❌ | ❌ | ❌ | ❌ |
| Subscription Settings | ✅ | ❌ | ❌ | ❌ | ❌ |
| Theme Presets | ✅ | ❌ | ❌ | ❌ | ❌ |
| Tenant Branding | ❌ | ✅ | ❌ | ❌ | ❌ |
| User Management | ❌ | ✅ | ❌ | ❌ | ❌ |
| Asset Management | ❌ | ❌ | ✅ | ❌ | 👁️ View |
| Create Notification | ❌ | ❌ | ❌ | ✅ | ❌ |
| Submit Notification | ❌ | ❌ | ❌ | ✅ | ❌ |
| Merge Notifications | ❌ | ❌ | ✅ | ❌ | ❌ |
| Solve Notifications | ❌ | ❌ | ✅ | ❌ | ❌ |
| View Dashboard | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## 4. Core Modules

### 4.1 Global Identity (SuperAdmin Only)

#### 4.1.1 App Branding
- **App Name**: Platform display name
- **Description**: Platform description text
- **Version**: Semantic version number
- **Logo URL**: Platform logo (header, login page)
- **Favicon URL**: Browser tab icon

#### 4.1.2 Theme Engine
Global theme presets (CSS variable sets) available for tenants:
- Color palette (primary, secondary, accent)
- Typography settings
- Border radius values
- Shadow definitions

#### 4.1.3 Subscription Engine

| Tier | Asset Limit | Features |
|------|-------------|----------|
| FREE | 2 Assets | Basic asset tracking, 1 user |
| PRO | Unlimited | Full features, unlimited users |

### 4.2 Tenant Identity (TenantAdmin)

- **Company Name**: Tenant organization name
- **Company Logo**: Custom branding for tenant views
- **Theme Selection**: Choose from SuperAdmin-defined presets
- **User Hierarchy Management**: Create/manage subordinate users

### 4.3 Asset Management (Planner)

#### 4.3.1 Asset Fields
| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `asset_name` | String | ✅ | Display name |
| `asset_code` | String | ✅ | Unique identifier (e.g., CNC-001) |
| `category` | String | ✅ | Classification (Manufacturing, Automation, etc.) |
| `location` | String | ✅ | Physical location |
| `status` | Enum | ✅ | active, maintenance, retired |
| `tenant_id` | UUID | ✅ | Owner tenant (auto-set) |

#### 4.3.2 ISO Compliance Fields
| Field | Type | Description |
|-------|------|-------------|
| `maintenance_cycle` | String | Frequency (e.g., "30 days", "quarterly") |
| `last_maintenance_date` | Date | Most recent maintenance |
| `next_maintenance_date` | Date | Upcoming scheduled maintenance |
| `technical_specs` | Text | Detailed specifications |
| `certifications` | Array | ISO 9001, CE Mark, TÜV, etc. |

#### 4.3.3 Safety Logs
| Field | Type | Description |
|-------|------|-------------|
| `date` | Date | Inspection date |
| `description` | Text | Inspection notes |
| `inspector_name` | String | Inspector identity |
| `passed` | Boolean | Pass/fail status |

### 4.4 Notification System

#### 4.4.1 Notification Fields
| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `title` | String | ✅ | Brief summary |
| `description` | Text | ✅ | Detailed explanation |
| `asset_id` | UUID | ✅ | Related asset |
| `reporter_id` | UUID | ✅ | Operator who created |
| `status` | Enum | ✅ | Current lifecycle stage |
| `priority` | Enum | ✅ | low, medium, high, critical |
| `tenant_id` | UUID | ✅ | Owner tenant (auto-set) |
| `merged_to_id` | UUID | ❌ | Parent notification if merged |

#### 4.4.2 Status Lifecycle

```
┌──────────┐    Submit    ┌───────────┐    Start    ┌──────────┐    Resolve   ┌────────┐
│  DRAFT   │ ──────────▶ │ SUBMITTED │ ──────────▶ │ PROGRESS │ ───────────▶ │ SOLVED │
└──────────┘              └───────────┘              └──────────┘              └────────┘
     │                          │
     │                          │  Merge
     │                          ▼
     │                    ┌──────────┐
     └───────────────────▶│  MERGED  │
          (Draft can be    └──────────┘
           merged too)
```

| Status | Description | Set By |
|--------|-------------|--------|
| `DRAFT` | Created, not yet visible to Planner | Operator |
| `SUBMITTED` | Sent to Planner for review | Operator |
| `MERGED` | Linked to existing notification | Planner |
| `PROGRESS` | Work in progress | Planner |
| `SOLVED` | Issue resolved (final state) | Planner |

#### 4.4.3 Merge Logic
When multiple notifications exist for the same asset issue:
1. Planner selects 2+ notifications
2. Chooses a primary notification
3. Secondary notifications move to `MERGED` status
4. `merged_to_id` references the primary
5. Updates sync across merged tickets

---

## 5. Database Schema

### 5.1 Entity Relationship Diagram

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│  GlobalConfig   │       │     Tenant      │       │   Subscription  │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ id              │       │ id              │◄──────│ id              │
│ app_name        │       │ tenant_name     │       │ tier            │
│ description     │       │ company_logo    │       │ max_assets      │
│ version         │       │ subscription_id │───────│ features[]      │
│ logo_url        │       │ created_at      │       └─────────────────┘
│ favicon_url     │       └─────────────────┘
│ theme_presets[] │               │
└─────────────────┘               │
                                  │ 1:N
                                  ▼
                    ┌─────────────────────────┐
                    │         User            │
                    ├─────────────────────────┤
                    │ id                      │
                    │ name                    │
                    │ email                   │
                    │ role                    │
                    │ tenant_id (nullable)    │◄─── NULL for SuperAdmin
                    │ parent_id               │───▶ Self-reference
                    │ created_at              │
                    └─────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
        ┌───────────────────┐       ┌───────────────────┐
        │      Asset        │       │   Notification    │
        ├───────────────────┤       ├───────────────────┤
        │ id                │◄──────│ id                │
        │ asset_name        │       │ asset_id          │
        │ asset_code        │       │ reporter_id       │───▶ User
        │ tenant_id         │       │ status            │
        │ category          │       │ priority          │
        │ location          │       │ tenant_id         │
        │ status            │       │ merged_to_id      │───▶ Self
        │ iso_data (JSONB)  │       │ title             │
        │ created_at        │       │ description       │
        │ updated_at        │       │ created_at        │
        └───────────────────┘       │ updated_at        │
                                    └───────────────────┘
```

### 5.2 SQL Schema (Cloudflare D1)

```sql
-- Note: D1 uses SQLite syntax, not PostgreSQL
-- Enums are implemented as TEXT with CHECK constraints

-- Global Config (SuperAdmin only)
CREATE TABLE global_config (
    id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    app_name TEXT NOT NULL,
    description TEXT,
    version TEXT,
    logo_url TEXT,
    favicon_url TEXT,
    theme_presets TEXT DEFAULT '[]', -- JSON stored as TEXT
    updated_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- Tenants
CREATE TABLE tenants (
    id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    tenant_name TEXT NOT NULL,
    company_logo TEXT,
    subscription_tier TEXT DEFAULT 'FREE' CHECK(subscription_tier IN ('FREE', 'PRO')),
    created_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- Auth Users (managed by Workers)
CREATE TABLE auth_users (
    id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    email TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    email_verified INTEGER DEFAULT 0,
    created_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- User Roles (separate table for security)
CREATE TABLE user_roles (
    id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    user_id TEXT NOT NULL REFERENCES auth_users(id) ON DELETE CASCADE,
    role TEXT NOT NULL CHECK(role IN ('superadmin', 'tenantadmin', 'planner', 'operator', 'executive')),
    UNIQUE (user_id, role)
);

-- User Profiles
CREATE TABLE profiles (
    id TEXT PRIMARY KEY REFERENCES auth_users(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    email TEXT NOT NULL,
    tenant_id TEXT REFERENCES tenants(id) ON DELETE CASCADE,
    parent_id TEXT REFERENCES profiles(id),
    avatar_url TEXT,
    created_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- Assets
CREATE TABLE assets (
    id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    asset_name TEXT NOT NULL,
    asset_code TEXT NOT NULL,
    tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    category TEXT NOT NULL,
    location TEXT NOT NULL,
    status TEXT DEFAULT 'active' CHECK(status IN ('active', 'maintenance', 'retired')),
    iso_data TEXT DEFAULT '{}', -- JSON stored as TEXT
    created_at INTEGER DEFAULT (strftime('%s', 'now')),
    updated_at INTEGER DEFAULT (strftime('%s', 'now')),
    UNIQUE (tenant_id, asset_code)
);

-- Notifications
CREATE TABLE notifications (
    id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    asset_id TEXT NOT NULL REFERENCES assets(id) ON DELETE CASCADE,
    reporter_id TEXT NOT NULL REFERENCES profiles(id),
    status TEXT DEFAULT 'DRAFT' CHECK(status IN ('DRAFT', 'SUBMITTED', 'MERGED', 'PROGRESS', 'SOLVED')),
    priority TEXT DEFAULT 'medium' CHECK(priority IN ('low', 'medium', 'high', 'critical')),
    tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    merged_to_id TEXT REFERENCES notifications(id),
    title TEXT NOT NULL,
    description TEXT,
    created_at INTEGER DEFAULT (strftime('%s', 'now')),
    updated_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- Indexes for performance
CREATE INDEX idx_assets_tenant ON assets(tenant_id);
CREATE INDEX idx_notifications_tenant ON notifications(tenant_id);
CREATE INDEX idx_notifications_asset ON notifications(asset_id);
CREATE INDEX idx_profiles_tenant ON profiles(tenant_id);
CREATE INDEX idx_user_roles_user ON user_roles(user_id);
```

---

## 6. Security Requirements

### 6.1 Data Isolation Strategy (Cloudflare Workers)

**Note:** D1 does not support Row-Level Security (RLS) like PostgreSQL. Security is enforced at the **Cloudflare Workers API layer**.

#### 6.1.1 Worker Middleware for Tenant Isolation

```typescript
// middleware/auth.ts
import { Context } from 'hono';
import { verify } from 'hono/jwt';

interface JWTPayload {
  userId: string;
  tenantId: string | null;
  role: string;
}

export async function authMiddleware(c: Context, next: Function) {
  const token = c.req.header('Authorization')?.replace('Bearer ', '');
  
  if (!token) {
    return c.json({ error: 'Unauthorized' }, 401);
  }

  try {
    const payload = await verify(token, c.env.JWT_SECRET) as JWTPayload;
    c.set('userId', payload.userId);
    c.set('tenantId', payload.tenantId);
    c.set('role', payload.role);
    await next();
  } catch (error) {
    return c.json({ error: 'Invalid token' }, 401);
  }
}

export function requireTenant(c: Context) {
  const tenantId = c.get('tenantId');
  if (!tenantId) {
    throw new Error('Tenant context required');
  }
  return tenantId;
}

export function hasRole(c: Context, role: string): boolean {
  return c.get('role') === role;
}
```

#### 6.1.2 Tenant-Scoped Queries

```typescript
// Example: Get assets for current tenant only
app.get('/api/assets', authMiddleware, async (c) => {
  const tenantId = requireTenant(c);
  
  const result = await c.env.DB.prepare(
    'SELECT * FROM assets WHERE tenant_id = ?'
  ).bind(tenantId).all();
  
  return c.json(result.results);
});

// Example: Create asset with auto-injected tenant_id
app.post('/api/assets', authMiddleware, async (c) => {
  const tenantId = requireTenant(c);
  const body = await c.req.json();
  
  // Force tenant_id from JWT, ignore client input
  const result = await c.env.DB.prepare(
    'INSERT INTO assets (asset_name, asset_code, tenant_id, ...) VALUES (?, ?, ?, ...)'
  ).bind(body.asset_name, body.asset_code, tenantId, ...).run();
  
  return c.json({ success: true });
});
```

### 6.2 SuperAdmin Privacy Shield

**Critical Requirement:** SuperAdmins must NOT have access to tenant operational data.

```typescript
// middleware/superadmin-guard.ts
export function blockSuperAdminFromTenantData(c: Context) {
  const role = c.get('role');
  
  if (role === 'superadmin') {
    throw new Error('SuperAdmins cannot access tenant data');
  }
}

// Apply to tenant routes
app.get('/api/assets', authMiddleware, async (c) => {
  blockSuperAdminFromTenantData(c);
  const tenantId = requireTenant(c);
  // ... query assets
});
```

### 6.3 Real-time Security with Durable Objects

```typescript
// durable-objects/NotificationRoom.ts
export class NotificationRoom {
  private tenantId: string;
  
  constructor(state: DurableObjectState, env: Env) {
    this.state = state;
    this.env = env;
  }
  
  async fetch(request: Request) {
    // Verify tenant context before allowing WebSocket connection
    const token = new URL(request.url).searchParams.get('token');
    const payload = await verify(token, this.env.JWT_SECRET);
    
    if (!payload.tenantId) {
      return new Response('Unauthorized', { status: 401 });
    }
    
    this.tenantId = payload.tenantId;
    // ... handle WebSocket connection
  }
}
```

### 6.4 API Layer Validation (Cloudflare Workers)

Every API request must:
1. Authenticate the user via JWT (signed with Workers secrets)
2. Extract `tenant_id` from JWT payload
3. Inject `tenant_id` into all D1 queries
4. Return empty set or 403 if tenant context is missing
5. Validate rate limits using Cloudflare Workers KV
6. Log all mutations to Cloudflare Analytics Engine for audit trail

---

## 7. User Interface Requirements

### 7.1 Design System

| Token | Light Mode | Dark Mode |
|-------|------------|-----------|
| `--background` | 209 40% 96% | 222 47% 11% |
| `--foreground` | 222 47% 11% | 210 40% 98% |
| `--primary` | 216 19% 26% | 212 26% 83% |
| `--secondary` | 215 19% 34% | 215 19% 34% |
| `--muted` | 215 20% 65% | 215 16% 46% |
| `--destructive` | 0 72% 50% | 0 84% 60% |

### 7.2 Status Color Tokens

| Status | Color Token |
|--------|-------------|
| DRAFT | `--status-draft: 215 16% 46%` |
| SUBMITTED | `--status-submitted: 217 91% 60%` |
| MERGED | `--status-merged: 271 91% 65%` |
| PROGRESS | `--status-progress: 45 93% 47%` |
| SOLVED | `--status-solved: 142 76% 36%` |

### 7.3 Page Structure

#### Landing Page
- Hero section with CTAs
- Feature grid (8 features)
- Role showcase cards
- Security highlights
- Footer with navigation

#### Dashboard Layout
- Fixed header with user menu
- Collapsible sidebar (role-filtered navigation)
- Main content area
- Responsive design (mobile-first)

---

## 8. API Endpoints

### 8.1 Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/signup` | Register new user |
| POST | `/auth/signin` | Login |
| POST | `/auth/signout` | Logout |
| GET | `/auth/session` | Get current session |

### 8.2 Assets
| Method | Endpoint | Description | Roles |
|--------|----------|-------------|-------|
| GET | `/assets` | List tenant assets | Planner, Executive |
| GET | `/assets/:id` | Get asset details | Planner, Executive |
| POST | `/assets` | Create asset | Planner |
| PATCH | `/assets/:id` | Update asset | Planner |
| DELETE | `/assets/:id` | Delete asset | Planner |

### 8.3 Notifications
| Method | Endpoint | Description | Roles |
|--------|----------|-------------|-------|
| GET | `/notifications` | List notifications | Planner, Operator |
| GET | `/notifications/:id` | Get notification | Planner, Operator |
| POST | `/notifications` | Create notification | Operator |
| PATCH | `/notifications/:id` | Update notification | Planner, Operator |
| POST | `/notifications/merge` | Merge notifications | Planner |
| PATCH | `/notifications/:id/status` | Update status | Planner |

### 8.4 Users
| Method | Endpoint | Description | Roles |
|--------|----------|-------------|-------|
| GET | `/users` | List tenant users | TenantAdmin |
| POST | `/users/invite` | Invite user | TenantAdmin |
| PATCH | `/users/:id` | Update user | TenantAdmin |
| DELETE | `/users/:id` | Remove user | TenantAdmin |

### 8.5 Admin
| Method | Endpoint | Description | Roles |
|--------|----------|-------------|-------|
| GET | `/admin/config` | Get global config | SuperAdmin |
| PATCH | `/admin/config` | Update global config | SuperAdmin |
| GET | `/admin/tenants` | List all tenants | SuperAdmin |
| PATCH | `/admin/tenants/:id` | Update tenant tier | SuperAdmin |

---

## 9. Non-Functional Requirements

### 9.1 Performance
- Page load time: < 2 seconds
- API response time: < 500ms
- Real-time updates: < 100ms latency

### 9.2 Scalability
- Support 1000+ concurrent users
- Handle 10,000+ assets per tenant
- Support 100+ tenants

### 9.3 Availability
- 99.9% uptime SLA
- Automatic failover
- Daily backups with 30-day retention

### 9.4 Compliance
- ISO 9001 asset tracking fields
- GDPR data handling
- SOC 2 Type II security controls
- Audit logging for all mutations

---

## 10. Release Plan

### Phase 1: MVP (Current)
- [x] Landing page with role showcase
- [x] Dashboard layouts for all roles
- [x] Asset management UI (mock data)
- [x] Notification workflow UI (mock data)
- [x] User management UI (mock data)
- [x] Responsive design

### Phase 2: Backend Integration
- [ ] Cloudflare D1 database setup
- [ ] D1 security policies implementation
- [ ] Cloudflare Workers authentication flow
- [ ] CRUD operations for assets via Workers
- [ ] Notification lifecycle logic in Workers
- [ ] User invitation system via Workers

### Phase 3: Advanced Features
- [ ] Real-time notifications via Durable Objects
- [ ] Notification merge functionality
- [ ] File attachments using Cloudflare R2
- [ ] Export reports (PDF/CSV) via Workers
- [ ] Email notifications via Cloudflare Email Workers

### Phase 4: Enterprise
- [ ] SSO integration via Cloudflare Access (SAML/OIDC)
- [ ] Advanced analytics with Cloudflare Analytics Engine
- [ ] API access for integrations via Workers
- [ ] White-label support
- [ ] Mobile app (Capacitor) with Workers backend

---

## 11. Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| User Adoption | 80% DAU/MAU | Analytics |
| Notification Resolution Time | < 48 hours | Database |
| Asset Uptime | > 95% active | Status tracking |
| User Satisfaction | NPS > 50 | Surveys |
| System Uptime | > 99.9% | Monitoring |

---

## 12. Appendix

### A. Glossary

| Term | Definition |
|------|------------|
| **Tenant** | An organization using the platform |
| **Asset** | Physical equipment or machinery being tracked |
| **Notification** | Issue report on an asset |
| **RLS** | Row-Level Security - database access control |
| **RBAC** | Role-Based Access Control |
| **ISO Data** | Compliance fields per ISO standards |

### B. References

- [Cloudflare D1 Documentation](https://developers.cloudflare.com/d1/)
- [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)
- [Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/)
- [Cloudflare R2 Storage](https://developers.cloudflare.com/r2/)
- [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/identity/access/)
- [ISO 55000 Asset Management](https://www.iso.org/standard/55088.html)
- [shadcn/ui Components](https://ui.shadcn.com/)
- [Hono Framework](https://hono.dev/)

---

*Document maintained by the AssetFlow Pro development team.*
