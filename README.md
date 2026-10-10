# The YouTube agent skill

## Local Creator Studio web app

This repository also includes an optional Express dashboard. On Windows, open PowerShell in the
project folder and run:

```powershell
npm install
Copy-Item .env.example .env
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Copy the generated 64-character hex value into `SETTINGS_ENCRYPTION_KEY` in `.env`. Keep this key
stable across restarts and back it up separately; changing or losing it makes encrypted settings
unreadable. Set a unique `JWT_SECRET` of at least 32 characters; the server refuses to start with a
missing or known development secret. The web app uses Firebase Authentication's Google provider
through `signInWithPopup`; the Firebase Web App configuration is served by `/api/auth/config`, and
the server validates Firebase ID tokens with Firebase Authentication before issuing its own session.
The included Firebase web configuration can be overridden with `FIREBASE_WEB_API_KEY`,
`FIREBASE_AUTH_DOMAIN`, `FIREBASE_PROJECT_ID`, `FIREBASE_APP_ID`, and optional
`FIREBASE_MEASUREMENT_ID` environment variables. In Firebase Console, enable Google at
Authentication > Sign-in method and add the app's exact host (including `localhost` for local use)
under Authentication > Settings > Authorized domains. A verified sign-in from
`rouintue@gmail.com` receives the administrator role; every other verified Google email receives
the user role. The browser cannot select a role or impersonate a seeded demo account. Admin API
routes always validate the signed session and persisted account role. User accounts are persisted
encrypted in the secure settings store; users add their own Gemini and Replicate API keys in
`Settings & integrations`. To enable the 11 Claude agents, set
11 **rotated/reissued** Anthropic API keys (one per line) in `ANTHROPIC_API_KEYS` in the private
`.env` file or deployment secret manager. The API assigns keys across the 11 agent skills and cools
down a key after rate-limit or overload responses before trying another configured key. Multiple
keys do not increase an Anthropic account's plan limits or guarantee requests will avoid throttling.
Configure the model with `ANTHROPIC_MODEL` if your account uses a different currently supported
model.
To enable channel connection and scheduled publishing, configure `YOUTUBE_CLIENT_ID`,
`YOUTUBE_CLIENT_SECRET`, and the matching `YOUTUBE_REDIRECT_URI` in `.env`; enable the YouTube Data
API and register the redirect URI in Google Cloud. Each user must save their own Replicate API token
before scheduled video generation can be enabled.

Administrators configure BitLabs (API token, secret and server-to-server key), CPX Research (App ID),
and Google AdSense publisher/placement IDs in the Admin panel. Integration values are encrypted at
rest; secret values remain masked in the UI and display a configured status after reload. AdSense
loads only the official Google script into the shared top and bottom banner placements on the user
and admin views.

To store BitLabs/CPX offerwall settings in Firebase Realtime Database, configure
`FIREBASE_DATABASE_URL`, `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, and
`FIREBASE_PRIVATE_KEY` in the server environment. The backend authenticates with the service account
and reads/writes `/settings/offerwall`; the Firebase web `apiKey` is not a server credential and
should not be used to authorize admin writes. Keep the service-account private key out of source
control and restrict client-side database rules so clients cannot read or write this settings path.
Without Firebase configuration, offerwall settings continue to use the encrypted local store and
the Admin panel indicates which backend is active.

For Realtime Database rules, deny direct client access to the admin-only settings node (the
service-account-backed server API continues to access it):

```json
{
  "rules": {
    "settings": {
      "offerwall": {
        ".read": false,
        ".write": false
      }
    }
  }
}
```

Start the server:

```powershell
npm start
```

Open `http://localhost:5000/`. The Admin view is intentionally not linked in the user sidebar; use
`http://localhost:5000/#admin` locally to reach the seeded administrator selector. Production Admin
API routes require an administrator JWT. `/admin` is the protected API namespace, not a standalone
dashboard page.

Integration settings, user API keys, YouTube tokens, schedules and run outcomes are stored in
`database/secure-settings.enc` encrypted with `SETTINGS_ENCRYPTION_KEY`. Keep this file and key on
persistent storage when deploying; ephemeral containers must mount a persistent volume or configure
`SECURE_STORE_PATH`. The scheduled worker runs only while the Node server is running. Enabling a
schedule explicitly authorizes provider usage and automatic YouTube uploads at the selected privacy
level. Review provider pricing, OAuth permissions, YouTube quota and generated videos; public uploads
are not the default.

Eleven Claude skills that run a YouTube channel. Free, MIT, no signup, no API key, nothing to
connect.

