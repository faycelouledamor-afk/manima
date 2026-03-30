# EuroTuni — Marketplace Europe ↔ Tunisie

## Architecture

- **Frontend**: React + TypeScript + Vite, TailwindCSS, shadcn/ui, framer-motion, wouter
- **Backend**: Express.js + TypeScript, Drizzle ORM, PostgreSQL
- **Auth**: Replit OpenID Connect (Passport.js)
- **Port**: 5000 (Express serves both API and Vite dev server)

## Auth Flow (Mode Réel)

1. User visits `/` → Onboarding page checks `/api/me`
2. If 401 → shows login button → `/api/login` → Replit OIDC
3. After login → `/api/callback` → creates/links app user in `users` table via `replitUserId`
4. Redirected to `/products`
5. All API mutations use `isAuthenticated + getAppUser` middleware to get the real `appUser.id`

## Key Files

- `shared/schema.ts` — DB schema (users, products, sales, transporters, messages, bookings)
- `shared/models/auth.ts` — Auth DB tables (sessions, auth_accounts)
- `server/routes.ts` — All API routes (all protected routes use isAuthenticated + getAppUser)
- `server/storage.ts` — DB operations (findOrCreateUserByReplitId, updateUserRole, etc.)
- `server/replit_integrations/auth/replitAuth.ts` — Replit OIDC auth + getAppUser middleware
- `client/src/hooks/use-auth.ts` — useAuth(), useUpdateProfile(), useUpdateRole() hooks
- `client/src/App.tsx` — Router with AuthGuard on all protected routes
- `client/src/pages/Onboarding.tsx` — Login gate (redirects to /products if authenticated)

## API Routes

### Public
- `GET /api/products` — List all products
- `GET /api/transporters` — List all transporters
- `GET /api/messages` — List messages
- `GET /api/users/:id` — Get user by ID
- `GET /api/login` — Replit OIDC login
- `GET /api/callback` — OIDC callback (creates/links app user)
- `GET /api/logout` — Logout

### Authenticated (requires Replit login)
- `GET /api/me` — Current user's app profile
- `PATCH /api/me` — Update current user's profile (name, location, countryCode, avatarUrl)
- `PATCH /api/me/role` — Update current user's role (Acheteur/Vendeur/Transporteur)
- `GET /api/orders` — Current user's orders
- `POST /api/orders` — Create order (buyerId = current user)
- `POST /api/bookings` — Create booking (buyerId = current user)
- `GET /api/seller/dashboard` — Seller dashboard for current user
- `GET /api/transporter/dashboard` — Transporter dashboard for current user
- `POST /api/products` — Add product (sellerId = current user)
- `POST /api/transporters` — Add transport trip (userId = current user)
- `DELETE /api/products/:id` — Delete product

## Database Schema

```sql
users (id serial, replit_user_id text UNIQUE, name, role, location, country_code, balance, orders_count, rating, avatar_url)
products (id serial, title, price_eur, weight_kg, category, image_url, seller_id, seller_name, seller_country, rating)
sales (id serial, product_id, seller_id, buyer_id, amount_eur, status, created_at)
transporters (id serial, user_id, name, rating, trips_count, price_per_kg_eur, route_from, route_to, ...)
messages (id serial, sender_name, content, timestamp, unread_count, avatar_url)
bookings (id serial, buyer_id, transporter_id, weight_kg, total_price_eur, status, created_at)
sessions (from connect-pg-simple)
auth_accounts (replit_user_id, email, first_name, last_name, profile_image_url)
```

## User Roles

Roles are stored in the `users` table (`role` column): `Acheteur`, `Vendeur`, `Transporteur`
- Changed via `PATCH /api/me/role`
- Controls which dashboard shortcuts appear in the Profile page

## Bilingual Support

- Languages: French (`fr`) and Arabic (`ar`)
- RTL: auto-applied when Arabic is selected
- Stored in `localStorage` as `"fr"` or `"ar"`
- Translations in `client/src/lib/i18n.ts`
- Context: `client/src/contexts/LanguageContext.tsx`
