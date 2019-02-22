---
title: Speaker notes
author: Alexey Shiklomanov
---

# Prerequisites

Software installed:

- R
- RStudio
- `tidyverse` R package
- Git (from git-scm.com or GitHub Desktop)

Make sure RStudio knows about Git:
"Tools -> Global options -> Git/SVN"

# Create the project

Create a new RStudio project
New project -> Check "use git"

Make a data directory.
(In the console...)

```r
dir.create("data")
```

Download bandlist data.

```r
download.file("https://raw.githubusercontent.com/FoRTExperiment/FoRTE-canopy/master/data/A_bandlist.csv",
              "data/A_bandlist.csv")
```

# First commit

Click the "git" tab in RStudio.
Should see 3 files: `.gitignore`, `data/`, and `forte-canopy-git.Rproj`.

Git will not track anything for you automatically.
_It is not Dropbox_.
Instead, think of Git as a wedding photo album.
- A **repository** refers to a collection of files and their revision history in a directory that are tracked by Git. Here, our repository is the `forte-canopy-git` directory. Think: photo album
- A **commit** is a snapshot of the repository (and all the files therein) at a particular point. A commit is usually accompanied by a helpful message describing what happened most recently. Think: Photo.
- The **working directory** is the files as you see them. Think: The stuff being photographed.

The menu you see shows the difference between your working directory and the last photo you took.
In this case, we have no photos, so everything shows up here...but git doesn't know what to do with these files yet -- photographer shows up, and these are the first people she sees, not sure what to do.

We want all of these files, so let's photograph them.
First, line up the people you want in this shot -- select the files.

- The **staging area** contains files as they are about to be committed. Think: Lining up a photo.

Then, click "Commit", add a message, and click "Commit".

# Commit 2

Create a new script (`analysis.R`).

Read in the data:

```r
library(tidyverse)
a_bandlist_raw <- read_csv("data/A_bandlist.csv")
a_bandlist_raw
glimpse(a_bandlist_raw)
```

Now, commit it.

# Commit 3

Goal is to summarize.
What's varying in the data?

```r
summarize_all(a_bandlist_raw, n_distinct)
```

What do the categories look like?

```r
count(a_bandlist_raw, Species)
count(a_bandlist_raw, Health_status)
count(a_bandlist_raw, Canopy_status)
```

Look at Git.
These are modifications (hence "M").
Let's commit them.

# Commit 4

A few problems:
1. There's a problem in the `Species` column: `FAGR#` is probably supposed to be `FAGR`.
2. `Health_status` and `Canopy_status` have short codes that aren't very meaningful.

Let's fix them.
 
```r
a_bandlist <- a_bandlist_raw %>%
  select(-X1) %>%
  mutate(
    Species = factor(Species) %>%
      fct_recode("FAGR" = "FAGR#"),
    Health_status = factor(Health_status, c("L", "M")) %>%
      fct_recode("alive" = "L", "dead" = "M"),
    Canopy_status = factor(Canopy_status, c("OD", "OS", "UN")) %>%
      fct_recode("canopy" = "OD", "subcanopy" = "OS", "understory" = "UN")
  )
a_bandlist
```

And now, we can create our summary.

```r
species_summary <- a_bandlist %>%
  group_by(Species) %>%
  summarize(
    mean_DBH = mean(DBH_cm, na.rm = TRUE),
    frac_alive = mean(Health_status == "alive", na.rm = TRUE),
    frac_canopy = mean(Canopy_status == "canopy", na.rm = TRUE),
    frac_subcanopy = mean(Canopy_status == "subcanopy", na.rm = TRUE)
  )
species_summary
```

Let's also create a plot summary.

```r
ggplot(a_bandlist) +
  aes(x = SubplotID, y = DBH_cm, fill = Species) +
  geom_boxplot()
```


## First commit