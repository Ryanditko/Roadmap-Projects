# Multi-Tenant SaaS Platform

<p align="center">
  <img src="https://images.unsplash.com/photo-1551434678-e076c223a692?w=600&q=80" alt="SaaS Platform" width="600">
</p>

**Level:** Advanced  
**Estimated Time:** 4-6 weeks  
**Prerequisites:** Full-stack development, Database design, Payment integration, API development, Authentication systems

---

## Description

Build a production-ready Software as a Service (SaaS) platform with multi-tenancy, subscription management, team workspaces, and usage-based billing. This comprehensive project teaches you how to architect, develop, and deploy a scalable SaaS application that can serve thousands of customers.

The platform implements complete tenant isolation, flexible subscription plans with Stripe integration, role-based access control (RBAC), public API with rate limiting, usage metering, and a comprehensive admin dashboard. You'll learn critical SaaS concepts including multi-tenant architecture patterns, billing automation, webhook handling, and API versioning.

This project simulates real-world SaaS applications like Slack, Notion, or Asana, where multiple organizations (tenants) use the same application with complete data isolation, different subscription levels, and team collaboration features.

---

## Core Features

### Must Have
- User registration and authentication (email/password, OAuth)
- Multi-tenant workspace/organization system
- Subscription plans (Free, Pro, Enterprise)
- Stripe integration for recurring payments
- Role-based access control (Owner, Admin, Member, Guest)
- Team member invitations and management
- Usage metering and quota enforcement
- Public REST API with API keys
- Rate limiting per subscription tier
- Admin dashboard for tenant management
- Billing dashboard with invoices
- Account settings and preferences

### Nice to Have
- Single Sign-On (SSO) with SAML 2.0
- Custom domain support per tenant
- White-labeling options
- Audit logs for compliance
- Two-factor authentication (2FA)
- Webhooks for external integrations
- Real-time notifications
- Advanced analytics and reporting
- Resource usage dashboards
- Automated backup and recovery
- Multi-region deployment
- GraphQL API alongside REST
- Terraform/CloudFormation for infrastructure

---

## Learning Objectives

By building this project, you will learn:

- **Multi-Tenant Architecture**: Schema-per-tenant vs shared-schema strategies
- **Subscription Management**: Handling recurring payments, upgrades/downgrades, proration
- **Access Control**: Implementing RBAC with hierarchical permissions
- **API Design**: Versioning, rate limiting, authentication, documentation
- **Payment Processing**: Stripe subscriptions, webhooks, invoice generation
- **Tenant Isolation**: Database design, security, and data segregation
- **Usage Metering**: Tracking and enforcing quota limits
- **Webhook Systems**: Reliable event delivery and retry logic
- **Scalability**: Horizontal scaling, caching, queue systems
- **Infrastructure**: Deployment, monitoring, logging, alerting

---

## Implementation Guide

### Step 1: Design Multi-Tenant Architecture

Choose your tenant isolation strategy:

**Option A: Shared Database, Shared Schema (Recommended for startups)**
```sql
-- All tenants share same tables with tenant_id column
CREATE TABLE organizations (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    subdomain VARCHAR(63) UNIQUE NOT NULL,
    plan_id VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE users (
    id UUID PRIMARY KEY,
    organization_id UUID NOT NULL REFERENCES organizations(id),
    email VARCHAR(255) UNIQUE NOT NULL,
    role VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE projects (
    id UUID PRIMARY KEY,
    organization_id UUID NOT NULL REFERENCES organizations(id),
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Row Level Security (PostgreSQL)
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON projects
    FOR ALL
    USING (organization_id = current_setting('app.current_tenant')::UUID);
```

**Option B: Shared Database, Separate Schemas**
```sql
-- Create schema per tenant
CREATE SCHEMA tenant_abc123;
CREATE SCHEMA tenant_xyz789;

-- Each schema has same table structure
CREATE TABLE tenant_abc123.projects (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
```

**Option C: Database per Tenant (Highest isolation)**
- Dedicated database for each customer
- Best for enterprise customers with compliance requirements

### Step 2: Implement Authentication System

