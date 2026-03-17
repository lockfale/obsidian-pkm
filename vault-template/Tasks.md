---
tags: [tasks, dashboard]
---

# Tasks

> **One-time setup:** Tasks plugin → Settings → General → enable **"Set created date on every added task"**. This powers the Stale review section.

Tasks live anywhere — daily notes, project notes, or added directly below. Check them off from any query result. New tasks go to the appropriate backlog automatically based on tag.

**Tag convention:** `#work` · `#personal` · `#home` · `#learn`
**Due date:** `📅 YYYY-MM-DD` · **Priority:** `⏫` high · `🔼` medium · `🔽` low

## Task Search
```tasks
description includes test
```



---
## 🔴 Overdue

```tasks
path does not include Admin/

not done
due before today
sort by priority
sort by due
```

---
## 🔴 Improperly Captured

```tasks
path does not include Admin/
path does not include Goals

not done
no tags
sort by priority
```


---
## 📅 Due Today

```tasks
path does not include Admin/

not done
due today
sort by priority
```

---
## 📆 Coming Up — Next 7 Days

```tasks
hide backlinks

path does not include Admin/

not done
due after today
due before in 8 days
sort by due
```

---

# Backlogs
---
## 💼 Work Backlog

```tasks
hide backlinks

path does not include /Admin

not done
tags include #work
sort by due
```

---
## 🏠 Personal Backlog

```tasks
hide backlink

path does not include Admin/

not done
tags include #personal
sort by due
```

---
## 🏠 Home Backlog

```tasks
hide backlink

path does not include Admin/

not done
tags include #home
sort by due
```

---
## 📚 Learning Backlog

```tasks
hide backlinks

path does not include Admin/

not done
tags include #learn
sort by due
```


---
## 📚 Health Reoccurring Tasks

- [ ] Morning routine #health 🔁 every day when done ➕ 2026-03-16 📅 2026-03-17
- [ ] Exercise #health 🔁 every day when done ➕ 2026-03-16 📅 2026-03-17


---

## ⚠️ Stale — Needs Review

*Review during weekly Sunday review. For each item: assign a date, do it this week, or delete it.*

### Work (no due date, 14+ days old)

```tasks
show created date

not done
no due date
tags include #work
created before 14 days ago
sort by created
path does not include /Admin/Templates
path does not include Archives
```

### Personal & Home (no due date, 30+ days old)

```tasks
show created date
not done
no due date
(tags include #personal) OR (tags include #home)
created before 30 days ago
sort by created
path does not include /Admin/Templates
path does not include Archives
```


---
## Reference
https://publish.obsidian.md/tasks/Queries/Layout

---
## 🏗️ Project Tasks

### Sample Project (#SampleProjectTag)

### Sample Second Project (#SampleSecondTag)


---
## ✏️ Non-Project Tasks
*`/close-day` files non-project tasks here. Tag + date to surface in daily queries.*

### Financial


### Home — Projects



### Personal



### Work (standalone)



## Related
[[Goals/2. Monthly Goals|2. Monthly Goals]]
[[Goals/3. Weekly Review|3. Weekly Review]]
