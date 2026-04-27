# Technology Stack Documentation
# RentKart — India's Premier Rental Marketplace
**Version:** 1.0 | **Architecture:** Monolithic Modular → Microservices-ready

---

## 1. Stack Overview

- **Architecture Pattern:** Modular Monolith (MVP) — structured for microservices extraction post-PMF
- **Deployment Strategy:** Containerised (Docker) deployed on AWS, served via CloudFront CDN
- **Justification:** Monolith allows fast iteration and lower operational overhead at MVP scale; module boundaries are well-defined so extraction to microservices (payments, notifications, disputes) is straightforward post-traction
- **Mobile Priority:** React Native for Buyer App and Seller App; lightweight PWA for Delivery Agent App

---

## 2. Frontend Stack

### Buyer & Seller Apps (Mobile)

| Technology | Version | Documentation | Reason | Alternatives Rejected |
|---|---|---|---|---|
| React Native | 0.73.6 | https://reactnative.dev/docs | Single codebase for iOS + Android; vast ecosystem; strong India talent pool | Flutter — Dart is less common; native — too slow for MVP |
| TypeScript | 5.3.3 | https://www.typescriptlang.org/docs | Type safety reduces runtime bugs; better IDE support | Plain JavaScript — unsafe for payment-critical app |
| Expo (Managed Workflow) | 50.0.14 | https://docs.expo.dev | Faster development; OTA updates; handles native build complexity | Bare React Native — too much native config overhead at MVP stage |
| React Navigation | 6.1.17 | https://reactnavigation.org/docs | Industry standard for RN navigation; supports deep links natively | React Native Navigation — more complex setup, overkill at MVP |
| Zustand | 4.5.2 | https://docs.pmnd.rs/zustand | Lightweight, boilerplate-free state management; easy to use | Redux Toolkit — verbose; Context — re-render issues at scale |
| React Hook Form | 7.51.0 | https://react-hook-form.com | Minimal re-renders; native validation; works well with TypeScript | Formik — heavier bundle; slower |
| Axios | 1.6.8 | https://axios-http.com/docs | Interceptors for auth token injection; consistent with web; request cancellation | Fetch — no interceptors natively |
| React Native Image Picker (in-app capture) | 7.1.0 | https://github.com/react-native-image-picker | Required for in-app photo capture (no gallery upload for inspection) | Direct Camera API — less abstraction, more platform-specific code |
| NativeWind | 4.0.1 | https://www.nativewind.dev | Tailwind CSS classes in React Native; consistent styling with web admin | StyleSheet — no design system cohesion across platforms |

### Admin Panel (Web)

| Technology | Version | Documentation | Reason | Alternatives Rejected |
|---|---|---|---|---|
| Next.js | 14.1.4 | https://nextjs.org/docs | SSR for fast admin load; API routes; file-based routing | Create React App — no SSR; Remix — less ecosystem maturity |
| TypeScript | 5.3.3 | https://www.typescriptlang.org/docs | Same as above | — |
| Tailwind CSS | 3.4.3 | https://tailwindcss.com/docs | Utility-first; fast to build dense admin UIs | CSS Modules — more files; MUI — harder to customise |
| shadcn/ui | 0.8.0 | https://ui.shadcn.com | Accessible, unstyled components built on Radix UI; easy to theme | Chakra UI — too much default styling; MUI — CSS-in-JS overhead |
| Zustand | 4.5.2 | https://docs.pmnd.rs/zustand | Same reasoning as mobile | — |
| React Hook Form | 7.51.0 | https://react-hook-form.com | Same reasoning as mobile | — |
| Axios | 1.6.8 | https://axios-http.com/docs | Same reasoning as mobile | — |
| Recharts | 2.12.2 | https://recharts.org | Declarative chart library for GMV/analytics dashboards | Chart.js — more verbose config; D3 — too low level |
| TanStack Query (React Query) | 5.28.6 | https://tanstack.com/query/v5 | Server state management for admin data; caching, background refetch | SWR — less feature-rich |

---

## 3. Backend Stack