```javascript
// auth.service.js
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import { db } from './database.js';

class AuthService {
    async register(email, password, organizationName) {
        // Create organization (tenant)
        const organization = await db.organizations.create({
            name: organizationName,
            subdomain: this.generateSubdomain(organizationName),
            plan_id: 'free',
            trial_ends_at: new Date(Date.now() + 14 * 24 * 60 * 60 * 1000) // 14 days
        });

        // Create owner user
        const hashedPassword = await bcrypt.hash(password, 10);
        const user = await db.users.create({
            organization_id: organization.id,
            email,
            password: hashedPassword,
            role: 'owner'
        });

        // Generate JWT with tenant context
        const token = jwt.sign(
            {
                userId: user.id,
                organizationId: organization.id,
                role: user.role
            },
            process.env.JWT_SECRET,
            { expiresIn: '7d' }
        );

        return { token, user, organization };
    }

    async login(email, password) {
        const user = await db.users.findOne({ where: { email } });
        if (!user) throw new Error('Invalid credentials');

        const valid = await bcrypt.compare(password, user.password);
        if (!valid) throw new Error('Invalid credentials');

        // Check if organization has active subscription
        const organization = await db.organizations.findByPk(user.organization_id);
        if (organization.status === 'suspended') {
            throw new Error('Account suspended - please update payment method');
        }

        const token = jwt.sign(
            {
                userId: user.id,
                organizationId: organization.id,
                role: user.role
            },
            process.env.JWT_SECRET,
            { expiresIn: '7d' }
        );

        return { token, user, organization };
    }

    generateSubdomain(name) {
        return name.toLowerCase()
            .replace(/[^a-z0-9]/g, '-')
            .replace(/-+/g, '-')
            .substring(0, 63);
    }
}

export default new AuthService();
```

### Step 3: Implement Subscription Plans

```javascript
// subscription-plans.js
export const PLANS = {
    free: {
        id: 'free',
        name: 'Free',
        price: 0,
        interval: 'month',
        features: {
            maxUsers: 3,
            maxProjects: 5,
            apiCallsPerMonth: 1000,
            storage: '1GB',
            support: 'Community'
        }
    },
    pro: {
        id: 'pro',
        name: 'Professional',
        price: 29,
        interval: 'month',
        stripePriceId: 'price_xxx',
        features: {
            maxUsers: 10,
            maxProjects: 50,
            apiCallsPerMonth: 50000,
            storage: '50GB',
            support: 'Email',
            customDomain: true
        }
    },
    enterprise: {
        id: 'enterprise',
        name: 'Enterprise',
        price: 99,
        interval: 'month',
        stripePriceId: 'price_yyy',
        features: {
            maxUsers: -1, // unlimited
            maxProjects: -1,
            apiCallsPerMonth: 1000000,
            storage: '1TB',
            support: '24/7 Priority',
            customDomain: true,
            sso: true,
            whiteLabel: true,
            sla: '99.9%'
        }
    }
};

// subscription.service.js
import Stripe from 'stripe';
import { db } from './database.js';
import { PLANS } from './subscription-plans.js';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

class SubscriptionService {
    async createCheckoutSession(organizationId, planId) {
        const organization = await db.organizations.findByPk(organizationId);
        const plan = PLANS[planId];

        const session = await stripe.checkout.sessions.create({
            customer_email: organization.owner_email,
            client_reference_id: organizationId,
            payment_method_types: ['card'],
            line_items: [{
                price: plan.stripePriceId,
                quantity: 1
            }],
            mode: 'subscription',
            success_url: `${process.env.APP_URL}/billing/success?session_id={CHECKOUT_SESSION_ID}`,
            cancel_url: `${process.env.APP_URL}/billing/canceled`
        });

        return session;
    }

    async handleWebhook(event) {
        switch (event.type) {
            case 'checkout.session.completed':
                await this.handleCheckoutCompleted(event.data.object);
                break;
            
            case 'customer.subscription.created':
            case 'customer.subscription.updated':
                await this.handleSubscriptionUpdated(event.data.object);
                break;
            
            case 'customer.subscription.deleted':
                await this.handleSubscriptionCanceled(event.data.object);
                break;
            
            case 'invoice.payment_failed':
                await this.handlePaymentFailed(event.data.object);
                break;
        }
    }

    async handleCheckoutCompleted(session) {
        const organizationId = session.client_reference_id;
        const subscription = await stripe.subscriptions.retrieve(session.subscription);

        await db.organizations.update(
            {
                stripe_customer_id: session.customer,
                stripe_subscription_id: subscription.id,
                plan_id: this.getPlanIdFromStripe(subscription),
                status: 'active'
            },
            { where: { id: organizationId } }
        );
    }

    async handleSubscriptionUpdated(subscription) {
        await db.organizations.update(
            {
                plan_id: this.getPlanIdFromStripe(subscription),
                status: subscription.status
            },
            { where: { stripe_subscription_id: subscription.id } }
        );
    }

    async handleSubscriptionCanceled(subscription) {
        await db.organizations.update(
            {
                plan_id: 'free',
                status: 'canceled',
                canceled_at: new Date()
            },
            { where: { stripe_subscription_id: subscription.id } }
        );
    }

    async handlePaymentFailed(invoice) {
        const organization = await db.organizations.findOne({
            where: { stripe_customer_id: invoice.customer }
        });

        // Send email notification
        await this.sendPaymentFailedEmail(organization);

        // Suspend after 3 failed attempts
        if (invoice.attempt_count >= 3) {
            await db.organizations.update(
                { status: 'suspended' },
                { where: { id: organization.id } }
            );
        }
    }

    getPlanIdFromStripe(subscription) {
        const priceId = subscription.items.data[0].price.id;
        return Object.values(PLANS).find(p => p.stripePriceId === priceId)?.id || 'free';
    }
}

export default new SubscriptionService();
```

