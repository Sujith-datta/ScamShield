🛡️ ScamShield: Check before you trust

ScamShield is a web app that helps people spot scams in emails, text messages and links. Paste a message or URL and get a clear, explainable assessment: Likely safe, Needs caution or Likely scam, with a confidence indicator, the exact suspicious phrases highlighted, plain-language reasons, and practical next steps.

It was built for an AI hackathon as a prototype. It never opens submitted links, never claims access it doesn't have, and clearly labels what its checks can and can't do.

Heads-up: ScamShield gives guidance, not guarantees. The built-in checker is rule-based and can miss scams or flag real messages. Always verify through an official website or a phone number you already trust.

Features
Area	What you get
Scam checker	Paste an email, SMS or URL, or pick a sample. Results appear with a verdict, confidence, risk score and numbered warning signs highlighted in the text. Verdicts use icons and text, never color alone.
Message analysis	Detects urgency, threats, requests for passwords/OTPs/money, impersonated brands, too-good-to-be-true offers, secrecy and generic greetings.
Link analysis	Flags misleading brand domains, shortened links, IP-address hosts, look-alike (punycode) characters, @ tricks, many subdomains, odd endings and missing HTTPS. Links are analyzed as text only, never visited.
Next steps	Tailored advice for each verdict: verify officially, call a known number, report it.
Scan history	Item, result and date, stored in your browser. Reopen past results or clear everything.
SMS Scams guide	The six most common text-message scams, red flags, what to do, and example texts you can test with one click.
Inbox scan (optional)	Connect Gmail with read-only access and scan recent inbox emails. Needs your own Google credentials.
AI second opinion (optional)	An Ask the AI button sends the checked text to Anthropic or Groq for a second verdict and reasons. Off by default, only runs when clicked.
Settings and privacy	Honest status of every integration, and a plain description of where data goes.
Accessible and responsive	Keyboard friendly, visible focus states, screen-reader announcements, works on phones.

The app works fully with no keys at all. Gmail and AI are opt-in extras.

Quick start

Requirements: Node.js 18 or newer (npm comes with it). Check with node -v and npm -v.

bash
# 1. Install dependencies
npm install

# 2. Create your config file
cp .env.example .env        # Windows (cmd): copy .env.example .env

# 3. Start the server
npm start

Open http://localhost:3000. Always use this address, not the HTML file directly, or Gmail and AI will not work.

You can use the checker right away. Add keys to .env later for Gmail or AI, then restart.

npm commands
Command	What it does
npm install	Downloads the libraries listed in package.json (once, or when it changes).
npm start	Starts the server at http://localhost:3000.
npm run dev	Starts the server and restarts it when files change (needs Node 18.11+).

Stop the server with Ctrl + C. After editing .env, stop and start it again, because it is only read at startup.

Configuration (.env)

Copy .env.example to .env and fill in what you need. Use no quotes and no spaces around =. Never commit .env.

Variable	Needed for	Notes
PORT	Optional	Default 3000.
BASE_URL	Gmail	Default http://localhost:3000. Must match your Google redirect URI base.
SESSION_SECRET	Gmail	Long random string. Generate: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET	Gmail	From Google Cloud (see below).
ANTHROPIC_API_KEY, ANTHROPIC_MODEL	AI (option A)	Model default claude-haiku-5-5.
GROQ_API_KEY, GROQ_MODEL	AI (option B)	Model default llama-3.3-70b-versatile. Change it if Groq renames or retires models.
AI_PROVIDER	Optional	anthropic or groq. If both keys are set, Anthropic is used unless this says otherwise.
NODE_ENV	Optional	Set to production when deployed to hide provider error details from the page.

At startup the terminal prints whether AI is on, for example AI second opinion: Groq (llama-3.3-70b-versatile).

Optional: Gmail inbox scan (read-only)
In Google Cloud Console, create a project and enable the Gmail API.
Set up the OAuth consent screen: user type External, publishing status Testing, and add your Gmail address under Test users.
Add the scopes gmail.readonly and userinfo.email.
Create an OAuth client ID of type Web application with this redirect URI: http://localhost:3000/auth/google/callback
Put the client ID and secret in .env, set SESSION_SECRET, and restart.
In the app, go to Settings → Connect Gmail, then open Inbox. Tick the permission to view your email on Google's screen, and click through the "unverified app" warning (expected for test apps).

