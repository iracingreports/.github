# iRacing Reports v5

![System Overview](https://github.com/iracingreports/.github/blob/main/profile/system_overview.png?raw=true)

## Architecture Summary
- Collectors harvest live and historical data from the official iRacing API, normalize it, and persist it to PostgreSQL.
- A shared schema and helper library in `common` keeps every service talking the same database dialect and schema version.
- RabbitMQ decouples producers from consumers, letting the collection workers, bot backend, and web stack scale independently.
- Django surfaces the collected data through the public website, admin tooling, and premium workflows (including Patreon integration).
- The Discord bot delivers real-time statistics, announcements, and premium features straight to guilds using the same database records.

## Repository Responsibilities
| Repository | Purpose | Highlights | Interfaces |
| --- | --- | --- | --- |
| `common/` | Shared schema and database utilities | Canonical `database_schema.sql`, connection pooling helpers, ad-hoc scripts | Exported SQL schema; imported Python helpers by other repos |
| `data_collect/` | Data ingestion workers | OAuth 2.1 client, detectors, queuers, collectors, monitoring dashboard, task automation | iRacing REST APIs, PostgreSQL, RabbitMQ |
| `discord_bot/` | Discord-facing service | Slash-command backend, image rendering, Patreon-aware perks, systemd units | RabbitMQ, PostgreSQL, Discord API |
| `website/` | Django application and site | Cookiecutter-Django stack, analytics views, management commands, deployment assets | PostgreSQL, Patreon webhooks, Discord bot integration |
| `discordbot.iracingreports.com/` | Static marketing site | Jekyll-based docs, onboarding guides, hosted on GitHub Pages | Links to production Discord bot endpoint and Django site |

## Shared Infrastructure
- **PostgreSQL 15**: Single source of truth living under the `schema` schema. Role-based access (`data_collector`, `discord_bot`, `django`, `read_only`) keeps services isolated while sharing data.
- **RabbitMQ**: Message broker for results queues (`results_priority`, `results_collection`) and bot command/event traffic. Enables fan-out and back-pressure handling.
- **OAuth 2.1 Credentials**: Required for iRacing API access; managed centrally and consumed by the collectors and service-info sync jobs.
- **Environment Configuration**: All services load credentials from `.env` files and reuse the same variable naming (`DATABASE_USER`, `DATABASE_HOST`, etc.) exported by `common`.

## Public Endpoints
- Primary site: https://iracingreports.com
- Discord bot marketing & docs: https://discordbot.iracingreports.com

## Data Flow
1. **Service Info Sync** keeps reference tables up to date (series, seasons, cars, tracks).
2. **Results Detector** watches for completed subsessions and hands them to the **Results Queuer**, which prioritizes work in RabbitMQ.
3. **Results Collector** workers consume queue messages, call the iRacing API, and write normalized results into the shared database.
4. **Race Announcer** and other downstream tasks publish updates through RabbitMQ, enabling the Discord bot to push live announcements.
5. **Django Website** aggregates and exposes the stored data via web UI, APIs, and admin tools, while the **Discord Bot** serves guild commands using the same records.
6. **Patreon Webhooks** flow into the Django app to update subscription state consumed by both the website and the bot.

## Operating the Stack
- Bring up PostgreSQL and RabbitMQ first; apply the schema from `common/database_schema.sql`.
- Run the sync and collector scripts from `data_collect/` to populate and maintain race data.
- Deploy the Django site (`website/`) and the Discord bot (`discord_bot/`) once data is flowing; both expect the shared database to be seeded.
- Use the monitoring scripts (`monitor_collectors.py`, bot dashboards) to keep an eye on queue depth, worker health, and announcement throughput.

## Deployment Status
- Current infrastructure runs on a Debian 12 host in an Azure VPS, exposed through `dev.iracingreports.com`.
- Production migration from the legacy stack to this v5 architecture is in progress; expect dual-running until cutover completes.
- Host snapshot:
	- VM: `debian` (Generation V2, x64) on Standard E2bds v5 (2 vCPUs, 16 GiB RAM)
	- OS image: `debian/debian-12/12-gen2`
	- Agent: Azure Linux Agent 2.7.3.0, hibernation disabled, SCSI disk controller
	- Networking: public IP `20.3.138.194`, private IP `10.0.0.4`, VNet `debian-vnet/default`
- Legacy production (v4) host snapshot:
	- VM: `ubuntu` (Generation V2, x64) on Standard D2as v4 (2 vCPUs, 8 GiB RAM)
	- OS image: `ubuntu/ubuntu-20.04`
	- Agent: Azure Linux Agent 2.14.0.1, hibernation disabled, SCSI disk controller
	- Networking: public IP `51.143.43.222`, private IP `10.1.0.5`, VNet `iracingreports-vnet/default`