### Step 4: Implement RBAC (Role-Based Access Control)

```javascript
// permissions.js
export const ROLES = {
    owner: {
        level: 4,
        permissions: ['*'] // All permissions
    },
    admin: {
        level: 3,
        permissions: [
            'user:read', 'user:create', 'user:update', 'user:delete',
            'project:*',
            'billing:read',
            'settings:*'
        ]
    },
    member: {
        level: 2,
        permissions: [
            'user:read',
            'project:read', 'project:create', 'project:update',
            'settings:read'
        ]
    },
    guest: {
        level: 1,
        permissions: [
            'user:read',
            'project:read'
        ]
    }
};

// rbac.middleware.js
export function requirePermission(permission) {
    return async (req, res, next) => {
        const user = req.user; // Set by auth middleware
        const userRole = ROLES[user.role];

        // Owner has all permissions
        if (userRole.permissions.includes('*')) {
            return next();
        }

        // Check specific permission
        const hasPermission = userRole.permissions.some(p => {
            if (p === permission) return true;
            // Check wildcard permissions (e.g., 'project:*')
            if (p.endsWith(':*')) {
                const prefix = p.split(':')[0];
                return permission.startsWith(prefix + ':');
            }
            return false;
        });

        if (!hasPermission) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }

        next();
    };
}

// Usage in routes
app.delete('/api/users/:id', 
    authenticate,
    requirePermission('user:delete'),
    userController.delete
);
```

### Step 5: Implement Usage Metering

```javascript
// usage-meter.service.js
import { db } from './database.js';
import { PLANS } from './subscription-plans.js';

class UsageMeterService {
    async trackApiCall(organizationId) {
        const today = new Date().toISOString().split('T')[0];

        await db.usageMetrics.upsert({
            organization_id: organizationId,
            date: today,
            api_calls: db.sequelize.literal('api_calls + 1')
        });

        // Check if quota exceeded
        const usage = await db.usageMetrics.findOne({
            where: { organization_id: organizationId, date: today }
        });

        const organization = await db.organizations.findByPk(organizationId);
        const plan = PLANS[organization.plan_id];

        if (usage.api_calls > plan.features.apiCallsPerMonth / 30) {
            throw new Error('Daily API quota exceeded');
        }
    }

    async checkQuota(organizationId, resource) {
        const organization = await db.organizations.findByPk(organizationId);
        const plan = PLANS[organization.plan_id];

        switch (resource) {
            case 'users':
                const userCount = await db.users.count({
                    where: { organization_id: organizationId }
                });
                if (plan.features.maxUsers !== -1 && userCount >= plan.features.maxUsers) {
                    throw new Error(`User limit reached (${plan.features.maxUsers})`);
                }
                break;

            case 'projects':
                const projectCount = await db.projects.count({
                    where: { organization_id: organizationId }
                });
                if (plan.features.maxProjects !== -1 && projectCount >= plan.features.maxProjects) {
                    throw new Error(`Project limit reached (${plan.features.maxProjects})`);
                }
                break;
        }
    }
}

export default new UsageMeterService();
```

### Step 6: Create Public API with Rate Limiting