One of them writes your script off 21 hook formulas and scores the hook before you waste a take on
it. One lints the title and the thumbnail as a single pairing, because writing them separately is
why half of your click surface says the same thing twice. One reads your audience-retention export
and tells you the exact second people left and what you were saying when they did. One turns a
transcript into an edit decision list. One finds the Shorts already hiding inside a long video. One
goes and finds what is working in your niche and ranks it by how far each video beat its own
channel, not by how big the channel is.

**Nothing gets published until you do it.** These skills write. You upload.

## Install

Paste this into Claude:

```
https://github.com/Jakeschincariol/youtube-agent-skill

Install this skill, then confirm /yt-script works.
```

Or do it yourself, in Claude Code:

```bash
git clone https://github.com/Jakeschincariol/youtube-agent-skill.git
cp -r youtube-agent-skill/skills/yt-* ~/.claude/skills/
```

Or as a plugin:

```
/plugin marketplace add Jakeschincariol/youtube-agent-skill
/plugin install youtube-agent
```

Project-local instead of global: copy the same folders into your repo's `.claude/skills/`. No Claude
Code at all? Paste any single `SKILL.md` at the top of a chat and it runs as a mode. You lose the six
Python tools, which is most of the point of `/yt-script`, `/yt-retention` and `/yt-edit`, but the
rest works.

Then spend ten minutes on [`templates/voice.md`](templates/voice.md). Copy it to
`~/.claude/youtube/voice.md` and fill it in, or send Claude three of your own videos and say "write
my voice.md from these". Every skill reads that file. It matters more here than anywhere else,
because you have to say the words out loud.

## The eleven

| command | what it does |
| --- | --- |
| `/yt-script` | One idea into a script. Five hooks off [21 formulas](skills/yt-script/hooks.json), scored, then the spoken script with the retention beats marked. |
| `/yt-package` | Title and thumbnail as one pairing, linted for truncation, duplication and vagueness. |
| `/yt-edit` | A transcript into an edit decision list: dead air, filler cues, retakes, with timecodes. |
| `/yt-comment` | The comment section triaged into four piles, then replies in your voice. Says which one to pin. |
| `/yt-plan` | A week that fits the hours you actually have. One anchor, one cheap one, three Shorts. |
| `/yt-viral` | What is working in your niche, ranked by multiple over each channel's own median. |
| `/yt-retention` | Your retention export read properly: hook leak, the cliffs, the slide, and what to change. |
| `/yt-shorts` | The Shorts already inside a long video, with a new first line written for each. |
| `/yt-seo` | The description, the tags that are worth having, and the three queries this should win. |
| `/yt-chapters` | Chapters from a transcript, validated against YouTube's own rules so they render. |
| `/yt-audit` | The whole channel, ending in ONE fix rather than twenty. |

## The six tools

Every one of these runs on a clean Python 3 with no dependencies. They are the reason the skills are
not just prompts.

```bash
python3 skills/yt-script/hookscore.py --hook "one line"        # 5-property hook panel
python3 skills/yt-package/title.py --title "..." --thumb "..." # title + thumbnail linter
python3 skills/yt-edit/deadair.py transcript.srt               # edit decision list
python3 skills/yt-chapters/chapters.py transcript.srt          # validated chapters
python3 skills/yt-retention/retention.py retention.csv         # where they left, and why
python3 skills/yt-viral/swipe.py collected.json --min 2.0      # outliers by own-channel multiple
```

## The fine print

The part most packs leave out.

**The standalone skills do not publish.** They end in a block you copy and a question — ship it, or
change it? The optional Local Creator Studio web app described above is a separate integration and
can publish scheduled videos only after you explicitly authorize it and configure YouTube OAuth,
Gemini, and the video provider.

**`hookscore.py` is a heuristic, not a predictor.** The panel was built and calibrated against 74
real short-form hooks (the first 15 seconds of auto-captions, top-8 and bottom-8 by views across
five channels). It separates deliberately bad hooks from real ones well. It separates a given
creator's hits from their own misses barely at all. A low score is a reason to look again; a high
score is not a promise.

**The formula classifier is about the words, not the result.** When `/yt-viral` says a title used
The Statistic, that is a judgement about the title you can see, not a claim about why the video got
its views.

**`/yt-viral` reads, it does not scrape.** Public listings only. It never logs in as you and never
touches your credentials.

**Tags barely matter** and this pack says so instead of selling you a tag generator.

**Nothing invents a number.** If a skill wants a figure it does not have, it asks you for it or
writes the line without it.

## Licence

MIT. Use it, change it, ship it.
