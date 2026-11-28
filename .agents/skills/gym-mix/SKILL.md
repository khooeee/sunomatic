---
name: gym-mix
description: >-
  Makes today's gym playlist on Suno: 20 new high-energy instrumental songs in mixed
  genres, in a private dated playlist. Spends Suno credits. Use only when the user
  asks to make their gym mix or gym playlist, or invokes gym-mix.
---

# Gym Mix

Make one private Suno playlist for today's workout, using the Suno MCP tools.

Running this skill approves **10 `audio-create` calls** (2 songs each) and up to
**2 retries** for failed calls, all in Max Mode. Anything beyond that needs the
user. If nobody is there to ask, stop and report.

Suno returns 2 similar songs per call. Both go in the playlist: the first song
of every call in part 1, then the second song of every call in part 2.

## Settings

Edit these to change every mix.

| Setting | Value |
|---|---|
| Calls per mix | 10 (each gives 2 songs, so 20 in the playlist) |
| Wildcards | 2 of the 10 calls |
| New instrument | 3 of the 8 fused calls |
| Playlist name | `Gym YYYY-MM-DD` (local date) |
| Workspace for songs used in mixes | `Gym` |
| Taste playlist (read only) | `Gym` (exact name, not the dated ones) |
| Model | `v6` |
| Max Mode | On for every song (`max_mode: true`) |
| Target length | 210 seconds (3:30) per song. Longer `v6` songs turn garbled |
| Controls | `style_influence` 0.8, `weirdness` 0.35, `variety` 0 |
| Minimum credits to start | 240 |

## Avoid list

Sounds to keep out of every song, sent to Suno as the negative style. Wildcards
only avoid the vocal ones: the first five. Edit freely, one per line.

- vocals
- singing
- rap
- spoken word
- choir
- slow ballad
- acoustic folk
- ambient
- lo-fi
- trap
- dubstep
- long intro
- long breakdown
- fade-out

## Never

- Download songs or make anything public.
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
4. Find the `Gym` workspace with `workspaces-list` (exact name). Create it if missing.
5. List songs in `Gym` created today. `created_at` is UTC, so convert it first.
   If there are any, an earlier run stopped partway: keep them and make only the
   missing calls, in genres not already used today. Two songs with the same
   `created_at` came from the same call. Songs in genres outside the taste
   playlist were wildcards.

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

Make every song except the wildcards a fusion of two taste songs:

- Pick two taste songs in different genres, such as surf rock and big beat. Take
  the base genre and groove from one, and signature sounds or arrangement moves
  from the other.
- Use each genre at most once per mix. Close cousins count as the same genre. If
  there are too few, reuse a genre, but never a pair.
- Use at least three genres that were not in the last mix, if there are enough.
- Otherwise choose at random, leaning towards genres common in the taste playlist.
- Skip a slow genre, or use its instruments at a faster tempo.

The wildcards are songs the user would not expect. Be creative:

- Pick a genre or an unusual blend that is not in the taste playlist, and not a
  close cousin of anything in it. Make the two very different from each other.
- Do not repeat a wildcard from the recent mixes.
- They must have no vocals and enough energy for a workout. Nothing else about the
  sound is fixed: the BPM, style, and arrangement rules below do not apply.

Give some fused calls a new instrument, as many as Settings says (both songs from
each such call share it):

- Pick a real instrument the user has likely never heard in their music, such as
  hurdy-gurdy, duduk, or cimbalom. It must not appear in any taste song's style
  or in the recent mixes, and each song gets a different one.
- Make it one of the lead sounds and give it a featured part, such as the main
  hook or the halfway change, so it is clearly heard.

Shuffle all the songs, so similar genres and tempos are not back to back. The
wildcards can go anywhere.

Pick a BPM that suits the fusion, between the two songs' BPMs where possible and
never below 110. Take sounds and moves, not melodies, so neither song is copied.
Write each style in 300–600 characters, in this order:

1. The fused genre, exact BPM, and "instrumental"
2. Drums, bass, and two or three lead sounds
3. How the track moves: opening, build, first peak, a change halfway (new lead,
   rhythm, or key), a bigger second peak, ending
4. The mix: low end, width, grit

Get to full energy within 20 seconds. No long breakdowns. Give every song a
different lead instrument and a short, original title.

### 4. Generate

Call `audio-create` once per call, one at a time and in the shuffled order, with:

- `instrumental: true`, `style`, `title`
- `negative_style`: every entry in the Avoid list, comma-separated. For
  wildcards, only the vocal entries
- `model`, `max_mode`, `controls`, and `duration` from Settings
- `duration_strict: false`, so songs end naturally instead of cutting off
- `workspace_id`: the `Gym` workspace
- `idempotency_key`: `gym-YYYY-MM-DD-song-N`

Do not wait for a song to finish before making the next call.

A `409` means that call was already made today. Reuse its songs. If Max Mode is
refused, stop and report. Do not make songs without it.

Wait with `audio-wait` (`until: complete`, up to 12 ids per call) until every
song is settled. If one song from a call failed, use the other. If both failed,
retry that call once with `-retry` added to its key.

### 5. Keep both songs from each call

Do not use `audio-listen`, and do not choose between takes. Nothing is checked by
ear: energy, mix quality, and whether a new instrument is heard are all unverified,
and the report says so. `instrumental: true` keeps vocals out of every take.

### 6. Build the playlist

1. Create the playlist with `playlists-create` (private) and list the genres in
   one line. If today's empty playlist already exists, reuse it.
2. Add the first song of each call in call order (part 1), then the second song of
   each call in call order (part 2), with one `playlists-add-clips` call. Use new
   songs only. Check `failed`, and retry those once. If one take of a call failed,
   the other take fills its part 1 slot.
3. Move every song into the `Gym` workspace with `workspaces-update-clips`. Check
   `skipped`.
4. Rename the two takes of each call so each title is unique: part 1 gets
   `<title> 1` and part 2 gets `<title> 2`, with `audio-edit`. Check that each title
   is unique before reporting.

### 7. Report

Keep it short:

- Playlist link, genres, number of songs, total minutes
- The two taste songs each song fused
- Each new instrument and its song (not checked by ear)
- The titles of the wildcards, with their genres
- Credits spent and credits left
- Anything failed, or any wildcard that did not finish

Remind the user how to steer the next mix: add songs they like to the taste
playlist to get more like them, or add sounds to the Avoid list in this skill to get fewer.
