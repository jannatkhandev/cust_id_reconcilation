# Bitespeed Identity Reconciliation

A web service that identifies and consolidates customer contact information across multiple purchases on FluxKart.com.

When a customer checks out using different emails or phone numbers over time, this service links those contacts together under a single primary identity — so FluxKart always knows who it's talking to.

**Live endpoint:** `https://customer-identity-reconcilation.onrender.com/identify`

---

## Stack

- **Runtime:** Node.js + TypeScript
- **Framework:** Express
- **ORM:** Prisma
- **Database:** PostgreSQL (Supabase)
- **Hosting:** Render

---

## Local Setup

**1. Install dependencies**
```bash
pnpm install
```

**2. Configure environment**
```bash
cp .env.example .env
```

Fill in your Supabase connection strings in `.env`:
- `DATABASE_URL` → Session pooler (port 5432)
- `DIRECT_URL` → Session pooler (port 5432)

Both are available under Supabase → Settings → Database → Connection string → Session pooler.

**3. Run migrations**
```bash
pnpm migrate
```

**4. Start dev server**
```bash
pnpm dev
```

---

## API

### `POST /identify`

Accepts an email, phone number, or both. Returns the consolidated contact for that customer.

**Request**
```json
{
  "email": "mcfly@hillvalley.edu",
  "phoneNumber": "123456"
}
```

At least one field is required. Both are optional individually.

**Response**
```json
{
  "contact": {
    "primaryContatctId": 1,
    "emails": ["lorraine@hillvalley.edu", "mcfly@hillvalley.edu"],
    "phoneNumbers": ["123456"],
    "secondaryContactIds": [23]
  }
}
```

- Primary contact's email and phone always appear first in their arrays.
- A new secondary contact is created when the request contains information not yet linked to the cluster.
- If two previously separate contact clusters are linked by a request, the older one stays primary and the newer one is demoted to secondary.

---

## Tests

All cases were verified against the live endpoint.

**1. New contact — creates a primary**
```bash
curl -X POST https://customer-identity-reconcilation.onrender.com/identify \
  -H "Content-Type: application/json" \
  -d '{"email":"lorraine@hillvalley.edu","phoneNumber":"123456"}'
```
```json
{"contact":{"primaryContatctId":1,"emails":["lorraine@hillvalley.edu"],"phoneNumbers":["123456"],"secondaryContactIds":[]}}
```

**2. Same phone, new email — creates a secondary and merges**
```bash
curl -X POST https://customer-identity-reconcilation.onrender.com/identify \
  -H "Content-Type: application/json" \
  -d '{"email":"mcfly@hillvalley.edu","phoneNumber":"123456"}'
```
```json
{"contact":{"primaryContatctId":1,"emails":["lorraine@hillvalley.edu","mcfly@hillvalley.edu"],"phoneNumbers":["123456"],"secondaryContactIds":[2]}}
```

**3. Email only / phone only — returns existing cluster**
```bash
curl -X POST https://customer-identity-reconcilation.onrender.com/identify \
  -H "Content-Type: application/json" \
  -d '{"email":"mcfly@hillvalley.edu"}'
```
```json
{"contact":{"primaryContatctId":1,"emails":["lorraine@hillvalley.edu","mcfly@hillvalley.edu"],"phoneNumbers":["123456"],"secondaryContactIds":[2]}}
```

**4. Two separate primaries get linked — older stays primary, newer is demoted**
```bash
# Two independent contacts exist (IDs 3 and 4).
# A request arrives linking george's email with biff's phone.
curl -X POST https://customer-identity-reconcilation.onrender.com/identify \
  -H "Content-Type: application/json" \
  -d '{"email":"george@hillvalley.edu","phoneNumber":"717171"}'
```
```json
{"contact":{"primaryContatctId":3,"emails":["george@hillvalley.edu","biffsucks@hillvalley.edu"],"phoneNumbers":["919191","717171"],"secondaryContactIds":[4]}}
```

**5. Missing identifiers — returns 400**
```bash
curl -X POST https://customer-identity-reconcilation.onrender.com/identify \
  -H "Content-Type: application/json" \
  -d '{}'
```
```json
{"error":"Provide at least one of email or phoneNumber."}
```

---

## Deployment (Render)

1. Push to GitHub
2. Create a new **Web Service** on Render and connect the repo
3. Set the following:
   - **Build command:** `pnpm install --frozen-lockfile && pnpm build`
   - **Start command:** `pnpm start`
4. Add environment variables: `DATABASE_URL`, `DIRECT_URL` (both pointing to Supabase session pooler)

Migrations run automatically on each deploy via `prisma migrate deploy`.
