
+++
title = "EzProject: Discord Bot for Assignment and Task Management"
date = "2024-08-22"
summary = "EzProject is a modular Python Discord bot that helps manage assignments, tasks, and deadlines in a collaborative server environment."
tags = ["Python", "Discord", "Automation", "Bot", "Task Management"]
+++

## 📌 Project Overview

**EzProject** is a student productivity-focused Discord bot built using Python. It provides an organized way to manage tasks, assignments, and deadlines within a Discord server, using both traditional and modern slash commands. The bot is designed with modular components (called *cogs*) that handle specific features, making the system easy to scale and maintain.

---

## 🗂️ Project Structure Overview

Here's a breakdown of the main components:

```
EzProject/
├── bot/                      # Main bot logic and commands
│   ├── main.py              # Bot launcher
│   ├── slash_commands.py    # Slash command definitions
│   ├── commands.py          # Legacy command support
│   └── cogs/                # Functional modules
│       ├── assignment_management.py
│       ├── deadline_management.py
│       ├── task_management.py
│       ├── role_management.py
│       └── utility_commands.py
├── Datamodule/db.py         # Local data storage
├── config.py                # Token and bot configuration
├── requirements.txt         # Dependencies
├── Procfile, runtime.txt    # Heroku deployment support
```

---

## 🧠 How EzProject Works

### 1. **Bot Initialization**
The entry point is `bot/main.py`, which initializes the Discord client and loads all cog modules from `bot/cogs/`.

```python
client.load_extension("bot.cogs.assignment_management")
client.load_extension("bot.cogs.task_management")
```

This dynamic loading allows each feature (assignment, task, role, utility) to work independently.

---

### 2. **Command Handling**
EzProject uses both traditional commands (`commands.py`) and modern Discord **slash commands** (`slash_commands.py`), offering a smooth user experience.

#### Example Slash Command
```python
@app_commands.command(name="add_assignment", description="Add a new assignment.")
async def add_assignment(interaction: discord.Interaction, name: str, due_date: str):
    ...
```
When this command is triggered in Discord, it captures the input and stores the assignment info using helper functions in `Datamodule/db.py`.

---

### 3. **Cog System**
Each feature is implemented as a **cog**:
- `assignment_management.py`: Add, list, or delete assignments.
- `task_management.py`: Log tasks, update progress.
- `deadline_management.py`: View and manage due dates.
- `role_management.py`: Assign and manage user roles.
- `utility_commands.py`: Extra helper commands (e.g., help, ping, etc.)

This structure makes the bot modular—each cog handles a specific context, listens to relevant commands, and interacts with the data layer.

---

### 4. **Data Storage**
`Datamodule/db.py` provides functions to store user inputs such as:
- Assignment details
- Task status
- User-specific information

The data is currently stored locally in structured files, offering lightweight persistence without needing a database.

---

### 5. **Logging & Config**
Logs are created for debugging or monitoring purposes (optional), and all secrets like bot tokens are set via `config.py`.

---

### 6. **Deployment**
The bot includes a `Procfile` and `runtime.txt`, which means it can be easily deployed on **Heroku**:
```bash
git push heroku main
```
This makes it easy to run 24/7 in the cloud.

---

## ✅ Summary of Workflow

1. User invokes a command (e.g., `/add_assignment`).
2. Bot processes it via the correct cog.
3. Data is stored locally through `db.py`.
4. Bot responds with confirmation (e.g., "Assignment added!").
5. Optional: Data can be retrieved later via `/list_assignments`, etc.

---

This clean, modular flow allows EzProject to work reliably in any Discord server while offering extensibility for future additions.
