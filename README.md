# eleventy-blog-website

# Contentful + 11ty + Nunjucks Blog Starter

A minimal static blog using **Contentful CMS**, **11ty static site generator**, and **Nunjucks templates**.

---

## ✅ Features
- Headless CMS (Contentful)
- Static generation with 11ty
- API fetching via Axios
- Templating with Nunjucks
- Environment config support
- Easy deployment to Netlify

---

## 🚀 Quick Start

### 1. Clone and Install
```bash
git clone https://github.com/your-username/contentful-11ty-blog.git
cd contentful-11ty-blog
npm install
```

### 2. Setup Environment Variables
Create a `.env` file:
```
CONTENTFUL_SPACE_ID=your_space_id
CONTENTFUL_ACCESS_TOKEN=your_access_token
```

> Create a **Content model** called `Blog Post` with fields:
> - `title` (Text)
> - `slug` (Short text)
> - `body` (Long text)
> - `date` (Date)
> - `coverImage` (Media)

Add 1–2 blog entries manually via Contentful UI.

### 3. Run the Project
```bash
npx eleventy --serve
```
Visit: http://localhost:8080

---

## 📁 Folder Structure
```
my-blog/
├── .eleventy.js
├── .env
├── package.json
├── src/
│   ├── blog/
│   │   └── blog.njk
│   ├── includes/
│   │   └── layout.njk
│   └── index.njk
└── _data/
    └── posts.js
```

---

## 🔧 Configuration

### `.eleventy.js`
```js
module.exports = function (eleventyConfig) {
  eleventyConfig.addPassthroughCopy("src/assets");
  return {
    dir: {
      input: "src",
      includes: "includes",
      data: "../_data",
      output: "dist",
    },
  };
};
```

### `_data/posts.js`
```js
const axios = require("axios");
require("dotenv").config();

module.exports = async function () {
  const url = `https://cdn.contentful.com/spaces/${process.env.CONTENTFUL_SPACE_ID}/environments/master/entries`;
  const res = await axios.get(url, {
    headers: {
      Authorization: `Bearer ${process.env.CONTENTFUL_ACCESS_TOKEN}`,
    },
    params: {
      content_type: "blogPost",
    },
  });

  const assets = res.data.includes.Asset;
  return res.data.items.map(item => {
    const imageId = item.fields.coverImage.sys.id;
    const imageUrl = assets.find(a => a.sys.id === imageId)?.fields?.file?.url;
    return {
      title: item.fields.title,
      slug: item.fields.slug,
      body: item.fields.body,
      date: item.fields.date,
      coverImage: `https:${imageUrl}`,
    };
  });
};
```

### `src/index.njk`
```nunjucks
---
layout: layout.njk
title: Home
---

<h1>Blog Posts</h1>
<ul>
  {% for post in posts %}
    <li>
      <h2><a href="/blog/{{ post.slug }}/">{{ post.title }}</a></h2>
      <img src="{{ post.coverImage }}" alt="{{ post.title }}" width="300" />
      <p>{{ post.date }}</p>
    </li>
  {% endfor %}
</ul>
```

### `src/blog/blog.njk`
```nunjucks
---
layout: layout.njk
pagination:
  data: posts
  size: 1
  alias: post
  addAllPagesToCollections: true
permalink: "/blog/{{ post.slug }}/"
---

<article>
  <h1>{{ post.title }}</h1>
  <img src="{{ post.coverImage }}" alt="{{ post.title }}" />
  <p>{{ post.date }}</p>
  <div>{{ post.body }}</div>
</article>
```

### `src/includes/layout.njk`
```nunjucks
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{{ title }}</title>
</head>
<body>
  <nav><a href="/">Home</a></nav>
  <main>
    {{ content | safe }}
  </main>
</body>
</html>
```

---

## 📦 Deploy to Netlify
- Push repo to GitHub
- Import in Netlify
- Set environment variables in Netlify dashboard
- Build command: `npx eleventy`
- Output directory: `dist`

---