| Technology | Version | Documentation | Reason | Alternatives Rejected |
|---|---|---|---|---|
| Node.js | 20.11.1 (LTS) | https://nodejs.org/en/docs | JavaScript throughout the stack; large India hiring pool; proven for marketplace backends | Python/Django — slower for real-time; Java — heavyweight for MVP |
| Fastify | 4.26.2 | https://fastify.dev/docs | 2× faster than Express; built-in schema validation; TypeScript support | Express — slower; NestJS — over-engineered for MVP |
| TypeScript | 5.3.3 | https://www.typescriptlang.org/docs | Shared types with frontend; safer API contract | — |
| PostgreSQL | 16.2 | https://www.postgresql.org/docs | ACID compliance critical for escrow/payment transactions; JSONB for flexible meta | MySQL — weaker JSONB; MongoDB — no ACID for financial data |
| Prisma ORM | 5.11.0 | https://www.prisma.io/docs | Type-safe queries; auto-generated types; migration system | Drizzle — less mature at time of decision; Raw SQL — unsafe |
| Redis | 7.2.4 | https://redis.io/docs | Session cache, OTP store (60s TTL), rate limiting, order state pub/sub | Memcached — no pub/sub; in-memory — no persistence |
| Razorpay (Payment Gateway) | SDK 2.9.2 | https://razorpay.com/docs | PA-licensed in India; supports UPI/cards/wallets/netbanking; escrow-safe; widely used | PayU — weaker API docs; Paytm — trust issues; Stripe — no UPI |
| Firebase Cloud Messaging (FCM) | Admin SDK 12.0.0 | https://firebase.google.com/docs/cloud-messaging | Push notifications for Android + iOS; reliable delivery in India | OneSignal — third-party dependency; direct APNs — split implementation |
| Twilio / MSG91 (SMS) | MSG91 SDK 3.x | https://docs.msg91.com | India-specific SMS provider; DLT-registered sender IDs (required by TRAI) | Twilio — higher cost for India; AWS SNS — higher latency |
| AWS S3 | SDK 3.x | https://docs.aws.amazon.com/s3 | Object storage for listing photos, inspection photos, KYC documents; low cost at scale | Cloudinary — more expensive; Supabase Storage — less mature |
| AWS CloudFront | — | https://docs.aws.amazon.com/cloudfront | CDN for fast image delivery across India | — |
| Nodemailer + AWS SES | Nodemailer 6.9.13 | https://nodemailer.com/about | Transactional emails (confirmations, refunds, alerts); SES is cheapest at scale | SendGrid — expensive; Resend — less battle-tested for volume |
| ClearTax GST API | REST v2 | https://developer.cleartax.in | GST computation on rental fees; automated invoice generation | Manual GST — non-compliant; Zoho — harder API integration |

---

## 4. DevOps & Infrastructure

### Hosting
- **Backend API:** AWS EC2 (t3.medium) → auto-scaling group from Month 3
- **Database:** AWS RDS PostgreSQL 16.2 (Multi-AZ from Month 3)
- **Redis:** AWS ElastiCache (cache.t3.micro at MVP)
- **File Storage:** AWS S3 (ap-south-1 — Mumbai region for DPDP compliance)
- **Admin Panel / Web:** Vercel (Next.js)
- **Mobile Apps:** Expo EAS Build for distribution; Apple App Store + Google Play Store

### Git Strategy
- **Branching:** Git Flow — `main` (production), `develop` (integration), `feature/*`, `hotfix/*`
- **Commit Convention:** Conventional Commits (feat, fix, chore, docs, refactor)
- **PR Rules:** Minimum 1 reviewer approval + passing CI before merge to develop; 2 reviewers for main

### CI/CD
- **Platform:** GitHub Actions
- **Pipeline:** Push → lint → type-check → unit tests → build → deploy (staging on `develop`, prod on `main` tag)
- **Mobile:** EAS Build triggered on `develop` push for internal testing; store submission from `main` tag

### Monitoring & Error Tracking
- **Error Tracking:** Sentry (Node.js + React Native) — https://sentry.io
- **Uptime Monitoring:** UptimeRobot (5-minute checks on API health endpoints)
- **Logging:** Winston (structured JSON logs) → AWS CloudWatch
- **Performance:** AWS CloudWatch metrics; custom dashboard for API latency and error rates

