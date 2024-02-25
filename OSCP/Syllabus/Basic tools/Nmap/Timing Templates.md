---
tags:
  - web-guide
---
https://nmap.org/book/performance-timing-templates.html
```embed
title: "Timing Templates (-T) | Nmap Network Scanning"
image: "https://nmap.org/images/sitelogo.png"
description: "While the fine-grained timing controls discussed in the previous section are powerful and effective, some people find them confusing. Moreover, choosing the appropriate values can sometimes take more time than the scan you are trying to optimize. So Nmap offers a simpler approach, with six timing templates. You can specify them with the -T option and their number (0–5) or their name. The template names are paranoid (0), sneaky (1), polite (2), normal (3), aggressive (4), and insane (5). The first two are for IDS evasion. Polite mode slows down the scan to use less bandwidth and target machine resources. Normal mode is the default and so -T3 does nothing. Aggressive mode speeds scans up by making the assumption that you are on a reasonably fast and reliable network. Finally insane mode assumes that you are on an extraordinarily fast network or are willing to sacrifice some accuracy for speed."
url: "https://nmap.org/book/performance-timing-templates.html"
```

You can tell nmap to speed up or slow down the scan according to how stealthy you want to be.

"If you are on a decent broadband or ethernet connection, I would recommend always using `-T4`. Some people love `-T5` though it is too aggressive for my taste. People sometimes specify `-T2` because they think it is less likely to crash hosts or because they consider themselves to be polite in general. They often don't realize just how slow `-T polite` really is. They scan may take ten times longer than a default scan. Machine crashes and bandwidth problems are rare with the default timing options (`-T3`) and so I normally recommend that for cautious scanners. Omitting version detection is far more effective than playing with timing values for reducing these problems."

Table 6.3. Timing templates and their effects

|                                                   | T0                                        | T1         | T2         | T3         | T4         | T5         |
| ------------------------------------------------- | ----------------------------------------- | ---------- | ---------- | ---------- | ---------- | ---------- |
| Name                                              | Paranoid                                  | Sneaky     | Polite     | Normal     | Aggressive | Insane     |
| `min-rtt-timeout`                                 | 100 ms                                    | 100 ms     | 100 ms     | 100 ms     | 100 ms     | 50 ms      |
| `max-rtt-timeout`                                 | 5 minutes                                 | 15 seconds | 10 seconds | 10 seconds | 1250 ms    | 300 ms     |
| `initial-rtt-timeout`                             | 5 minutes                                 | 15 seconds | 1 second   | 1 second   | 500 ms     | 250 ms     |
| `max-retries`                                     | 10                                        | 10         | 10         | 10         | 6          | 2          |
| Initial (and minimum) scan delay (`--scan-delay`) | 5 minutes                                 | 15 seconds | 400 ms     | 0          | 0          | 0          |
| Maximum TCP scan delay                            | 5 minutes                                 | 15,000     | 1 second   | 1 second   | 10 ms      | 5 ms       |
| Maximum UDP scan delay                            | 5 minutes                                 | 15 seconds | 1 second   | 1 second   | 1 second   | 1 second   |
| `host-timeout`                                    | 0                                         | 0          | 0          | 0          | 0          | 15 minutes |
| `script-timeout`                                  | 0                                         | 0          | 0          | 0          | 0          | 10 minutes |
| `min-parallelism`                                 | Dynamic, not affected by timing templates |            |            |            |            |            |
| `max-parallelism`                                 | 1                                         | 1          | 1          | Dynamic    | Dynamic    | Dynamic    |
