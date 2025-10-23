name: Profile Metrics

on:
  workflow_dispatch:
  push:
    branches: [ main, master ]
  schedule:
    - cron: "17 2 * * *"  # daily at 02:17 UTC

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      # ---- Overall metrics (contribs, PRs, issues, stars, followers, repos) ----
      - name: Overall metrics
        uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          user: ARIHO256
          template: classic
          base: header, activity, community, repositories, metadata
          config_timezone: Africa/Kampala
          plugin_lines: yes
          plugin_followup: yes
          plugin_repositories: yes
          filename: metrics/overall.svg

      # ---- Languages (compact) ----
      - name: Languages
        uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          user: ARIHO256
          template: classic
          base: ""
          plugin_languages: yes
          plugin_languages_ignored: html, css
          plugin_languages_limit: 8
          plugin_languages_sections: most-used
          plugin_languages_indepth: yes
          plugin_languages_analysis_timeout: 15
          filename: metrics/languages.svg

      # ---- Achievements / trophies ----
      - name: Achievements
        uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          user: ARIHO256
          template: classic
          base: ""
          plugin_achievements: yes
          plugin_achievements_display: detailed
          plugin_achievements_threshold: C
          filename: metrics/achievements.svg

      # ---- Coding habits (streak-like info and commit times) ----
      - name: Habits
        uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          user: ARIHO256
          template: classic
          base: ""
          plugin_habits: yes
          plugin_habits_facts: yes
          plugin_habits_charts: yes
          filename: metrics/habits.svg

      # ---- Isocalendar heatmap (contribution graph-like) ----
      - name: Isocalendar
        uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          user: ARIHO256
          template: classic
          base: ""
          plugin_isocalendar: yes
          plugin_isocalendar_duration: full-year
          filename: metrics/isocalendar.svg

      # ---- Typing banner (static, locally rendered) ----
      - name: Typing banner (SVG)
        run: |
          mkdir -p metrics
          cat > metrics/typing.svg <<'SVG'
          <svg xmlns="http://www.w3.org/2000/svg" width="600" height="40">
            <rect width="100%" height="100%" fill="#0d1117"/>
            <text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle"
                  font-family="Fira Code, monospace" font-size="16" fill="#70A5FD">
              Software Engineer | Full Stack Developer · Django & React · Lifelong Learner
            </text>
          </svg>
          SVG

      - name: Commit generated assets
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add metrics/*.svg || true
          if ! git diff --cached --quiet; then
            git commit -m "chore(metrics): update profile svgs"
            git push
          fi