---

## 5. Development Tools

- **Linter:** ESLint 8.57.0 with config: `@typescript-eslint`, `eslint-plugin-react`, `eslint-plugin-react-native`
- **Formatter:** Prettier 3.2.5 — single quotes, 100 char line width, trailing commas (ES5)
- **Git Hooks:** Husky 9.0.11 + lint-staged — run ESLint + Prettier on staged files pre-commit
- **API Testing:** Bruno (open-source Postman alternative) — collection stored in `/api-collections`
- **DB GUI:** Prisma Studio (dev) + TablePlus (production-safe read-only access)

---

## 6. Environment Variables

```env
# --- DATABASE ---
DATABASE_URL="postgresql://rentkart_user:PASSWORD@HOST:5432/rentkart_prod"
DATABASE_REPLICA_URL="postgresql://rentkart_user:PASSWORD@REPLICA_HOST:5432/rentkart_prod"

# --- REDIS ---
REDIS_URL="redis://HOST:6379"
REDIS_PASSWORD="PASSWORD"

# --- AUTH ---
JWT_ACCESS_SECRET="64-char-random-hex"
JWT_REFRESH_SECRET="64-char-random-hex"
JWT_ACCESS_EXPIRY="15m"
JWT_REFRESH_EXPIRY="30d"
BCRYPT_ROUNDS="12"

# --- PAYMENT ---
RAZORPAY_KEY_ID="rzp_live_XXXX"
RAZORPAY_KEY_SECRET="XXXX"
RAZORPAY_WEBHOOK_SECRET="XXXX"

# --- FILE STORAGE ---
AWS_ACCESS_KEY_ID="XXXX"
AWS_SECRET_ACCESS_KEY="XXXX"
AWS_REGION="ap-south-1"
AWS_S3_BUCKET_NAME="rentkart-media-prod"
AWS_CLOUDFRONT_URL="https://cdn.rentkart.in"

# --- NOTIFICATIONS ---
FCM_SERVICE_ACCOUNT_JSON='{"type":"service_account",...}'
MSG91_AUTH_KEY="XXXX"
MSG91_SENDER_ID="RENTKART"
MSG91_TEMPLATE_ID_OTP="XXXX"

# --- EMAIL ---
AWS_SES_REGION="ap-south-1"
EMAIL_FROM="noreply@rentkart.in"

# --- GST / TAX ---
CLEARTAX_API_KEY="XXXX"
RENTKART_GSTIN="27XXXXX"

# --- KYCEKYC ---
DIGILOCKER_CLIENT_ID="XXXX"
DIGILOCKER_CLIENT_SECRET="XXXX"
DIGILOCKER_REDIRECT_URI="https://api.rentkart.in/auth/digilocker/callback"

# --- APP ---
NODE_ENV="production"
PORT="3000"
API_BASE_URL="https://api.rentkart.in"
CORS_ALLOWED_ORIGINS="https://admin.rentkart.in,https://rentkart.in"
PLATFORM_COMMISSION_RATE="0.15"
NON_RETURN_AUTO_SALE_DAYS="25"
```

---

## 7. Package.json Scripts

```json
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc -p tsconfig.build.json",
    "start": "node dist/server.js",
    "lint": "eslint 'src/**/*.{ts,tsx}'",
    "lint:fix": "eslint 'src/**/*.{ts,tsx}' --fix",
    "format": "prettier --write 'src/**/*.{ts,tsx,json}'",
    "type-check": "tsc --noEmit",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate deploy",
    "db:migrate:dev": "prisma migrate dev",
    "db:seed": "tsx prisma/seed.ts",
    "db:reset": "prisma migrate reset",
    "db:studio": "prisma studio",
    "build:docker": "docker build -t rentkart-api .",
    "logs": "aws logs tail /rentkart/api --follow"
  }
}
```

---

## 8. Dependencies Lock

