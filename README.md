# Bitespeed Identity Reconciliation

A web service that identifies and consolidates customer contact information across multiple purchases on FluxKart.com.

When a customer checks out using different emails or phone numbers over time, this service links those contacts together under a single primary identity — so FluxKart always knows who it's talking to.

**Live endpoint:** `https://your-service.onrender.com/identify`

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
npm install
```

**2. Configure environment**
```bash
cp .env.example .env
```

Fill in your Supabase connection strings in `.env`:
- `DATABASE_URL` → Session pooler (port 5432)
- `DIRECT_URL` → Direct connection (port 5432)

Both are available under Supabase → Settings → Database → Connection string.

**3. Run migrations**
```bash
npm run migrate
```

**4. Start dev server**
```bash
npm run dev
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

## Deployment (Render)

1. Push to GitHub
2. Create a new **Web Service** on Render and connect the repo
3. Set the following:
   - **Build command:** `npm run build`
   - **Start command:** `npm start`
4. Add environment variables: `DATABASE_URL`, `DIRECT_URL`

Migrations run automatically on each deploy via `prisma migrate deploy`.
