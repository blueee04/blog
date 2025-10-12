+++
title = "Got Bored... Tried n8n"
date = 2025-10-12
summary = "How I automated my job search so I can focus on applying."
tags = ["n8n", "automation", "job-search", "AI", "productivity"]
+++

# Got Bored... Tried n8n

Job hunting was exhausting. Between scrolling through LinkedIn, tailoring my resume for each position, and keeping track of where I've applied, it felt like a full-time job itself. So I did what any reasonable person would do—I automated the whole thing.

I have been scrolling through Twitter and n8n has really interested me for a while, so I thought — why not try it out? I went through some tutorials; it took an hour to get the basics running.

Here's what it does:

![Workflow diagram](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2025-10-13-Got%20Bored...Tried%20n8n/workflow.png)

- Pulls my resume
- Looks for new jobs that match my preferences
- Scores how well I fit each role
- Writes match details and a short tailored cover letter to a Google Sheet
- Sends a quick email when new opportunities are ready

![Results in a sheet](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2025-10-13-Got%20Bored...Tried%20n8n/sheets.png)

i mean i could improve this more like adding more job boards more than linkedIn like naukri and all...but for now i should focus on other stuffs since some deadlines are really close lmao ...

I didn't test auto-applying yet. I'll drop the JSON file below if you wanna try it for yourself — just ping me if you need help setting up any of the nodes.

![Here You Go](https://raw.githubusercontent.com/blueee04/blog/main/content\Downloads\n8n-jobcheck.json)
