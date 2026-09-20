# Services Blue Sky Thinking

Site for the NZF Services Team strategy days, 30 September and 1 October 2026.

`index.html` is the whole thing. One file, no build step, no dependencies to install.

---

## Putting it online

Create a repo, drop `index.html` in the root, then Settings → Pages → deploy from branch → `main` / root. It'll be live at `https://<your-username>.github.io/<repo>/` within a minute or two.

Note that free GitHub Pages needs a **public** repo, which means the strategy is readable by anyone who finds the URL. If that's not what you want, either use a private repo on a paid plan or keep the site on claude.ai instead.

---

## Live answers

Out of the box the site runs in `none` mode. Everything works, but anything anyone submits stays on their own device. Good for testing, useless in a room.

To switch it on, open `index.html`, find the `STORE` block near the top of the script (search for `DATA LAYER`) and fill in one of the two options.

### Option A — Supabase (recommended)

Responses push instantly, simultaneous submissions can't overwrite each other, and the key you publish is designed to be public.

1. Create a free project at supabase.com.
2. SQL Editor → paste and run:

```sql
create table submissions (
  id    uuid primary key default gen_random_uuid(),
  block text   not null,
  text  text   not null,
  at    bigint not null
);

create table picks (
  id       text primary key,
  exercise text   not null,
  payload  jsonb  not null,
  at       bigint not null
);

alter table submissions enable row level security;
alter table picks       enable row level security;

create policy "read submissions"  on submissions for select using (true);
create policy "add submissions"   on submissions for insert with check (true);
create policy "read picks"        on picks for select using (true);
create policy "add picks"         on picks for insert with check (true);
create policy "update picks"      on picks for update using (true);

alter publication supabase_realtime add table submissions;
alter publication supabase_realtime add table picks;
```

3. Authentication → Providers → turn **Anonymous sign-in** on. Without this nobody can submit.
4. Settings → API → copy the Project URL and the anon / publishable key.
5. In `index.html`:

```js
var STORE = {
  mode: "supabase",
  supabase: {
    url:     "https://xxxxxxxx.supabase.co",
    anonKey: "eyJhbGci..."
  },
  ...
```

**Free projects pause after a week of idle.** Set this up well ahead and it will be asleep on the morning of the 30th. Open the Supabase dashboard the day before to wake it, and load the site once that morning to be sure.

### Option B — JSONBin

Simpler to set up, more fragile in a room. Whole-bin writes mean two people submitting at the same second can overwrite each other, which is exactly what happens after a breakout. There's a retry in the code that catches most of it, not all of it.

1. Create a bin at jsonbin.io containing `{"submissions": [], "picks": {}}`.
2. Create an Access Key with read and update permission on that bin.
3. In `index.html`:

```js
var STORE = {
  mode: "jsonbin",
  ...
  jsonbin: { binId: "6xxxxxxxxxxxxxx", accessKey: "$2a$10$...", pollMs: 5000 }
};
```

**Only the wall screen polls.** Open the projector's browser at `...index.html?wall` and it'll refresh every few seconds. Phones opened at the plain URL submit without polling, which is what keeps you inside the free request allowance. On Supabase this doesn't apply, everyone gets live updates.

---

## On the day

- Projector: open the site with `?wall` on the end if you're on JSONBin.
- Present mode: open any session and press **Present mode**. Arrow keys or the on-screen buttons advance. Escape exits.
- Phones: share the plain URL, or a QR code pointing at it.

---

## Adding the film

When the Yusuf film is cut, drop it in the repo as `yusuf.mp4` and tell Claude. The seven story frames in the vision session get replaced by the video.

Keep it under about 50MB for GitHub's comfort. There's no 18MB ceiling here, unlike the claude.ai version, so you can use a better quality export.

---

## Still to fill in

- Pre-reading has space for anything else you want people to read beforehand.
- Amyn's two impact measures sessions on day two are deliberately empty, waiting on his content.
- The third cross-cutting theme is chosen in the room on day one.
