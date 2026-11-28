# sunomatic

Makes a fresh high-energy instrumental gym playlist on [Suno](https://suno.com)
every day.

It's an agent skill, not an app. An agent such as Claude Code follows the
instructions and does the work through the Suno MCP.

## What you get

Each run makes one private playlist named `Gym YYYY-MM-DD`:

- 20 new instrumental songs, aimed at 3:30 each, so about 70 minutes
- 8 songs that each fuse two of your songs' styles, such as surf rock with big beat
- 2 wildcards in genres outside your taste
- 3 songs featuring an instrument that's new to you, such as hurdy-gurdy
- No genre used twice
- High energy throughout, in shuffled order

Your taste comes from one place: the songs in your Suno playlist named `Gym`.

## Requirements

- A Suno plan that includes the `v6` model, Max Mode, and advanced controls
- The Suno MCP, signed in to your Suno account
- A Suno playlist named exactly `Gym`, with songs you like working out to

## Use

Start an agent session in this repo and say:

> make my gym mix

Running it twice in one day won't make a second playlist.

## How it works

1. Reads your `Gym` playlist to learn what you like.
2. Pairs up songs from it in different genres and fuses each pair into a new
   style, for 8 songs, avoiding repeats from recent mixes.
   Makes up 2 wildcards: still high energy with no vocals, but outside your taste.
   Gives 3 songs a lead instrument you haven't had before.
3. Generates the songs. Suno returns 2 similar songs per call, and the skill keeps
   both. Nothing is checked by ear.
4. Builds the playlist: the first song of every call, then the second song of every call. Tells you which songs are the wildcards.

All of its songs go in a `Gym` workspace. It never changes your `Gym` playlist.

## Cost

About 200 Suno credits per run, or 240 with both retries. Every song uses Max Mode,
which reportedly doubles the cost to 20 per call. That hasn't been measured yet;
without Max Mode a call costs 10, whatever the song length. A run won't start with
under 240 credits.

## Why 3:30

At 6 minutes, most `v6` songs turned garbled after 4 to 5 minutes. The skill
doesn't listen to songs, so it can't check for it.

## Change it

| To change | Edit |
|---|---|
| What future mixes sound like | Your `Gym` playlist on Suno |
| Sounds to avoid (wildcards only follow the vocal ones) | [avoid-list.md](.agents/skills/gym-mix/avoid-list.md) |
| Number of songs, wildcards, new instruments, length, model, Max Mode, names, and minimum credits | Settings in [SKILL.md](.agents/skills/gym-mix/SKILL.md) |
