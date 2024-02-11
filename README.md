# Adding Articles to the Repository

Welcome to our repository! This guide is designed to help contributors add articles to [ecosystem.api3.org](https://ecosystem.api3.org/) with ease while ensuring consistency and quality. It covers everything from creating a new article to formatting it with custom Markdown components.

## Table of Contents

1. [Creating an Article](#creating-an-article)
   - [Prepare the Article Workspace](#prepare-the-article-workspace)
   - [Adding Metadata to Your Article](#adding-metadata-to-your-article)
2. [Using Components](#using-components)
   - [GitHub Gist](#github-gist)
   - [Figure Block](#figure-block)
3. [Tips and Best Practices for Writing Your Article](#tips-and-best-practices-for-writing-your-article)
   - [Specify Language in Code Blocks](#specify-language-in-code-blocks)

## Creating an Article

### Prepare the Article Workspace


To start crafting your article, follow these two straightforward steps:

1. **Create a Folder:**
   - Create a new folder within the repository to store your article's content.
   - The name of this folder will also serve as the URL slug for your article (e.g., `/articles/your-folder-name`).

2. **Create the Markdown File:**
   - Within this new folder, establish a file named `index.md`. This Markdown file will contain the entirety of your article's content, acting as the primary document.

### Adding Metadata to Your Article

Begin your `index.md` file with the following YAML front matter to provide essential metadata for your article:


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

### Figure Block

When you want to include images within your article with a descriptive caption, use the `figure-block` component. 


This component allows you to insert an image, provide alternative text for accessibility, and add a caption to give context or additional information about the image.

Here's how you can use it in your Markdown file:

```md
::figure-block
---
image: "Image.jpg"  # The link to the image file
alt: "Alternative Text"     # Descriptive text for accessibility and SEO
caption: "Image Caption"    # A caption describing the image
---
::
```


## Tips and Best Practices for Writing Your Article

When writing your article, keep the following tips in mind to ensure your content is rendered better:

### Specify Language in Code Blocks

When including code snippets in your article, always specify the programming language immediately after the opening three backticks. 

This enables syntax highlighting, making your code blocks easier to read and understand. For example:

```javascript
const greeting = 'Hello, world!';
console.log(greeting);
```
