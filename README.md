# bored

Say `/bored` in Claude Code. It gets to know your taste once, then gives you three picks
(a safe bet, a stretch, and something to learn about, from black holes to sourdough). It also keeps two private pages for you on claude.ai:

- **Up Next**: your watch/read queue. Tap to change status or rating.
- **Upskill Tracks**: short lessons, one level at a time. Tap how each one went.

## Install

```
/plugin marketplace add openmohit28/bored-plugin
/plugin install bored@bored-plugin
```

Then say `I'm bored`. The first run asks a few questions (about 2 minutes) and creates your pages.

## Notes

- Your profile is stored in `~/.claude/bored/taste-profile.md`, on your machine only.
- The pages need Claude Code signed in with a claude.ai account. Without that, recommendations still work, just without the pages.
- Each person gets their own pages. Nobody sees anyone else's.
