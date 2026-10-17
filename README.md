# Synergie-Bingo 3000™

Gemeinsames Buzzword-Bingo für Meetings. Alle tragen Begriffe ins 5×5-Raster ein; fällt einer im Meeting, wird er abgehakt. Kein Wettbewerb, nur gelebte Teamkultur.

- Statische Seite (`index.html`) für GitHub Pages
- Live-Sync über Supabase (Tabelle `bingo_cells`, Realtime)
- Mehrere Boards über den Link-Anker, z. B. `…/#weekly` oder `…/#sprint-review`

## Supabase-Setup:

```sql
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
```

URL und Publishable Key stehen in `index.html` unter `BINGO_CONFIG`. Der Key ist öffentlich gedacht; jede Person mit dem Link kann Boards bearbeiten.