```javascript
// api-key.service.js
import crypto from 'crypto';
import { db } from './database.js';

class ApiKeyService {
    async generateKey(organizationId, name) {
        const key = 'sk_' + crypto.randomBytes(32).toString('hex');
        
        await db.apiKeys.create({
            organization_id: organizationId,
            name,
            key: await this.hashKey(key),
            last_used: null
        });

        // Return unhashed key only once
        return key;
    }

    async hashKey(key) {
        return crypto.createHash('sha256').update(key).digest('hex');
    }

    async validateKey(key) {
        const hashed = await this.hashKey(key);
        const apiKey = await db.apiKeys.findOne({
            where: { key: hashed },
            include: [{ model: db.organizations }]
        });

        if (!apiKey) throw new Error('Invalid API key');

        // Update last_used
        await apiKey.update({ last_used: new Date() });

        return apiKey.organization;
    }
}

export default new ApiKeyService();

// rate-limiter.middleware.js
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import redis from './redis.js';
import { PLANS } from './subscription-plans.js';

export function createRateLimiter() {
    return rateLimit({
        store: new RedisStore({ client: redis }),
        windowMs: 15 * 60 * 1000, // 15 minutes
        max: async (req) => {
            const organization = req.organization;
            const plan = PLANS[organization.plan_id];
            
            // Requests per 15 minutes = daily limit / 96 (15-min intervals per day)
            return Math.floor(plan.features.apiCallsPerMonth / 30 / 96);
        },
        keyGenerator: (req) => req.organization.id,
        handler: (req, res) => {
            res.status(429).json({
                error: 'Too many requests',
                message: 'Upgrade your plan for higher rate limits'
            });
        }
    });
}
```

### Step 7: Implement Team Invitations

```javascript
// invitation.service.js
import crypto from 'crypto';
import { db } from './database.js';
import { sendEmail } from './email.service.js';

class InvitationService {
    async inviteUser(organizationId, email, role, invitedBy) {
        // Check user quota
        await usageMeterService.checkQuota(organizationId, 'users');

        const token = crypto.randomBytes(32).toString('hex');
        const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 7 days

        await db.invitations.create({
            organization_id: organizationId,
            email,
            role,
            token,
            invited_by: invitedBy,
            expires_at: expiresAt
        });

        // Send invitation email
        await sendEmail({
            to: email,
            subject: 'You have been invited to join',
            html: `
                <p>You have been invited to join an organization.</p>
                <a href="${process.env.APP_URL}/accept-invitation/${token}">
                    Accept Invitation
                </a>
            `
        });

        return { success: true };
    }

    async acceptInvitation(token, userId) {
        const invitation = await db.invitations.findOne({
            where: { token, accepted_at: null }
        });

        if (!invitation) throw new Error('Invalid or expired invitation');
        if (new Date() > invitation.expires_at) {
            throw new Error('Invitation expired');
        }

        // Add user to organization
        await db.users.update(
            {
                organization_id: invitation.organization_id,
                role: invitation.role
            },
            { where: { id: userId } }
        );

        // Mark invitation as accepted
        await invitation.update({ accepted_at: new Date() });

        return { success: true };
    }
}

export default new InvitationService();
```

---

## Technology Stack

### Backend
- **Node.js + Express** or **Python + FastAPI** or **Ruby on Rails**
- **PostgreSQL**: Multi-tenant database with Row Level Security
- **Redis**: Caching, rate limiting, session storage
- **Bull/Bee-Queue**: Background job processing

### Payment & Billing
- **Stripe**: Subscription management, webhooks, invoices
- **Stripe Customer Portal**: Self-service billing management

### Authentication
- **Passport.js / Auth0**: OAuth, SAML, JWT
- **bcrypt**: Password hashing
- **jsonwebtoken**: JWT generation

### API & Rate Limiting
- **express-rate-limit**: Request throttling
- **Swagger/OpenAPI**: API documentation
- **API Gateway**: Kong or AWS API Gateway

### Infrastructure
- **Docker**: Containerization
- **Kubernetes**: Orchestration
- **AWS/GCP/Azure**: Cloud hosting
- **CloudFront/Cloudflare**: CDN and DDoS protection
- **Terraform**: Infrastructure as Code

### Monitoring & Logging
- **Sentry**: Error tracking
- **DataDog/New Relic**: Application performance
- **ELK Stack**: Log aggregation
- **Prometheus + Grafana**: Metrics and alerting

---

## Testing Checklist

