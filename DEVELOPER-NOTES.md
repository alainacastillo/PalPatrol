# Developer notes — where user answers live

## Short answer

**Everything is stored in the visitor's own browser, using `localStorage`. You cannot
see another person's answers, and neither can anyone else.**

There is no backend, no database, no analytics, and no network request of any kind.
A search of `app.js` and `index.html` for `fetch(`, `XMLHttpRequest`, `sendBeacon`,
`WebSocket` and `axios` returns **zero** matches. The site is three static files.

## Where each thing is stored

All keys below are `localStorage` keys on the visitor's device.

| Feature | Key | Contents |
| --- | --- | --- |
| 💭 Maybe Try This? | `little-rest-personal-ideas-v1` | `activities`, `foods`, `places`, `tries`, `happy`, `favorites`, `saved`, plus `mood` / `energy` / `time` / `location` |
| 🍎 Grow Your Fruit | `little-rest-fruit-v1` | Per-fruit reflection text, chosen small step, completion flag |
| 🗑️ Leave It Behind | `little-rest-leave-v1` | Items written, whether each is put away, "make space for" answers |
| 🔥 Let It Go | *(nothing)* | Deliberately never stored — the note only exists on screen |
| 📝 Journal | `journal` | Journal text |
| 📖 Calm Book | `book`, `thought`, `gratitude`, `comfort`, `places`, `steps`, `drawing` | Each little page |
| 🧴 Fill the Bottles | `bottles` | Bottle entries |
| 🎡 Reset Wheel | `wheel` | Wheel choices |
| 🙏 Faith | `verses` | Saved verses |

## What this means for you as the developer

- On **your own** browser you can read every answer with the browser console:
  ```js
  JSON.parse(localStorage.getItem('little-rest-personal-ideas-v1'))
  ```
- On **someone else's** phone or laptop, the data never leaves their device. There is
  no copy for you to look at. Clearing their browser data deletes it permanently.
- This is a genuine privacy feature, and the on-page note already tells users the
  truth: *"Your answers are saved on this device to personalize your suggestions."*

## Option A — keep localStorage (recommended default)

Change nothing. Users keep full privacy, the site stays a zero-cost static host,
and there is no legal/consent surface to manage. **You will never see other users'
answers.** For a thesis, you can still gather data by asking participants to run the
one-line console snippet above, or by adding an "Export my answers" button that
downloads their own JSON — they then choose to send it to you. That keeps consent
explicit and is usually the easiest thing to defend in a thesis write-up.

## Option B — add Supabase (only if you truly need remote answers)

Simplest secure shape, roughly 30 lines of code:

1. Create a free Supabase project.
2. Create a table, e.g. `maybe_try_this_answers`:
   `id uuid default gen_random_uuid()`, `created_at timestamptz default now()`,
   `question_key text`, `answer text`, `source text` (`'chip'` or `'custom'`),
   `client_id text` (a random per-browser id, **not** a name or email).
3. Turn **Row Level Security ON** for that table, then add exactly one policy:
   allow `INSERT` for the `anon` role, and allow **no** `SELECT`. That means the
   website can add answers but nobody can read them back through the public key.
4. You read the answers by logging into the Supabase dashboard's Table Editor.
   No admin page on the website — that is the part that would be insecure.
5. Put the URL + anon key in the front end. Because the site has no build step,
   they would live in a small `config.js` (git-ignored) or be pasted directly into
   `app.js`. The anon key is designed to be public; the RLS policy is what protects
   the data. **The `service_role` key must never appear in front-end code.**

### Things that must change if you choose Option B

- **Keep writing to `localStorage` as well.** The site must still work offline and
  must not lose its current behaviour if the network fails.
- **Update the privacy note in `index.html`.** The current wording says answers stay
  on the device, which would no longer be true. Replace it with something like:
  *"Your answers help personalize your experience. Some responses are securely stored
  to help improve and maintain the website."*
- Collect **no** names, emails, or anything identifying. A random `client_id` is enough.
- Do not store 🔥 Let It Go or 📝 Journal text remotely. Those are the most private
  things on the site, and the whole point of Let It Go is that it is not kept.

Credentials are **not** included anywhere in this repo. See `.env.example` for the
list of values you would need to obtain yourself.
