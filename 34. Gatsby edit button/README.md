---
title: "Add an 'edit post button' to your Gatsby blog"
tags: ["gatsby", "git", "webdev"]
license: "public-domain"
slug: "gatsby-edit-button"
canonical_url: "https://haseebmajid.dev/blog/gatsby-edit-button/"
date: "2020-09-7"
published: true
cover_image: "images/cover.png"
---

In this article we will look how we can add an "edit post" button, to your Gatsby blog. Where this button when clicked will take the user to your markdown related to this post.

`video({ src = "./images/main.mp4" })`

## Setup

Before we add search Gatsby blog, let's setup a simple Gatsby site using the `Gatsby blog starter`, you can of course
skip this step and add search to an existing site.

```bash
npm -g install gatsby-cli
gatsby new my-blog-starter https://github.com/gatsbyjs/gatsby-starter-blog
```

The main thing you will need is the `gatsby-source-filesystem`, to import our markdown files. So that your `gatsby-config.js` looks like this:

```js
  {
    resolve: `gatsby-source-filesystem`,
    options: {
      path: `${__dirname}/content/blog`,
      name: `blog`,
    },
  },
```

and the `gatsby-transformer-remark` plugin:

```js
  {
    resolve: `gatsby-transformer-remark`,
  },
```

## Edit Button

## Appendix
