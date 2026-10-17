# Meeting-Bingo

Gemeinsames Bingo für Meetings. Alle tragen Begriffe ins 5×5-Raster ein; fällt einer im Meeting, wird er abgehakt.

- Statische Seite (`index.html`) für GitHub Pages
- Live-Sync über Supabase (Realtime)
- Mehrere Boards über den Link-Anker, z. B. `…/#weekly` oder `…/#sprint-review`
- „Häufig genutzt“: zeigt automatisch die meistgenutzten Begriffe des Boards

## Supabase-Setup

```sql
-- Aktuelles Raster
create table public.bingo_cells (
  board text not null check (board ~ '^[a-z0-9_-]{1,40}$'),
  cell smallint not null check (cell between 0 and 24 and cell <> 12),
  text text not null check (char_length(text) between 1 and 60),
  done boolean not null default false,
  added_by text check (added_by is null or char_length(added_by) <= 30),
  updated_at timestamptz not null default now(),
  primary key (board, cell)
);
alter table public.bingo_cells enable row level security;
create policy "bingo read"   on public.bingo_cells for select to anon, authenticated using (true);
create policy "bingo insert" on public.bingo_cells for insert to anon, authenticated with check (true);
create policy "bingo update" on public.bingo_cells for update to anon, authenticated using (true) with check (true);
create policy "bingo delete" on public.bingo_cells for delete to anon, authenticated using (true);
grant select, insert, update, delete on table public.bingo_cells to anon, authenticated;
alter table public.bingo_cells replica identity full;
alter publication supabase_realtime add table public.bingo_cells;

-- Verlauf für "Häufig genutzt"
create table public.bingo_term_uses (
  id bigint generated always as identity primary key,
  board text not null check (board ~ '^[a-z0-9_-]{1,40}$'),
  term text not null check (char_length(term) between 1 and 60),
  created_at timestamptz not null default now()
);
create index bingo_term_uses_board_idx on public.bingo_term_uses (board);
alter table public.bingo_term_uses enable row level security;
create policy "term uses read"   on public.bingo_term_uses for select to anon, authenticated using (true);
create policy "term uses insert" on public.bingo_term_uses for insert to anon, authenticated with check (true);
grant select, insert on table public.bingo_term_uses to anon, authenticated;

create view public.bingo_top_terms with (security_invoker = true) as
select board, mode() within group (order by term) as term, count(*)::int as uses, max(created_at) as last_used
from public.bingo_term_uses
group by board, lower(term);
grant select on public.bingo_top_terms to anon, authenticated;
```

URL und Publishable Key stehen in `index.html` unter `BINGO_CONFIG`. Der Key ist öffentlich gedacht; jede Person mit dem Link kann Boards bearbeiten.
