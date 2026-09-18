# AllDebrid Discord Download Notifier

Automatically sends a Discord notification when a new download finishes processing in **AllDebrid**.

Designed for setups where **Seanime** handles anime downloads, AllDebrid processes the magnets, and you want a Discord notification without keeping a PC or NAS running.

## How It Works

```text
Seanime
   │
   ▼
AllDebrid
   │
   │  Magnet finishes
   ▼
GitHub Actions
   │
   │  Checks every ~5 minutes
   ▼
Discord Webhook
   │
   ▼
📢 Role Mention + Download Complete Embed
```

The workflow uses the AllDebrid API to check for magnets in the `Ready` state. When it finds a magnet that has not been notified before, it sends a Discord webhook notification.

The workflow runs entirely on GitHub's hosted runners, so no computer or NAS needs to remain powered on.

## Features

* 🔄 Automatically checks AllDebrid for completed downloads
* ⏱️ Runs approximately every 5 minutes
* 📢 Mentions a specific Discord role
* 📦 Shows the completed filename
* 💾 Shows file size
* 🖥️ Detects common video resolutions
* 📝 Cleans up the anime title for the Discord embed
* 🧠 Remembers previously notified downloads
* 🔐 Uses GitHub Secrets for API credentials
* 💻 No software installation required
* 📴 No PC needs to remain powered on
* ▶️ Can also be triggered manually from GitHub Actions

## Requirements

You need:

* A GitHub repository
* GitHub Actions enabled
* An AllDebrid account
* An AllDebrid API key
* A Discord server
* A Discord webhook
* A Discord role to mention

## Repository Structure

```text
.
├── .github/
│   └── workflows/
│       └── seanime-discord.yml
│
├── .alldebrid-notified.json
│
└── README.md
```

### `seanime-discord.yml`

This is the GitHub Actions workflow that:

1. Runs on a schedule.
2. Connects to AllDebrid.
3. Finds ready magnets.
4. Checks whether they have already been notified.
5. Sends a Discord notification for newly completed downloads.
6. Saves the notified magnet IDs.
7. Commits the updated state back to the repository.

### `.alldebrid-notified.json`

This file stores the AllDebrid magnet IDs that have already generated a Discord notification.

Example:

```json
[
  "757703954",
  "757809908"
]
```

This prevents the same download from generating repeated Discord messages.

## GitHub Secrets

The workflow requires two repository secrets.

Go to:

**Repository → Settings → Secrets and variables → Actions**

Add:

### `ALLDEBRID_API_KEY`

Your AllDebrid API key.

### `DISCORD_WEBHOOK_URL`

Your Discord webhook URL.

These values should **not** be placed directly inside the workflow file.

The workflow accesses them through:

```yaml
${{ secrets.ALLDEBRID_API_KEY }}
```

and:

```yaml
${{ secrets.DISCORD_WEBHOOK_URL }}
```

## Discord Role Mention

The workflow currently mentions this Discord role:

```text
1324095223655043092
```

The role is included using:

```json
{
  "content": "<@&1324095223655043092>",
  "allowed_mentions": {
    "roles": ["1324095223655043092"]
  }
}
```

This produces a real Discord role mention rather than displaying the role ID as plain text.

To use a different role, replace the role ID in both places.

## Notification Example

A completed download produces a notification similar to:

```text
@Anime Downloads

Download Complete

Frieren Episode 1

Filename:
Frieren - 01 [1080p].mkv

File size:
1.42 GB

Resolution:
1080p
```

The exact filename, size, title, and resolution depend on the completed AllDebrid download.

## Schedule

The workflow uses:

```yaml
on:
  schedule:
    - cron: "*/5 * * * *"
```

This means GitHub schedules the workflow to run every 5 minutes.

GitHub's current documentation states that scheduled workflows have a minimum interval of 5 minutes. Scheduled runs can occasionally be delayed during periods of high GitHub Actions load.

The workflow also supports manual execution:

```yaml
workflow_dispatch:
```

You can manually start it from:

**GitHub → Actions → AllDebrid Download Notifications → Run workflow**

Manual execution is useful for testing.

## First Run

The first time the workflow runs, it creates the notification state.

Existing ready downloads are added to:

```text
.alldebrid-notified.json
```

They are **not** sent to Discord.

This prevents the first run from flooding Discord with downloads that were already completed before the notifier was installed.

Only subsequently detected downloads generate notifications.

## Troubleshooting

### Workflow runs but Discord does not receive a message

Open:

**GitHub → Actions → AllDebrid Download Notifications**

Open the latest run and check the output.

You should see something similar to:

```text
Found 1 ready AllDebrid magnet(s).
Newly completed download(s): 1
Discord notification sent for: example.mkv
Done.
```

If you see:

```text
Newly completed download(s): 0
```

the magnet is probably already listed in:

```text
.alldebrid-notified.json
```

### The same download is being notified repeatedly

Check `.alldebrid-notified.json`.

The magnet ID should be present after a successful Discord notification.

The workflow only adds the ID after Discord successfully accepts the notification.

### GitHub Actions is not running

Make sure:

* The workflow exists in `.github/workflows/`
* The workflow is committed to the repository's default branch
* GitHub Actions is enabled
* The schedule is still set to `*/5 * * * *`

Scheduled workflows run from the default branch. GitHub also notes that scheduled runs can be delayed during high-load periods.

### Discord role is not being pinged

Verify:

1. The role ID is correct.
2. The role is still present in the Discord server.
3. The webhook's channel permissions allow the appropriate mentions.
4. The role ID appears in both `content` and `allowed_mentions`.

## Security

### Never commit secrets

Do **not** put your AllDebrid API key or Discord webhook URL directly into the repository.

Use GitHub Secrets instead.

If a secret is accidentally committed to a public repository, immediately rotate/revoke it.

### `.alldebrid-notified.json` is safe to commit

This file contains only AllDebrid magnet IDs used to track which downloads have already generated notifications.

It does not contain your API key or Discord webhook URL.

## Changing the Discord Role

To change the role being mentioned, edit:

```python
"content": "<@&1324095223655043092>",
```

and:

```python
"roles": ["1324095223655043092"]
```

Replace `1324095223655043092` with the new role ID.

Both values should use the same role ID.

## Changing the Schedule

The current schedule is:

```yaml
cron: "*/5 * * * *"
```

This is the minimum supported GitHub Actions schedule interval.

Do not change it to:

```yaml
*/1 * * * *
```

GitHub Actions does not support scheduled workflows more frequently than once every 5 minutes.

## Why GitHub Actions?

This setup was designed around a few specific requirements:

* No PC needs to stay powered on.
* AllDebrid remains responsible for processing the download.
* GitHub provides the cloud execution environment.
* Discord provides the notification.
* The repository stores the small amount of state needed to prevent duplicate notifications.

GitHub Actions workflows are defined in `.github/workflows/` and can run on scheduled triggers.

## Data Flow & Privacy

The workflow communicates with:

**AllDebrid**

Used to check the status of your magnets.

**Discord**

Used to send the download notification through your webhook.

**GitHub Actions**

Runs the notifier script and stores the notification state in the repository.

Your AllDebrid API key and Discord webhook URL are provided to the workflow through GitHub Secrets.

## Credits

Built for a Seanime + AllDebrid setup using GitHub Actions and Discord webhooks.

---

### Status

**Working setup:** ✅

**Automatic checking:** ✅

**Discord notifications:** ✅

**Role mentions:** ✅

**PC required:** ❌

**NAS software required:** ❌
