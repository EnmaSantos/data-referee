# Free deployment: Vercel + Neon

The API and a read-only web dashboard run together on Vercel. PostgreSQL runs on
Neon's Free plan. The existing Streamlit dashboard remains available locally.
No Render service is needed for this deployment.

## Deployment

1. Create a Neon Free project in AWS US East (N. Virginia), with a PostgreSQL
   database named `live_music`.
2. Import this GitHub repository into Vercel using the FastAPI framework preset.
   Use the Hobby plan for this personal portfolio project.
3. In Vercel's production environment settings, add encrypted variables:
   - `DATABASE_URL`: the Neon pooled connection string (hostname contains `-pooler`).
   - `DATABASE_URL_UNPOOLED`: the direct connection string for schema initialization.
4. Deploy the reviewed commit. The build runs `python -m app.initialize`,
   applies the version-controlled SQL files, and loads the bundled sample CSV only
   when the database has no events. Runtime requests do not run migrations.
5. Check `/health`, `/data-quality-report`, all report endpoints, and the dashboard
   at `/`. Confirm the database is reachable and the reports contain records.
6. Replace the README's live-demo links with the verified production URL and
   update the repository's About website.

Keep credentials in provider settings or an ignored local environment file.
Do not commit them. Preview deployments need a separate Neon development branch
and their own environment variables; this setup configures production only.

## Existing data

The bundled CSV is sample data, not the historical 2,799-event Ticketmaster
dataset discussed in the findings reports. Export the old database, if it remains
accessible, before deleting any Render resources. Restore that backup using a
direct Neon connection, or re-run Ticketmaster ingestion with your API key.
The migration does not delete or overwrite the old Render database.

## Local development

```bash
python -m pip install -e ".[dev,dashboard]"
docker compose up --build
```

Vercel excludes Streamlit from the API's base dependencies to keep the function
bundle smaller. Docker installs the dashboard extra so both local services work.
The hosted dashboard calls the existing API endpoints and cannot reset data.

Free plans have usage limits and may pause or throttle services when limits are
reached. Review current limits at [Vercel Hobby](https://vercel.com/docs/plans/hobby)
and [Neon pricing](https://neon.com/pricing). An idle database may need a cold start.
