# month-end
A fun survival game where you dodge debit orders and survive from the 25th to the 25th
# Month-End deploy checklist

## 1. Deploy (about 5 min)
1. Create a new GitHub repo, upload index.html and og.png (unzipped).
2. Vercel > Add New > Project > import the repo > Deploy. No settings needed.
3. Project Settings > Name: rename to something short, e.g. month-end. You get month-end.vercel.app free, if the name is free.

## 2. Custom link
- Free option: the renamed vercel.app address above.
- Own domain: buy one at a registrar (a .co.za is the natural fit; check current price and registrar rules yourself), then Vercel > Settings > Domains > Add, and copy the DNS records Vercel shows into your registrar.
- Then replace both YOUR-LINK placeholders in index.html (og:url, og:image) with your final address, commit, and redeploy. Without this, WhatsApp previews will not show the card.

## 3. Analytics + leaderboard (Supabase, free tier)
1. Create a Supabase project > SQL Editor > run:

create table runs(id bigint generated always as identity primary key, created_at timestamptz default now(), nick text, tribe text, score int, won boolean, day int);
create table events(id bigint generated always as identity primary key, created_at timestamptz default now(), name text, tribe text, from_challenge boolean);
alter table runs enable row level security;
alter table events enable row level security;
create policy "ins runs" on runs for insert to anon with check (score between 0 and 99999 and char_length(nick) between 1 and 14 and tribe in ('Capitech','First Nationwide','Standard Bunk','Absolutely'));
create policy "ins events" on events for insert to anon with check (name in ('open','start','finish','share','challenge_open') and tribe in ('Capitech','First Nationwide','Standard Bunk','Absolutely'));
create view tribe_board as select tribe, count(*)::int plays, round(avg(score))::int avg_score, max(score) best from runs group by tribe;
create view leaderboard as select nick, tribe, max(score) score from runs group by nick, tribe;
grant select on tribe_board, leaderboard to anon;

If you already ran the older SQL, run this instead:

alter table runs add column nick text;
drop policy "ins runs" on runs;
create policy "ins runs" on runs for insert to anon with check (score between 0 and 99999 and char_length(nick) between 1 and 14 and tribe in ('Capitech','First Nationwide','Standard Bunk','Absolutely'));
create view leaderboard as select nick, tribe, max(score) score from runs group by nick, tribe;
grant select on leaderboard to anon;

2. Project Settings > API: copy the Project URL and the anon public key into SB_URL and SB_KEY near the top of the script in index.html. Commit.
3. Optional: Vercel > Analytics > Enable Web Analytics (adds page-view counts).

## 4. Read your test results
select name, count(*) from events group by name;
Pass bars: finish/start over 60%, share/finish over 15%. Runs per start over 1.3 means replays.

## Known weaknesses
- Scores are client-submitted, so anyone can fake them. Fine for a test, not for prizes.
- Tribe standings are by average score and have no provinces yet.
- Open Graph previews are cached by WhatsApp; test the link in a chat with yourself before the launch.

## Nicknames: privacy and moderation
- Collect a nickname only. Never ask for real names, phone numbers or emails; that brings POPIA obligations (consent, purpose, storage, deletion) you do not want for a test.
- The built-in word filter is tiny. Check the table daily: Supabase > Table Editor > runs. To remove a name: delete from runs where nick = 'TheName';
- Anyone can type any nickname, so players can impersonate each other. There are no accounts.