- [ ] User registration creates organization and owner
- [ ] Subscription checkout and webhooks work correctly
- [ ] Plan upgrades/downgrades handle proration
- [ ] Tenant isolation prevents cross-organization data access
- [ ] RBAC correctly enforces permissions
- [ ] API rate limiting varies by plan
- [ ] Usage quotas enforced (users, projects, API calls)
- [ ] Invitation flow works end-to-end
- [ ] Payment failure suspends account appropriately
- [ ] API keys authenticate and track usage
- [ ] Webhooks retry on failure
- [ ] Database queries use tenant context

---

## Additional Challenges

Once you complete the basic version, try these extensions:

### Easy
- Email notifications for important events
- Activity logs for audit trail
- Team member profile pages
- Billing history and invoice downloads

### Medium
- Custom domain support with SSL certificates
- Advanced analytics dashboard with charts
- Webhook system for external integrations
- Two-factor authentication (2FA)
- CSV/Excel export functionality
- White-labeling options (custom branding)

### Hard
- Single Sign-On (SSO) with SAML 2.0
- Multi-region deployment with data residency
- Advanced security: encryption at rest, audit logs
- Machine learning usage predictions
- Automated scalability (auto-scaling, load balancing)
- Disaster recovery and backup automation
- Compliance certifications (SOC 2, GDPR, HIPAA)
- GraphQL API alongside REST
- Real-time collaboration features

---

## Common Pitfalls

**Tenant data leakage**: Always filter queries by organization_id  
**Hardcoded subscription limits**: Store limits in database for flexibility  
**Webhook replay attacks**: Verify Stripe webhook signatures  
**Poor scalability**: Use connection pooling, caching, and async processing  
**Insufficient error handling**: Log errors and notify admins of critical issues  
**No backup strategy**: Implement automated backups and test restoration  

---

## Resources

### Documentation
- [Stripe Subscriptions Guide](https://stripe.com/docs/billing/subscriptions/overview)
- [Multi-Tenant Architecture Patterns](https://docs.microsoft.com/en-us/azure/architecture/patterns/)
- [PostgreSQL Row Level Security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [OAuth 2.0 Specification](https://oauth.net/2/)

### Tutorials
- [Building a SaaS App from Scratch](https://www.indiehackers.com/start)
- [Stripe Webhook Best Practices](https://stripe.com/docs/webhooks/best-practices)
- [Implementing RBAC](https://auth0.com/docs/manage-users/access-control/rbac)

### Books
- "Building Multi-Tenant Applications" by Microsoft Patterns & Practices
- "The SaaS Playbook" by Rob Walling
- "Implementing Stripe" by Nicholas Krayacich

---

## Deployment Guide

### Docker Compose (Development)

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/saas_db
      - REDIS_URL=redis://redis:6379
      - STRIPE_SECRET_KEY=${STRIPE_SECRET_KEY}
    depends_on:
      - db
      - redis
  
  db:
    image: postgres:14
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: saas_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Production (AWS)

1. **Compute**: ECS Fargate or EKS
2. **Database**: RDS PostgreSQL with Multi-AZ
3. **Cache**: ElastiCache Redis cluster
4. **Storage**: S3 for files
5. **CDN**: CloudFront
6. **Monitoring**: CloudWatch + Sentry
7. **Load Balancer**: Application Load Balancer

---

## Submission Checklist

Before considering your project complete:

- [ ] Multi-tenant architecture implemented with isolation
- [ ] Stripe subscription system working end-to-end
- [ ] Role-based access control enforcing permissions
- [ ] Public API with authentication and rate limiting
- [ ] Usage metering and quota enforcement
- [ ] Team invitation and management system
- [ ] Admin dashboard with organization management
- [ ] Billing dashboard with invoice access
- [ ] Webhook handling with retry logic
- [ ] Comprehensive error handling and logging
- [ ] Unit and integration tests written
- [ ] API documentation (Swagger/OpenAPI)
- [ ] Deployment configuration (Docker/K8s)
- [ ] Security best practices implemented
- [ ] Performance optimizations (caching, indexing)

---

## Portfolio Tips

When showcasing this project:

1. **Architecture Diagram**: Show multi-tenant data flow and isolation
2. **Demo Video**: Walkthrough of signup, subscription, and team features
3. **Metrics**: Display response times, scalability tests, uptime
4. **Code Quality**: Highlight clean architecture, testing, documentation
5. **Security**: Explain RBAC, tenant isolation, and payment security
6. **Scalability**: Discuss how the system handles thousands of tenants

---

## License

This project idea is part of the [Coding Projects Library](https://github.com/Ryanditko/biblioteca-de-projetos) and is available under the MIT License.
