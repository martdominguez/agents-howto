Act as a senior reviewer after implementation and testing.

Review the database change end to end.

Tasks:
1. Compare the implementation against docs/changes/<ticket>/impact-analysis.md.
2. Identify any missed files, missing tests, risky assumptions, compatibility problems, or migration issues.
3. Produce a concise review report.
4. Suggest follow-up fixes in priority order.

Focus on:
- missing query updates
- broken DTO mappings
- nullability issues
- old-data compatibility
- migration order
- performance regressions
- test gaps
