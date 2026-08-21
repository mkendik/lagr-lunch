# Lágr Lunch

One page, four days, one button per day. People say whether they are eating so
the cook knows how many to shop for. No login, no build step, no framework.

Live: https://mkendik.github.io/lagr-lunch/

## How it works

`index.html` is the whole app. It talks straight to the Supabase REST API with
plain `fetch` — no client library, no CDN dependency.

- Table: `public.lunch_votes` in the `nextap-attendance` project
- One row per person: `person_id` (slug), `name`, `days` (jsonb, date -> bool)
- A date missing from `days` means that person has not answered yet, which is
  what the grey dashed chips show
- Reads on load, on tab focus, and every 15s. Writes are one upsert per vote.

The key in `index.html` is Supabase's *publishable* key. It is meant to be in
client code. Row Level Security allows anonymous select/insert/update on
`lunch_votes` only — no delete policy, and a CHECK constraint limits
`person_id` to the ten people below, so nobody can add junk rows. The
attendance tables (`people`, `entries`, `app_config`) stay locked to the team
login and return nothing with this key.

## Changing things

Edit the two arrays at the top of the `<script>` in `index.html`, commit, push.
Pages redeploys in about a minute.

- `PEOPLE` — the roster. Adding someone also needs the DB constraint updated:
  ```sql
  alter table public.lunch_votes drop constraint lunch_votes_person_id_check;
  alter table public.lunch_votes add constraint lunch_votes_person_id_check
    check (person_id in ('martin','viktor', ... ,'newperson'));
  ```
- `DAYS` — dates, day labels, soup, main dish, note. `eatout: true` renders the
  restaurant card with no soup line.

## Afterwards

This is a four-day throwaway. When the offsite is over, drop the table:

```sql
drop table public.lunch_votes;
```
