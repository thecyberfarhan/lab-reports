# Lab reports

> Pentest-style reports written against intentionally vulnerable training apps, in my own
> isolated lab.

![Reports](https://img.shields.io/badge/-Pentest%20Reports-B4232C?style=flat-square)
![Scope](https://img.shields.io/badge/scope-own%20lab%20only-444?style=flat-square)

## At a glance

| | |
| --- | --- |
| **Kind** | Field note |
| **Status** | Active |
| **License** | MIT |
| **Scope** | Own lab only |
| **Skill area** | Reporting |

## Why

A finding is only half the job. The other half is writing it up the way a client actually reads
it: what was tested, what was found, how bad it is, and how to fix it. These reports follow a real
engagement's structure against apps built to be broken, so the structure can be shown without
touching anyone else's system.

## What this shows

- Reporting: scope and rules of engagement, a summary a non-technical reader can act on, findings
  with evidence and risk ratings, and fixes in priority order — not just "I found a bug."
- Using my own tools as part of a real engagement, not just running them standalone: the DVWA
  report's appendix is the actual [reconkit](../reconkit) and [apicheck](../apicheck) output from
  that test.
- Confirming findings by hand, not trusting a scanner's output: the report says explicitly that
  the vulnerabilities were confirmed with minimal, safe proof-of-concept requests.

## What's inside

| Target | App | Status |
| --- | --- | --- |
| [DVWA (low security)](dvwa-low/report.md) | [Damn Vulnerable Web Application](https://github.com/digininja/DVWA) | Done |
| WebGoat | [OWASP WebGoat](https://owasp.org/www-project-webgoat/) | Planned |

Each report follows the same shape: scope and rules of engagement, a summary, findings with
evidence and a risk rating, overall risk, and recommendations in priority order. The DVWA report
additionally appendices the raw output of [reconkit](../reconkit) and [apicheck](../apicheck) run
against the same target, as evidence of the recon phase that led to the findings.

## Method

Black-box recon first (unauthenticated), then authenticated testing with the app's default
low-privilege demo account. No automated exploitation tools were used against the live app beyond
the two tools in this portfolio; every vulnerability below was confirmed by hand with minimal,
safe proof-of-concept requests.

## Limitations

- Every target here is a training app built to be broken; findings and fixes reflect that, not a
  production system's full threat model.
- Only one report (DVWA, low security setting) is complete; WebGoat is planned but not started.

## Responsible use

All testing here is against instances run by me, on a network only I can reach. Nothing in this
repo targets a system I don't own or don't have permission to test.

## License

MIT. See [LICENSE](LICENSE).
