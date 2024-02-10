# Adding Articles to the Repository

Welcome to our repository! This guide is designed to help contributors add articles with ease while ensuring consistency and quality. It covers everything from creating a new article to formatting it with custom Markdown components.

## Table of Contents

1. [Creating an Article](#creating-an-article)
   - [Prepare the Article Workspace](#prepare-the-article-workspace)
   - [Write Your Article](#write-your-article)
2. [Formatting Your Article](#formatting-your-article)
3. [Using Components](#using-components)
   - [GitHub Gist](#github-gist)

## Creating an Article

1. **Prepare the Article Workspace:**
   - **Create a Folder:** Start by creating a new folder within the repository. This folder will house all the content related to your article.
   - **Create the Markdown File:** Inside this folder, create a Markdown file named `index.md`. This file will serve as the main document for your article.

2. **Write Your Article:**
   - **Content:** Draft your article directly in the `index.md` file, utilizing standard Markdown syntax for all formatting needs.

## Formatting Your Article

To ensure uniformity across the repository, begin your `index.md` file with the following YAML front matter:

```yaml
---
title: Article Title
subtitle: Article Subtitle
author: Author Name
date_published: YYYY-MM-DD # Format should be Year-Month-Day
category: Category         # For example, Technology, Health, etc.
image: Path/To/CoverImage.jpg  # Include a link to your cover image
hidden: true                  # Set to true to prevent automatic listing
---
```


## Using Components

### Github Gist

To include a GitHub Gist in your article, use the following block:

```md
::github-gist
---
gistUrl: "https://gist.github.com/Username/GistID"
gistId: "GistID"
---
::
```
