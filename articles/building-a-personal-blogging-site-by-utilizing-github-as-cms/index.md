<img src="https://raw.githubusercontent.com/sifatulrabbi/blogs/main/articles/building-a-personal-blogging-site-by-utilizing-github-as-cms/banner.jpg" height="380px" width="650px" style="background-size: cover" alt="Building a personal blogging site by utilizing Github as the CMS" />

While I was discovering Github a few years ago I found there is a **raw** button for all the files. We can use this feature to get the raw content of the selected file. Also, Github has a pretty decent markdown editor that gives me the preview while I'm editing the file. Recently I wanted to build a personal blogging site to publish my blogs as well as a portfolio to showcase my projects. But I don't want to use WordPress because I like to code. I started to think what if I could utilize Github's raw to serve as my CMS or blog database? And it worked! Here is a breakdown of how everything works.

## Understanding the Github RAW feature

Github serves all its files in raw mode from this URL. So we can use this link to get any of our files if the repository is set to **Public**.

```
https://raw.githubusercontent.com/[username]/[repository]/[branch]/[file_path]
```

### Repository structure

So, I set my repository to public and started structuring my repo to serve as the database for my blogging site. Now, while I was deciding my data schema I've found a couple of limitations of this database.

1. No indexes that can help me with getting all the blog entries.
2. No indexes to give me filtered blogs.
    - Filter by tags
    - Filter by category

So I've structured the repository in the following manner.

```
index.json
articles/
  | [article-id]/
  |   | banner.jpg
  |   | index.md
  |   | [images-for-the-article].jpg
categories/
  | index.json
tags/
  | index.json
```

- `index.json` - This is the entry point for all the published articles. This includes articles with their titles, summaries, timestamps, tags, and categories.
- `articles/` - This directory stores all the articles under their IDs.
    - `banner.jpg` - The banner image of the article
    - `index.md` - This is the primary content of the article.
    - `[images-for-the-article].jpg` - Images or any other required files will also be stored under the article's ID.
- `tags/index.json` - The entry for all the available tags.
- `categories/index.json` - The entry for all the categories.

