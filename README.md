# Mini-Project 5: Audit an MCP Server

**IT7075: Applied AI for Cybersecurity**

This project audits a local Model Context Protocol (MCP) server. It compares a loose CVE lookup with a version-aware lookup, then tests what happens when the knowledge base is missing an entry or uses a heading that is too vague. The lab uses simulated scan results and a local Markdown knowledge base. It does not scan live hosts or require an LLM or API key.

## Run the lab

If you already have a working `.venv`, use it. For a fresh checkout, open Terminal in the project folder and run:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install mcp ipykernel
```

Open [MiniProject_MCP_Starter.ipynb](MiniProject_MCP_Starter.ipynb) in VS Code, select `.venv` as the notebook kernel, and run the setup cell first. The notebook launches `project_mcp_server.py` as a subprocess over standard input and output. You do not need to start the server separately.

The final knowledge base includes the entries added during the project. To reproduce the original baseline, use a separate copy of the knowledge base from before those entries were added. Running the baseline cells against the expanded knowledge base will not reproduce the original scores.

## Project files

| File | Purpose |
| --- | --- |
| [MiniProject_MCP_Starter.ipynb](MiniProject_MCP_Starter.ipynb) | Notebook for tool discovery, resource reads, simulated scans, and scoring |
| [project_mcp_server.py](project_mcp_server.py) | MCP server with simulated hosts, CVE lookup tools, and `summarize_host` |
| [project_cve_kb.md](project_cve_kb.md) | Local knowledge base, exposed as the `kb://cve` resource |
| [expected_cves.json](expected_cves.json) | Starter lab answer key |
| [results_mine.csv](results_mine.csv) | Final results for the custom host |
| [results_mine_before.csv](results_mine_before.csv) | Custom-host results before adding the Jetty entry |
| [results_mine_vague.csv](results_mine_vague.csv) | Custom-host sweep with the vague Jetty heading |
| [SOURCES.md](SOURCES.md) | References for the added knowledge-base entries |

## Starter lab results

The starter sweep scored 11 service rows across four simulated hosts.

| Knowledge base | Lookup tool | Grounded | Blind spot | Wrong version | True gap |
| --- | --- | ---: | ---: | ---: | ---: |
| Baseline | `lookup_cve` | 6 | 2 | 1 | 2 |
| Baseline | `lookup_cve_v2` | 9 | 0 | 0 | 2 |
| Expanded | `lookup_cve` | 8 | 2 | 1 | 0 |
| Expanded | `lookup_cve_v2` | 11 | 0 | 0 | 0 |

Better matching fixed the Telnet and Redis blind spots and stopped the unrelated Apache 2.4.49 advisory from appearing for the Apache 2.2.8 banner. Adding entries for nginx and PostgreSQL closed two gaps that matching alone could not fix.

## Custom host and vague-heading test

The custom host, `10.0.0.30`, had three simulated services: ProFTPD 1.3.5, OpenSSH 7.2p2, and Jetty 9.4.12. The `summarize_host` tool reported three open services on ports 21, 22, and 8080. Jetty went from a true gap to a grounded result for both lookup tools after its entry was added. Both tools scored 3 out of 3 grounded on the final sweep.

Changing the heading from `# Jetty 9.4.12` to `# Jetty` did not change the regular sweep scores. A separate test uncovered the problem: `lookup_cve_v2` returned the 9.4.12 advisory for the made-up banner `Jetty 99.0.0` when the heading was vague, but returned no match when the heading included the version. The specific heading was restored after the test.

## Takeaway

“No known CVEs” means the lookup found no match in this local knowledge base, not that the service is safe. A returned advisory is also only a candidate until its applicability is verified. A client should distinguish tool errors from no-match results and verify the product and version before acting.
