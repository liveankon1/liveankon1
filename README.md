name: Update README Time

on:
  schedule:
    - cron: "*/5 * * * *" # every 5 minutes
  workflow_dispatch:

permissions:
  contents: write

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Update time in README
        run: |
          node - <<'NODE'
          const fs = require('fs');

          // ✅ Change timezone here if you want:
          const tz = 'America/Chicago';

          const now = new Date();
          const date = new Intl.DateTimeFormat('en-US', {
            timeZone: tz,
            weekday: 'long',
            year: 'numeric',
            month: 'short',
            day: '2-digit'
          }).format(now);

          const time = new Intl.DateTimeFormat('en-US', {
            timeZone: tz,
            hour: '2-digit',
            minute: '2-digit',
            second: '2-digit',
            hour12: true
          }).format(now);

          const line = `**${date} — ${time}** (${tz})`;

          const readmePath = 'README.md';
          let md = fs.readFileSync(readmePath, 'utf8');

          md = md.replace(
            /<!--TIME_START-->[\s\S]*<!--TIME_END-->/,
            `<!--TIME_START-->${line}<!--TIME_END-->`
          );

          fs.writeFileSync(readmePath, md);
          console.log('Updated:', line);
          NODE

      - name: Commit changes
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add README.md
          git commit -m "Update README time" || exit 0
          git push
