+++
title = "EzProject: Discord Bot for Assignment and Task Management"
date = "2024-08-22"
summary = "EzProject is a modular Python Discord bot that streamlines assignment, task, and deadline management for collaborative teams."
tags = ["Python", "Discord", "Automation", "Bot", "Task Management"]
+++

# EzProject: Your Discord Buddy for Assignments & Tasks

Hey there! 👋

Ever felt like keeping track of all your assignments and group tasks on Discord is a mess? EzProject is here to help! It's a Python-powered Discord bot that makes managing assignments, tasks, and deadlines super easy—right inside your server.

---

## 🚀 What Can EzProject Do?

- **Add & List Assignments:** Quickly add new assignments and see what's due.
- **Task Tracking:** Log your tasks, update progress, and check things off as you go.
- **Deadline Reminders:** Never miss a due date again!
- **Role Management:** Assign roles to your friends or teammates for better collab.
- **Handy Commands:** Need help or info? There are commands for that too.
- **Slash Commands:** Use modern Discord slash commands for a smooth experience.
- **Modular Design:** Each feature is its own "cog" (module), so it's easy to add more cool stuff later.

---

##  How's It Built?

Here's a peek at the folder structure:

```
EzProject/
├── bot/
│   ├── main.py                # Bot launcher
│   ├── slash_commands.py      # Slash command definitions
│   ├── commands.py            # Old-school command support
│   └── cogs/                  # All the feature modules
│       ├── assignment_management.py
│       ├── deadline_management.py
│       ├── task_management.py
│       ├── role_management.py
│       └── utility_commands.py
├── Datamodule/db.py           # Where your data lives
├── config.py                  # Bot token & config
├── requirements.txt           # Stuff you need to install
```

- **Bot loads up all the cogs** so each feature works on its own.
- **Supports both old and new commands** (but slash commands are the way to go!).
- **Stores your data locally**—no fancy database needed to get started.
- **Config is simple**—just set your token and you're good.

---

##  How Do I Use It?

**Add an assignment:**
```python
@app_commands.command(name="add_assignment", description="Add a new assignment.")
async def add_assignment(interaction: discord.Interaction, name: str, due_date: str):
    ...
```

**Typical flow:**
1. You type a command (like `/add_assignment`).
2. The bot does its thing and saves your info.
3. You get a friendly reply ("Assignment added!").
4. Check or update stuff whenever you want.

---

##  How to Deploy on Railway

Want to run EzProject 24/7? Railway makes it easy:

1. Fork or clone the repo to your GitHub.
2. Sign in at [Railway](https://railway.app/) and hit "New Project".
3. Pick "Deploy from GitHub Repo" and connect your EzProject repo.
4. Set up your environment variables (like your Discord bot token).
5. Click "Deploy" and you're live!

If you get stuck, check out the [Railway Docs](https://docs.railway.app/).

---

## Wrap Up

EzProject is all about making Discord group work less stressful and more organized. Whether you're a student, a club leader, or just want to keep your server tidy, give it a try!

---

Want to see the code or contribute? Check it out here: [EzProject on GitHub](https://github.com/blueee04/EzProject)
