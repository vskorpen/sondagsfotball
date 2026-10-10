# Søndagsfotball

Sign-up page for Sunday football at 20:30 in Sandardhallen.

**Live page:** https://vskorpen.github.io/sondagsfotball/

## What it does

- Players type their name and press **Jeg kan** (I'm in) or **Jeg kan ikke** (I'm out).
- The page shows two lists, **Kommer** and **Kommer ikke**, with a count for each. Players in the first list are numbered in sign-up order.
- Pressing the other button later moves you to the other list. Names are matched case-insensitively, so "ola" and "Ola" are the same player.
- The page refreshes the list every 30 seconds and whenever you come back to the tab.

## Weekly reset (every Monday at 06:00)

No scheduled job is involved. The reset is worked out from the date:

1. The page works out which Sunday it is for, using Norwegian time (`Europe/Oslo`). From Monday 06:00 until the next Monday 06:00, that is the coming Sunday.
2. The stored list carries the Sunday it belongs to, e.g. `"week": "2026-10-11"`.
3. When the page reads the list and the stored Sunday doesn't match the current one, it treats the list as empty. The next sign-up overwrites the old list.

The heading date ("Søndag 11. oktober kl. 20:30") is computed the same way, so it moves to the next Sunday at the same moment. To change the reset hour, edit `RESET_HOUR` in `index.html`.

## Storage

**The shared list** is one small JSON record on [textdb.dev](https://textdb.dev), a free key-value service that needs no account. Its address is in `store.json`:

```json
{ "url": "https://textdb.dev/api/data/<secret-key>" }
```

The record looks like this:

```json
{
  "week": "2026-10-11",
  "players": [
    { "name": "Vetle", "status": "in", "ts": 1791629123875 }
  ]
}
```

`status` is `"in"` or `"out"`, and `ts` is when the player last pressed a button.

- **Reading:** `GET` the URL.
- **Writing:** `POST` the whole record as the body, with `Content-Type: text/plain`. Other content types make textdb.dev store a mangled value or fail.
- **Simultaneous sign-ups:** each write reads the latest list, changes only your own entry and writes it back. It then reads the list again to check your entry stuck, and retries up to four times if someone else overwrote it at the same moment.

**Your name** is saved in a cookie (`fotball_navn`) in your own browser for about a year, so it's filled in next time.

### Things to know

- Anyone with the page link can sign up, and anyone who finds the URL in `store.json` could change the list. That's fine for a sign-up sheet but not for anything private.
- textdb.dev is a small free service with no uptime guarantee. If it goes away, pick another store and update `load()`/`save()` in `index.html` and the URL in `store.json`.
- To clear the list by hand, post an empty record:
  ```sh
  curl -X POST -H 'Content-Type: text/plain' \
    --data-binary '{"week":null,"players":[]}' "$(jq -r .url store.json)"
  ```
- To start over with a brand-new list, change the key at the end of the URL in `store.json` to a new random string.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The whole page: markup, styling and script, no build step |
| `header.jpg` | Header picture |
| `store.json` | Address of the shared list |

## Hosting

The site is served by GitHub Pages from the `main` branch root. Pushing to `main` redeploys it within a minute or two.
