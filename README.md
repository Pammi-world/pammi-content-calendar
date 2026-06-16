# Pammi Content Calendar

Google Sheets-backed source of truth for the Pammi Content System. All content agents (content-pammi, visual-content-pammi, social-network-pammi) read and write to this spreadsheet.

## Spreadsheet

| Field | Value |
|-------|-------|
| **Title** | Pammi Content Calendar |
| **ID** | `1rHM6bHIsq8h8a0jG83afWdlZs6wHV25yANutPdkhODk` |
| **URL** | https://docs.google.com/spreadsheets/d/1rHM6bHIsq8h8a0jG83afWdlZs6wHV25yANutPdkhODk/edit |
| **Owner** | pammibot6@gmail.com |

## Schema (4 Tabs)

### 1. Content Packages
Top-level content brief. Each package can spawn multiple platform-specific posts.

| Column | Type | Description |
|--------|------|-------------|
| `package_id` | string | Unique ID, e.g. `PKG-001` |
| `status` | enum | `IDEA` / `BRIEFED` / `IN_PROGRESS` / `READY` / `DONE` / `ARCHIVED` |
| `topic` | string | Short topic title |
| `angle` | string | Point of view / hook |
| `audience` | string | Target audience |
| `source_notes` | text | Source material / research links |
| `formats_requested` | string | Comma-separated: `linkedin`, `blog`, `newsletter` |
| `created_at` | datetime | ISO timestamp |
| `notes` | text | Free-form notes |

### 2. LinkedIn
LinkedIn-specific posts. Each post is a child of one Content Package.

| Column | Type | Description |
|--------|------|-------------|
| `post_id` | string | Unique ID, e.g. `LI-001` |
| `package_id` | string | FK to Content Packages |
| `status` | enum | `DRAFT` / `ASSET_READY` / `READY` / `APPROVED` / `SCHEDULED` / `POSTED` / `FAILED` |
| `topic` | string | Short topic title |
| `dedup_key` | string | Hash to prevent duplicate posts |
| `post_body` | text | The actual post content |
| `scheduled_at` | datetime | When to post (ISO) |
| `approved` | boolean | User approval flag |
| `asset_id` | string | FK to Assets (image for post) |
| `image_file_id` | string | Google Drive file ID of image |
| `image_drive_url` | string | Google Drive URL |
| `published_url` | string | LinkedIn post URL after publish |
| `notes` | text | Free-form notes |

### 3. Assets
Visual and media assets (images, videos, carousels). Linked to packages and posts.

| Column | Type | Description |
|--------|------|-------------|
| `asset_id` | string | Unique ID, e.g. `AST-001` |
| `package_id` | string | FK to Content Packages |
| `status` | enum | `REQUESTED` / `IN_PROGRESS` / `DRAFT` / `APPROVED` / `EXPORTED` |
| `asset_type` | enum | `image` / `carousel` / `video` / `thumbnail` / `diagram` / `chart` / `quote_card` / `mascot` |
| `target_platform` | string | `linkedin`, `twitter`, `blog`, etc. |
| `purpose` | string | What this asset is for |
| `brief` | text | Creative brief for visual-content-pammi |
| `source_file_id` | string | Source file (PSD/Figma/etc) Drive ID |
| `source_drive_url` | string | Source file URL |
| `export_file_id` | string | Final exported file Drive ID |
| `export_drive_url` | string | Final exported file URL |
| `mime_type` | string | `image/png`, `video/mp4`, etc. |
| `notes` | text | Free-form notes |

### 4. Publishing Log
Audit log of every publish attempt. Append-only.

| Column | Type | Description |
|--------|------|-------------|
| `log_id` | string | Unique log ID |
| `post_id` | string | FK to platform post (LinkedIn etc.) |
| `package_id` | string | FK to Content Packages |
| `platform` | string | `linkedin`, `twitter`, etc. |
| `published_at` | datetime | ISO timestamp of publish |
| `status` | enum | `SUCCESS` / `FAILED` / `RETRY_SCHEDULED` |
| `published_url` | string | URL of the published post |
| `error` | text | Error message if failed |
| `retry_at` | datetime | When to retry (if failed) |
| `created_at` | datetime | ISO timestamp of log entry |

## Setup

### Prerequisites
- `composio` CLI installed and authenticated
- Google Sheets tool enabled in Composio
- Account: pammibot6@gmail.com (or another with edit access)

### Creating from Scratch
```bash
# 1. Create the spreadsheet
composio execute GOOGLESHEETS_CREATE_GOOGLE_SHEET1 -d '{"title": "Pammi Content Calendar"}'

# 2. Add the 4 tabs
for tab in "Content Packages" "LinkedIn" "Assets" "Publishing Log"; do
  composio execute GOOGLESHEETS_ADD_SHEET -d "{\"spreadsheet_id\": \"YOUR_ID\", \"title\": \"$tab\"}"
done

# 3. Add headers (see schema.json for column lists)
# Use GOOGLESHEETS_VALUES_UPDATE with the headers as the values
```

## How Agents Use This

| Agent | Reads | Writes |
|-------|-------|--------|
| `pm-pammi` | All tabs | Content Packages (creates new packages) |
| `content-pammi` | Content Packages | LinkedIn (creates posts) |
| `visual-content-pammi` | Content Packages, Assets | Assets (creates visual assets) |
| `social-network-pammi` | LinkedIn, Assets, Publishing Log | LinkedIn, Publishing Log (publishes) |

## Files

- `schema.json` — Full machine-readable schema with column types and descriptions
- `README.md` — This file

## Notes

- This spreadsheet is the **single source of truth** for the content pipeline
- Append-only for Publishing Log (never delete entries, add corrections instead)
- Status fields are controlled vocabularies — agents should validate before writing
- All IDs follow the pattern `XXX-NNN` (zero-padded to 3 digits)
