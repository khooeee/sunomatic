# sunomatic

Makes a fresh high-energy instrumental gym playlist on [Suno](https://suno.com)
every day.

It's an agent skill, not an app. An agent such as Claude Code follows the
instructions and does the work through the Suno MCP.

## What you get

Each run makes one private playlist named `Gym YYYY-MM-DD`:

- 10 new instrumental songs, aimed at 6 minutes each, so about an hour
- A different genre for every song
- High energy throughout, in shuffled order

Your taste comes from one place: the songs in your Suno playlist named `Gym`.

## Requirements

- A Suno plan that includes the `v6` model and advanced controls
- The Suno MCP, signed in to your Suno account
- A Suno playlist named exactly `Gym`, with songs you like working out to

## Use

Start an agent session in this repo and say:

> make my gym mix

Running it twice in one day won't make a second playlist.

## How it works

1. Reads your `Gym` playlist to learn what you like.
2. Picks a genre for each song, avoiding repeats from recent mixes.
3. Generates the songs. Suno returns 2 similar songs per call, so it listens to
   both and keeps one.
4. Rules out songs with vocals or low energy.
5. Builds the playlist.

Songs it uses go in a `Gym` workspace, and the rest go in `Gym Spares`. It never
changes your `Gym` playlist.

## Cost

About 100 Suno credits per run, or 120 with both retries. This assumes 10 credits
per call and hasn't been measured yet. A run won't start with under 180 credits,
and stops if the first call costs more than 15.

## Change it

| To change | Edit |
|---|---|
| What future mixes sound like | Your `Gym` playlist on Suno |
| Genres, BPMs, and sounds to avoid | [flavours.md](.agents/skills/gym-mix/flavours.md) |
| Number of songs, length, model, names, and credit limits | Settings in [SKILL.md](.agents/skills/gym-mix/SKILL.md) |
