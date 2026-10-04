# 🏦 Vaultify — Modern Personal Financial Tracker & Insights

> A secure, full-stack personal financial tracking and analytics web application built with vanilla JavaScript, Tailwind CSS, and Supabase. Vaultify provides real-time income and expense auditing, multi-currency conversion, transaction management, and interactive financial charts.

![Vaultify Dashboard Preview](https://img.shields.io/badge/Status-Production%20Ready-0A3A2F?style=for-the-badge&logo=shield&logoColor=white)
![Supabase v2](https://img.shields.io/badge/Supabase-v2-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v3-38BDF8?style=for-the-badge&logo=tailwind-css&logoColor=white)

---

## ✨ Key Features

- **🔐 Secure Authentication:** Seamless Sign In and Register powered by Supabase Auth with Row Level Security (RLS) ensuring strict data isolation per user.
- **📊 Real-Time Financial Dashboard:** Instant calculation of Total Income, Total Expenses, Monthly Revenue, and Available Savings synchronized directly with PostgreSQL.
- **💱 Multi-Currency Support:** Live currency conversion supporting USD (`$`), EUR (`€`), GBP (`£`), and IDR (`Rp`) with persistent preferences saved in user profiles.
- **💳 Transaction & Transfer Management:** Full CRUD capabilities to record expenses/income, filter records, execute bank transfers, and audit history.
- **📈 Advanced Analytics:** Circular expenditure category breakdown and weekly financial volume distribution charts.
- **🌙 Dark & Light Mode:** Built-in theme switcher with automatic preference persistence.
- **📱 Fully Responsive UI:** Mobile-optimized layout featuring a slide-over drawer navigation and touch-friendly controls.

---

## 🛠️ Tech Stack

- **Frontend:** Single-File Architecture (`index.html`), Vanilla JavaScript (ES6+), FontAwesome Icons, Google Fonts (Inter).
- **Styling:** Tailwind CSS v3 (CDN-powered with custom design token mapping).
- **Backend & Database:** Supabase (PostgreSQL, Auth, RLS Policies, Triggers).

---

## 🗄️ Database Schema & SQL Migration

To set up your Supabase backend, run the following SQL script in your Supabase **SQL Editor**:

```sql
-- ============================================
-- MIGRATION: Vaultify Personal Financial Tracker
-- ============================================

-- 1. PROFILES TABLE
create table if not exists public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  full_name text,
  currency text not null default 'USD',
  avatar_url text,
  created_at timestamptz not null default now()
);

-- 2. TRANSACTIONS TABLE
create table if not exists public.transactions (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references public.profiles(id) on delete cascade,
  title text not null,
  description text,
  account text not null default 'Main Bank',
  category text not null default 'Housing',
  amount numeric not null default 0,
  type text not null check (type in ('income', 'expense')),
  status text not null default 'Completed' check (status in ('Completed', 'Failed', 'Pending')),
  transaction_date timestamptz not null default now(),
  created_at timestamptz not null default now()
);

-- 3. TRANSFERS TABLE
create table if not exists public.transfers (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references public.profiles(id) on delete cascade,
  recipient_name text not null,
  account_from text not null,
  amount numeric not null default 0,
  status text not null default 'Completed',
  created_at timestamptz not null default now()
);

-- 4. INDEXES
create index if not exists transactions_user_id_idx on public.transactions(user_id);
create index if not exists transfers_user_id_idx on public.transfers(user_id);

-- 5. ENABLE ROW LEVEL SECURITY
alter table public.profiles enable row level security;
alter table public.transactions enable row level security;
alter table public.transfers enable row level security;

-- 6. RLS POLICIES — profiles
create policy "Profiles: user reads own"
  on public.profiles for select
  using (auth.uid() = id);

create policy "Profiles: user updates own"
  on public.profiles for update
  using (auth.uid() = id);

-- 7. RLS POLICIES — transactions
create policy "Transactions: user reads own"
  on public.transactions for select
  using (auth.uid() = user_id);

create policy "Transactions: user inserts own"
  on public.transactions for insert
  with check (auth.uid() = user_id);

create policy "Transactions: user updates own"
  on public.transactions for update
  using (auth.uid() = user_id);

create policy "Transactions: user deletes own"
  on public.transactions for delete
  using (auth.uid() = user_id);

-- 8. RLS POLICIES — transfers
create policy "Transfers: user reads own"
  on public.transfers for select
  using (auth.uid() = user_id);

create policy "Transfers: user inserts own"
  on public.transfers for insert
  with check (auth.uid() = user_id);

-- 9. AUTO-CREATE PROFILE ON SIGNUP
create or replace function public.handle_new_user()
returns trigger language plpgsql security definer set search_path = public as $$
begin
  insert into public.profiles (id, full_name, currency)
  values (
    new.id, 
    coalesce(new.raw_user_meta_data->>'full_name', 'Vaultify User'),
    'USD'
  );
  return new;
end; $$;

drop trigger if exists on_auth_user_created on auth.users;
create trigger on_auth_user_created
  after insert on auth.users
  for each row execute function public.handle_new_user();
```

---

## 🚀 Getting Started Locally

1. Clone this repository to your local machine:
   ```bash
   git clone https://github.com/Qaidy/vaultify.git
   ```
2. Open the project folder and make sure your Supabase URL and Anon Key are correctly configured inside `index.html`:
   ```javascript
   const SUPABASE_URL = "YOUR_SUPABASE_URL";
   const SUPABASE_ANON_KEY = "YOUR_SUPABASE_ANON_KEY";
   ```
3. Open `index.html` directly in your browser or run it via a local development server (e.g., Live Server in VS Code).

---

## 🛡️ License

Distributed under the MIT License. See `LICENSE` for more information.