### Backend (package.json — production)
```json
{
  "dependencies": {
    "@aws-sdk/client-s3": "3.540.0",
    "@aws-sdk/client-ses": "3.540.0",
    "@prisma/client": "5.11.0",
    "axios": "1.6.8",
    "bcrypt": "5.1.1",
    "fastify": "4.26.2",
    "@fastify/cors": "9.0.1",
    "@fastify/rate-limit": "9.1.0",
    "@fastify/jwt": "8.0.1",
    "@fastify/multipart": "8.2.0",
    "firebase-admin": "12.0.0",
    "ioredis": "5.3.2",
    "jsonwebtoken": "9.0.2",
    "nodemailer": "6.9.13",
    "razorpay": "2.9.2",
    "winston": "3.13.0",
    "zod": "3.22.4"
  },
  "devDependencies": {
    "@types/bcrypt": "5.0.2",
    "@types/node": "20.11.30",
    "@types/nodemailer": "6.4.14",
    "@typescript-eslint/eslint-plugin": "7.3.1",
    "@typescript-eslint/parser": "7.3.1",
    "eslint": "8.57.0",
    "husky": "9.0.11",
    "lint-staged": "15.2.2",
    "prettier": "3.2.5",
    "prisma": "5.11.0",
    "tsx": "4.7.1",
    "typescript": "5.3.3",
    "vitest": "1.4.0"
  }
}
```

### Admin Panel (package.json — Next.js web)
```json
{
  "dependencies": {
    "@tanstack/react-query": "5.28.6",
    "axios": "1.6.8",
    "class-variance-authority": "0.7.0",
    "clsx": "2.1.0",
    "lucide-react": "0.359.0",
    "next": "14.1.4",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "react-hook-form": "7.51.0",
    "recharts": "2.12.2",
    "tailwind-merge": "2.2.2",
    "zustand": "4.5.2",
    "zod": "3.22.4",
    "@hookform/resolvers": "3.3.4"
  },
  "devDependencies": {
    "@types/react": "18.2.73",
    "@types/react-dom": "18.2.23",
    "@types/node": "20.11.30",
    "autoprefixer": "10.4.19",
    "eslint": "8.57.0",
    "eslint-config-next": "14.1.4",
    "postcss": "8.4.38",
    "prettier": "3.2.5",
    "tailwindcss": "3.4.3",
    "typescript": "5.3.3"
  }
}
```

---

## 9. Security Considerations

### Authentication
- **Password hashing:** bcrypt with `BCRYPT_ROUNDS=12`
- **Access token expiry:** 15 minutes (JWT)
- **Refresh token expiry:** 30 days (stored in Redis; revocable)
- **OTP TTL:** 60 seconds in Redis; max 5 attempts before 30-minute lockout per phone number
- **Session invalidation:** All refresh tokens invalidated on password change or account suspension

### CORS Configuration
```typescript
// Allowed origins
const allowedOrigins = process.env.CORS_ALLOWED_ORIGINS.split(',');
fastify.register(cors, {
  origin: (origin, cb) => {
    if (!origin || allowedOrigins.includes(origin)) cb(null, true);
    else cb(new Error('Not allowed by CORS'), false);
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
});
```

### Rate Limiting
- Login: 5 attempts / 15 min / IP
- Registration: 10 / hour / IP
- OTP send: 3 / 10 min / phone number
- API (public, unauthenticated): 100 requests / min / IP
- API (authenticated): 500 requests / min / user

### Data Protection
- Aadhaar/PAN stored encrypted at rest (AES-256-GCM via AWS KMS)
- All API traffic over TLS 1.3
- KYC documents stored in private S3 bucket with pre-signed URL access (5-minute expiry)
- Escrow funds in segregated Razorpay subaccount — never transferred to operating account

---

## 10. Version Upgrade Policy

- **Major versions (X.0.0):** Evaluate quarterly; require full test suite pass + 2-week staging soak
- **Minor versions (x.Y.0):** Evaluate monthly; require unit + integration tests pass
- **Patch/security (x.y.Z):** Apply within 48 hours of CVE disclosure; hotfix branch directly to main
- **Node.js LTS:** Upgrade to new LTS within 90 days of release (follow Node.js release schedule)
- **Prisma:** Pin exact version; upgrade with each DB migration cycle
- **Before any major upgrade:** Run full Vitest suite + manual smoke test of checkout, refund, and dispute flows