Access is read-only: ScamShield cannot send, delete or change email. The access token lives in server memory for the session, emails are fetched on request and not stored, and Disconnect revokes the token. In Testing mode only listed test users can sign in. Public use needs Google's app verification (Gmail read access is a restricted scope).

Optional: AI second opinion
Get a key from Anthropic or Groq. Both may have limits or costs, so check their current terms.
Add ANTHROPIC_API_KEY=... or GROQ_API_KEY=... to .env and restart.
Check a message and click Ask the AI.

The text is sent to the provider only when you click the button. The key stays on the server, the endpoint is rate-limited (15 per minute per IP) and needs a custom header. The model is told to treat the text as untrusted data and ignore any instructions inside it, but AI can still be wrong or manipulated, so it is shown as a second opinion, not a verdict.

How the analyzer works

All logic sits in one Analyzer object in public/index.html. It returns { level, score, confidence, signals, ranges, kind, text }, so a real backend or model can replace Analyzer.analyze() without touching the interface.

Input with no spaces that looks like a web address is treated as a link, anything else as a message.
Messages are tested against eight categories of patterns, and links found inside messages are also checked. Links are checked by their text only.
Each matched sign has a weight. They are added up (capped at 100), with a small bonus when three or more signs appear in a message.
Score 55+ is Likely scam, 22 to 54 is Needs caution, below 22 is Likely safe.
Confidence is a rough heuristic, labeled as demo confidence, not a statistical probability.
Privacy and data
Rule-based checks run in your browser. Pasted content is not sent to a server unless you click Ask the AI.
Scan history is stored in this browser's local storage, unencrypted, until you clear it on the History page or clear browser data.
If you use Gmail, your own server fetches read-only copies of recent emails using your permission. Nothing is stored server-side.
If you use AI, the checked text goes to the provider you configured, and that provider's policies apply. Read them before using real people's messages.
No claims of end-to-end encryption or automatic deletion are made, because the app doesn't do them. If you deploy this for others, update the privacy text to match what you actually do.
Project structure
scamshield/
├── server.js        Express server: Gmail OAuth, email fetch, AI proxy
├── public/
│   └── index.html   The whole front end (UI, styles, analyzer)
├── package.json     Dependencies and npm scripts
├── .env.example     Template for your config (copy to .env)
└── README.md

Stack: Node.js, Express, express-session, googleapis, dotenv, plain HTML/CSS/JavaScript on the front end (no build step).

API endpoints
Method and path	Purpose
GET /api/status	Reports whether Gmail is configured/connected and whether AI is on.
GET /auth/google and /auth/google/callback	Google sign-in (CSRF state check, read-only scopes).
GET /api/emails?max=15	Recent inbox emails (requires a connected session).
POST /auth/logout	Revokes the Google token and ends the session.
POST /api/ai-check	AI second opinion (rate-limited, needs X-Requested-With: ScamShield).
Troubleshooting
Problem	Fix
'npm' is not recognized	Install Node.js LTS, then open a new terminal.
Settings says "Google not set up"	.env is missing, misnamed (.env.txt) or empty, or the server wasn't restarted.
Error 400: redirect_uri_mismatch	The redirect URI in Google Cloud must exactly match http://localhost:3000/auth/google/callback.
Error 403: access_denied	Add your Gmail address under Test users on the consent screen.
"Couldn't load emails"	Enable the Gmail API, add the gmail.readonly scope, remove ScamShield at myaccount.google.com/permissions, reconnect and tick the email permission.
Ask the AI shows an error	The message ends with the provider's reason, such as (Groq 401: Invalid API Key). 401 means a bad key, 404 usually means an outdated model name.
Port 3000 in use	Set PORT and BASE_URL in .env and update the Google redirect URI to match.
Limitations
The built-in checker is rule-based, not AI, and can be wrong in both directions.
No reputation or domain-age lookups yet, and no link is ever opened.
A web page cannot read your text messages. SMS checking is paste-only.
Prototype session handling uses an in-memory store (fine for a demo, not for production).
Roadmap (coming soon, not built yet)
 AI-assisted analysis as a first-class option, not only a second opinion
 Reputation checks: domain age and known-bad link lookups
 SMS checking through a forwarding number (needs a messaging provider)
 Outlook support and a browser extension
 Google app verification for public Gmail use
 Production hardening: persistent session store, HTTPS deployment guide, broader rate limiting
 Automated tests for the analyzer
License

No license has been chosen yet. Add one (for example MIT) before publishing publicly.
