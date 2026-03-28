create extension if not exists "pgcrypto";

create table if not exists public.profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  username text unique not null,
  character_name text,
  bio text,
  color_index int not null default 0,
  created_at timestamptz not null default now()
);

create table if not exists public.rpg_posts (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references public.profiles(id) on delete cascade,
  parent_id uuid null references public.rpg_posts(id) on delete cascade,
  content text not null,
  created_at timestamptz not null default now()
);

alter table public.profiles enable row level security;
alter table public.rpg_posts enable row level security;

create policy "profiles are readable by authenticated users"
on public.profiles
for select
to authenticated
using (true);

create policy "users can insert their own profile"
on public.profiles
for insert
to authenticated
with check (auth.uid() = id);

create policy "users can update their own profile"
on public.profiles
for update
to authenticated
using (auth.uid() = id)
with check (auth.uid() = id);

create policy "posts are readable by authenticated users"
on public.rpg_posts
for select
to authenticated
using (true);

create policy "users can insert their own posts"
on public.rpg_posts
for insert
to authenticated
with check (auth.uid() = user_id);

create policy "users can update their own posts"
on public.rpg_posts
for update
to authenticated
using (auth.uid() = user_id)
with check (auth.uid() = user_id);

create policy "users can delete their own posts"
on public.rpg_posts
for delete
to authenticated
using (auth.uid() = user_id);

alter publication supabase_realtime add table public.profiles;
alter publication supabase_realtime add table public.rpg_posts;
