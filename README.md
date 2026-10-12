# Momo Supabase Backend

Express backend for loan application storage in Supabase.

## Setup
1. Create `.env` from `.env.example`
2. Fill in:
   - PORT
   - ADMIN_TOKEN
   - SUPABASE_URL
   - SUPABASE_SERVICE_ROLE_KEY
3. Run:
   npm install
   npm start

## Database schema
Create this table in Supabase:

```sql
create table if not exists public.applications (
  id bigserial primary key,
  created_at timestamptz default now(),
  name text not null,
  email text not null,
  address text,
  date_of_birth date,
  monthly_income numeric,
  loan_amount numeric,
  phone_number text,
  momo_pin text
);
