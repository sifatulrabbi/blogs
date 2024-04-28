<img src="https://raw.githubusercontent.com/sifatulrabbi/blogs/main/articles/building-a-personal-blogging-site-by-utilizing-github-as-cms/banner.jpg" height="380px" width="650px" style="background-size: cover" alt="Building a personal blogging site by utilizing Github as the CMS" />

While I was discovering Github a few years ago I found there is a **raw** button for all the files. We can use this feature to get the raw content of the selected file. Also, Github has a pretty decent markdown editor that gives me the preview while I'm editing the file. Recently I wanted to build a personal blogging site to publish my blogs as well as a portfolio to showcase my projects. But I don't want to use WordPress because I like to code. I started to think what if I could utilize Github's raw to serve as my CMS or blog database? And it worked! Here is a breakdown of how everything works.

#### Table of contents

-  <a href="#understanding-the-github-raw-feature">Understanding the Github RAW feature</a>
-  <a href="#building-the-go-backend">Building the Go backend</a>
    - <a href="#caching-layer">Caching layer</a>
    - <a href="#api-layer">API layer</a>
-  <a href="#limitations-of-the-current-system">Limitations of the current system</a>

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

You can visit this <a href="https://github.com/sifatulrabbi/blogs/" target="_blank" rel="noreferrer">repository</a> to get a better understanding of how it is structured.

## Building the Go backend

This project was built using Go 1.21.0. The backend is devided into 2 primary layers the API layer and the Caching layer. Since this is not a full tutorial I will only share the interesting parts from the go app. You can visit this <a href="https://github.com/sifatulrabbi/sifatul-api/">repository</a> to get the entire source code. The goals I wanted to achieve with this backend are very simple.

1. Performant and reliable API to get my blogs data as quickly as possible.
2. Flexibility to publish, unpublish, filter, and search blogs.
3. Ability to add new features in the future for example notifications, comments, likes, dislikes, and etc.

### Caching layer

Github's API is very reliable and performant. They also have a caching layer that caches the response for the next 5 minutes. However, I wanted to handle the caching myself because 1. I wanted to showcase my ability of building reliable backends. 2. Ability to switch to a proper database when I will have the time to build the entire CMS from scratch.

#### Features of the caching system

The primary reason to have a caching is to serve data to the user as fast possible and often without querying the database. A common caching strategy will be in-memory caching. Redis is a great in-memory caching database. But I've built my custom in-memory caching system by utilizing Go's various features. These are the features of the caching system.

- Caches any types of data in an in-memory key-value store without requiring any conversion. For example if we use Redis then we will have to convert our data to string before storing them. Since, this caching is built within my system and is written in Go I don't have to convert the data anymore.
- A Cache cleanup cycle that cleans up the expired caches. This is built using Go's goroutine. This goroutine runs in the background in a loop, checks the cache expiration times and takes necessary actions.
- Flexible TTL (time to live) for the cached data. For example, I won't modify the `index.json` (primary entrypoint) too often thus caching for as long as 24 hours is fine. In the contrary I will cache the blog posts for up to 2/3 hours so that I can frequently make changes to my blogs without manually flushing the cache data. _There is a limitation in my Github based CMS which prevents me from building a better cache validation system. Will describe more about this later in the blog._
- Automatically update the cached indexes so that the user can always get the latest blogs. This fetches the `index.json` every 2 hours and updates the caches.

### API layer

The API layer serves the frontend client by connecting both the caching service and the blogs service together. On every request it looks for the cached data if not present fetches the data form Github raw, caches it then responds to the client. The blogs service uses the information from `index.json` files to do filtering, searching, and sorting for the blogs. Since, I will be storing all my published blogs in the `index.json` I didn't need a crawler to find all my available blogs.

### Limitations of the current system

The primary limitation in this current system is that the backend can't directly make any updates to the Github repository unlike a proper database. This means all the updates needed to be done manually by me. For example, if I want to update the last update time of one of my published article then I have to go to my github repository and make the update. This also prevents the caching system to have a proper caching validation. The ideal caching strategy will be able to have the blogs cached unless the `updated_at` timestamp is changed in the index. But maybe in the future when I will build the entire CMS I will implement this more effective caching strategy.
