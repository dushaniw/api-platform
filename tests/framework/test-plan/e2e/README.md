# End-to-end test plan

This directory breaks down the end-to-end scenarios to write in this framework for API Platform. It covers both on-prem (OSS) and cloud deployments. Each feature has its own CSV, with one row per scenario. The plan is a working backlog. Turn rows into Gherkin features under `suites/*/features` following the rules in `../../CLAUDE.md`.

## Layout

| Path | Contents |
|------|----------|
| `index.csv` | One row per feature: file, scope, deployments, topologies, scenario counts by priority, and how many scenarios are already covered |
| `platform/` | platform-api features. Scenarios marked `both` apply to on-prem and cloud; `on-prem` scenarios need on-prem configuration |
| `cloud/` | Features that exist only in the cloud build: environments, managed gateways, pipelines, environment promotion, managed API Portals, logs, insights, trial, org lifecycle |

## Columns

| Column | Meaning |
|--------|---------|
| `id` | Stable scenario id, `<FEATURE>-NN`. Reference it from the Gherkin scenario title or a tag |
| `scenario` | The behaviour under test |
| `deployment` | `on-prem`, `cloud` or `both` |
| `topology` | Deployables the scenario must go through (see below) |
| `type` | `positive`, `negative`, `security`, `isolation` or `resilience` |
| `priority` | `P0` must have before release, `P1` should have, `P2` nice to have |
| `preconditions` | State required before the steps run. A scenario id here means "after that scenario" |
| `steps` | Short outline of the actions |
| `expected_result` | Observable outcome to assert |
| `external_systems` | Mock or real dependencies besides the topology (testbench, mock LLM, IDP, cloud services) |
| `existing_coverage` | Feature file that already covers it (`covered`), `Partial: ...`, `Written but not run: ...`, or `None` |
| `proposed_location` | Suggested feature file and block. `extend existing` means add scenarios to the file named in `existing_coverage` |
| `notes` | Open questions, framework gaps, contracts to confirm |

## Topology codes

| Code | Deployables | Typical block |
|------|-------------|---------------|
| `PA` | platform-api only | any block with `platform-api` |
| `PA+GW` | platform-api and gateway | `platform-api-sqlite` / `-postgres` / `-sqlserver`, `multigateway`, `gateway-restart` |
| `PA+AP` | platform-api and API Portal | `platform-api-gateway-api-portal` (publication), `api-portal-core` |
| `PA+GW+AP` | platform-api, gateway and API Portal | `devportal-webhook`, `platform-api-gateway-api-portal` |
| `GW` | gateway only | not used here; gateway-direct coverage is already broad |

For cloud rows, `PA` is the cloud platform-api, `GW` is a managed gateway and `AP` is a managed API Portal, all in a real cloud environment reached through `cloudconsole`.

## Breakdown by deployment and topology

| Deployment | PA | PA+GW | PA+AP | PA+GW+AP |
|------------|----|-------|-------|----------|
| both | 31 (5 P0) | 43 (17 P0) | 10 (1 P0) | 18 (5 P0) |
| on-prem | 14 (1 P0) | 24 (2 P0) | 1 | 1 |
| cloud | 32 (9 P0) | 26 (7 P0) | 7 (3 P0) | 3 (2 P0) |

## Main coverage gaps this plan targets

- **platform-api and gateway:** deploy coverage is mostly REST. These have no tests yet:
  - rollback / restore
  - gateway offline at deploy time
  - gateway-rejected artifacts
  - reconnect replay
  - platform-api HA
  - control-plane-driven policies for LLM / MCP / GraphQL
- **platform-api and API Portal:** only REST publication goes through platform-api. Monetization sync is simulated, and there is no multi-portal or GraphQL publication.
- **platform-api, gateway and API Portal:** only two three-way flows exist. There is no publish-to-consume journey with a plan limit, revoke, deprecate or retirement, and no browser journey with real (non-seeded) data.
- **Cloud:** the cloud suite has four REST smoke scenarios. Nothing covers managed portals, promotion, gateway lifecycle rules, pipelines, logs or isolation.

## Framework work needed before some rows can be written

| Gap | Blocks rows |
|-----|-------------|
| No IDP component in `core/catalog` | AUTH-08, AUTH-09 |
| `platform-api` component has no replicas option | SYNC-04 |
| Blocks hold only one API Portal per org | PUB-07 |
| No way to stop and start the gateway from a step in PA+GW blocks (restart blocks exist; generic step needed) | DEP-06, SYNC-02 |
| Older gateway release images in PA+GW blocks | DEP-09, COMPAT-*, SEC-05 |
| Cloud suite: a single `apip-platform-api` endpoint, no API Portal or gateway endpoint resolution | C-MAP-*, C-E2E-* |
| Cloud suite: no browser component | C-MAP-02, C-DEP-12, C-INS-*, C-TRIAL-* |
| Cloud suite: token comes from `CLOUD_CONSOLE_ACCESS_TOKEN`; no token acquisition for several users or orgs | C-AUTH-*, isolation rows |
| Cloud steps are bespoke helpers; IT platform-api steps cannot target cloud | Reuse of `both` rows on cloud |
| Cloud fault injection | C-DEP-11, C-ORG-02 |

Rows marked "Confirm contract first" in `notes` need the product behaviour agreed before a test is written.
