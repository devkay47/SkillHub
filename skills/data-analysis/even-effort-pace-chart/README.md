# Even-Effort Pace Chart

Convert GPX/KML routes or recorded FIT tracks into even-effort trail running pace charts, or compare FIT activities with checkpoint plans and segment elapsed times. Use for race pacing, SVG pace tables, or post-race actual-versus-plan comparisons.

Independent implementation inspired by the even-effort pacing approach from 耐力学社.

Source repository: https://github.com/yihouzenmeban/trail-running-pace

## Install

This skill needs its `scripts/` and `references/` folders to run, so install the whole skill directory from the source repository — copying this `SKILL.md` alone will not work.

**npx (recommended):**

```bash
npx skills add yihouzenmeban/trail-running-pace --skill even-effort-pace-chart
```

**Manual:** download the versioned ZIP from [GitHub Releases](https://github.com/yihouzenmeban/trail-running-pace/releases), extract `even-effort-pace-chart/`, and copy the whole directory into your assistant's skills directory.

## Use

Prepare a GPX, KML, or FIT route (or start with the public example in the source repo), then tell your assistant the route path and finish target, for example:

```text
$even-effort-pace-chart
Use race.gpx to generate an even-effort pace chart, target finish 14 hours, output to output/my-race/.
```

Python 3.10+ is required. GPX/KML pace charts use only the standard library; FIT input and activity comparison need the optional dependency described in the skill's configuration reference.
