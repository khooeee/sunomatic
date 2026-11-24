---
name: gym-mix
description: >-
  Makes today's gym playlist on Suno: 10 new high-energy instrumental songs in mixed
  genres, in a private dated playlist. Spends Suno credits. Use only when the user
  asks to make their gym mix or gym playlist, or invokes gym-mix.
---

# Gym Mix

Make one private Suno playlist for today's workout, using the Suno MCP tools.

Running this skill approves **one `audio-create` call per song** (10) and up to
**2 retries** for failed calls. Anything beyond that needs the user. If nobody is
there to ask, stop and report.

Suno returns 2 similar songs per call. Use **only the better one**. The other
never goes in a mix.

## Settings

Edit these to change every mix.

| Setting | Value |
|---|---|
| Songs per mix | 10 |
| Playlist name | `Gym YYYY-MM-DD` (local date) |
| Workspace for songs used in mixes | `Gym` |
| Workspace for unused songs | `Gym Spares` |
| Taste playlist (read only) | `Gym` (exact name, not the dated ones) |
| Model | `v6` |
| Target length | 360 seconds per song (Suno's maximum) |
| Controls | `style_influence` 0.8, `weirdness` 0.35, `variety` 0 |
| Minimum credits to start | 180 |
| Maximum credits per call | 15 |

## Never

- Use Max Mode, download songs, or make anything public.
- Edit or remove songs from any playlist other than today's.
- Name an artist, song, film, show, or game in a style or title.

## Steps

### 1. Check the starting state

1. Get the local date with `date`.
2. Find today's playlist with `playlists-list`. Match the name exactly, because
   `query` matches partial names.
   - It has songs: report it and stop.
   - It is empty: carry on, and reuse it in step 6.
3. Read `credits_remaining` with `account-usage`. Stop if it is below the minimum.
4. Find the `Gym` and `Gym Spares` workspaces with `workspaces-list` (exact
   names). Create any that are missing.
5. List songs in `Gym` created today. `created_at` is UTC, so convert it first.
   If there are any, an earlier run stopped partway: keep them and make only the
   missing songs, in genres not already used today. Two songs with the same
   `created_at` came from the same call.

### 2. Learn what the user likes

The user's taste is what they put in the taste playlist. Use nothing else: not
likes, other playlists, or the rest of the library.

1. Find it with `playlists-list`, matching the name exactly. If it is missing,
   stop and report. Read every song with `playlists-list-clips`, paging to the end.
2. Instrumental songs count most. From songs with vocals, use the genre,
   instruments, and energy, and ignore the voices and lyrics.
3. From each song's `metadata.style`, note its genre, BPM, instruments, textures,
   and arrangement moves.

Also read the most recent `Gym YYYY-MM-DD` playlists, up to 3, only to see which
genres were used and which styles not to repeat.

### 3. Pick a genre and write a style for each song

Every song is high energy. There is no warm-up, cool-down, or set order.

Pick the genres like this:

- Take them from the taste playlist.
- Use a different genre for every song. Close cousins count as the same genre. If
  there are too few, blend two.
- Use at least three that were not in the last mix, if there are enough.
- Otherwise choose at random, leaning towards genres common in the taste playlist.
- Skip a slow genre, or use its instruments at a faster tempo.

Shuffle them, so similar genres and tempos are not back to back.

Pick a BPM that suits the genre, close to the taste songs' BPM and never below 110.
Work in a few sounds from the taste songs, without copying any one song. Write each
style in 300–600 characters, in this order:

1. Genre, exact BPM, and "instrumental"
2. Drums, bass, and two or three lead sounds
3. How the track moves: opening, build, first peak, a change halfway (new lead,
   rhythm, or key), a bigger second peak, ending
4. The mix: low end, width, grit

Get to full energy within 20 seconds. No long breakdowns. Give every song a
different lead instrument and a short, original title.

### 4. Generate

Call `audio-create` once per song, one at a time and in the shuffled order, with:

- `instrumental: true`, `style`, `title`
- `negative_style`: everything in [avoid-list.md](avoid-list.md), comma-separated
- `model`, `controls`, and `duration` from Settings
- `duration_strict: false`, so songs end naturally instead of cutting off
- `workspace_id`: the `Gym` workspace
- `idempotency_key`: `gym-YYYY-MM-DD-song-N`

Do not wait for a song to finish before making the next call.

After the first call, read `account-usage` again. If the call cost more than the
maximum, stop and report.

A `409` means that call was already made today. Reuse its songs.

Wait with `audio-wait` (`until: complete`, up to 12 ids per call) until every
song is settled. If one song from a call failed, use the other. If both failed,
retry that call once with `-retry` added to its key.

### 5. Keep one song per call

Ask `audio-listen` about both songs from each call, at most 5 at a time. It is
free.

> Is there any singing, rapping, or speech? Rate the energy and the mix quality
> from 1 to 10. End with a JSON block with vocals (true/false), energy, quality,
> and bpm.

Choose between the two like this:

1. Rule out a song with vocals or with energy below 6. Chopped vocal samples are
   fine.
2. Keep the one with more energy. Break ties by quality, then by the BPM closest
   to the one you asked for.
3. If a check failed, choose from what you know and report the song as unchecked.

Move every song you did not keep to `Gym Spares`. If neither song passes, that
call adds nothing. Do not regenerate it.

### 6. Build the playlist

1. Create the playlist with `playlists-create` (private) and list the genres in
   one line. If today's empty playlist already exists, reuse it.
2. Add today's kept songs in the shuffled order with one `playlists-add-clips` call. Use
   new songs only. Check `failed`, and retry those once.

### 7. Report

Keep it short:

- Playlist link, genres, number of songs, total minutes
- Credits spent and credits left
- Anything ruled out, unchecked, or failed

Remind the user how to steer the next mix: add songs they like to the taste
playlist to get more like them, or add sounds to avoid-list.md to get fewer.
