# sgy — Schoology from the terminal

Your grades, classes and due dates, without opening a browser.

```console
$ sgy classes
p1   Biology - 3110            SmithA p1 T1
p2   Comp Prog Java - 2370     JonesB p2 T1
p3   PE 9 - 2510               LeeC p3 T1
p4   Spanish 2 - 4320          GarciaD p4 T1
p5   Algebra 2/Trig - 2320     PatelE p5 T1
p6   Lit/Writ - 1010           BrownF p6 T1

$ sgy grades -v
Algebra 2/Trig - 2320: PatelE p5 T1   B+
    Homework (15%)            94%
    Participation (5%)       100%
    Quizzes/Tests (65%)        84%
    Semester Final (15%)        —

$ sgy due 7
2026-09-04 23:00  Lab writeup: enzyme activity      Biology - 3110
2026-09-05 08:00  Vocab quiz 3                      Spanish 2 - 4320
```

<sub>Sample output — names and grades are made up.</sub>

## Setup

You need [uv](https://docs.astral.sh/uv/). If you don't have it:

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Then:

```sh
git clone https://github.com/SaarthurR/schoology-cli.git
cd schoology-cli
./sgy setup
```

`setup` asks for your school's Schoology address, downloads Chromium, links `sgy`
onto your `PATH`, and opens a window for you to sign in. That's the whole install.

You can skip the prompt by passing the address:

```sh
./sgy setup fuhsd.schoology.com
```

## Commands

| Command | What it does |
|---|---|
| `sgy classes` | Your classes, in period order |
| `sgy grades` | One line per class |
| `sgy grades -v` | Plus category weights and assignments |
| `sgy due [days]` | What's due; defaults to 14 |
| `sgy json` | Structured data — pipe it into `jq` |
| `sgy login` | Re-run when your session expires |

Everything else you can see in a browser:

| Command | What it does |
|---|---|
| `sgy messages` | Your inbox |
| `sgy feed` | Home feed / recent updates |
| `sgy info` | Your profile |
| `sgy materials [class]` | A course's materials; no argument lists your classes |
| `sgy page <path>` | **Any** Schoology page, as readable text |

`sgy page` is the escape hatch. Anything in Schoology that has a URL, it will
print — `sgy page /messages/sent`, `sgy page /assignment/7422452694`. JSON
endpoints come back as JSON; everything else is rendered to plain text.

Add `--all` to `classes` or `grades` to include non-graded enrolments — school
hubs, counseling pages, advisory, onboarding courses.

### Why `page` isn't a pile of parsers

Schoology server-renders most of itself, and the markup differs per screen and
per district. Bespoke selectors for each page would be guesswork that breaks on
the next template change. `page` strips scripts and navigation and prints what
the page actually says, so it works on screens this tool has never seen. The
trade is that a little interface furniture ("All Materials", notification
settings) comes through with it.

### Reading the output

A dash (`—`) means **no grade posted yet**. It is not a zero.

## Why it works this way

Schoology has an official REST API, but it needs a consumer key that most
districts don't hand out to students. The session cookie is `httpOnly`, so
there's also nothing to copy out of devtools into a config file.

So `sgy` keeps its own logged-in Chrome profile and calls the same internal JSON
endpoints the Schoology web app calls:

| Data | Endpoint |
|---|---|
| Courses | `/iapi2/site-navigation/courses` |
| Grades | `/grades/grades?id=<uid>` |
| Calendar | `/calendar/<uid>/<yyyy-mm>?ajax=1&start=<ts>&end=<ts>` |

Your user id is discovered at runtime, so there's nothing to configure but the
district address.

**These endpoints are undocumented.** Schoology can change them at any time and
this will break. That's the trade for not having an API key.

## Privacy

- `sgy` is **read-only**. It never writes to your Schoology account.
- It holds a **full session**, not a scoped key, so it can reach anything your
  account can reach — that's what makes `sgy page` work. It cannot reach
  anything you can't: no other students' grades, no teacher gradebooks, no
  district records.
- Your session lives in `~/.config/schoology-cli/state.json`, mode `600`. Anyone
  who gets that file is logged in as you. Don't commit it or paste it anywhere.
- Nothing is sent anywhere except your own school's Schoology server. There is no
  telemetry and no third-party service.

## Troubleshooting

**`Session expired. Run: sgy login`** — normal. Schoology sessions don't last
forever. Run `sgy login` and sign in again.

**A command returns nothing, or crashes** — Schoology probably changed an
endpoint. Open an issue with the output of `sgy json`.

**Your district doesn't use period markers** — the class filter looks for `p5`
style markers in section names. If yours don't have them, nothing is filtered and
you'll see every enrolment. That's the intended fallback.

## Licence

MIT.
