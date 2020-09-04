---
title: "Add an 'edit post' button to your Gatsby blog"
tags: ["gatsby", "react", "git", "webdev"]
license: "public-domain"
slug: "gatsby-edit-button"
canonical_url: "https://haseebmajid.dev/blog/gatsby-edit-button/"
date: "2020-09-07"
published: true
cover_image: "images/cover.png"
---

In this article, we will look at how we can add an "edit post" button, to your Gatsby blog. When this button is clicked it will take the user to your markdown file, on github/gitlab/ that was used to generate this blog post.

`video({ src = "./images/main.mp4" })`

## Setup

Before we add the edit button to a Gatsby blog, let's set up a simple Gatsby site using the `Gatsby blog starter`,
you can of course skip this step and add search to an existing site.

```bash
npm -g install gatsby-cli
gatsby new my-blog-starter https://github.com/gatsbyjs/gatsby-starter-blog
```

The main thing you will need is the `gatsby-source-filesystem`, to import our markdown files. So that your `gatsby-config.js` looks like this:

```js:title=gatsby-config.js
  {
    resolve: `gatsby-source-filesystem`,
    options: {
      path: `${__dirname}/content/blog`,
      name: `blog`,
    },
  },
```

and the `gatsby-transformer-remark` plugin:

```js:title=gatsby-config.js
  {
    resolve: `gatsby-transformer-remark`,
    options: {
      // ...
    },
  },
```

## (Optional) Blog Post

Let's assume our `gatsby-node.js` file looks like this, this is how we create
a new blog post from each of our markdown files.

```js:title=gatsby-node.js
exports.createPages = async ({ graphql, actions }) => {
  const { createPage } = actions;

  const blogPost = path.resolve(`./src/templates/blog-post.js`);
  const result = await graphql(
    `
      {
        allMarkdownRemark(
          sort: { fields: [frontmatter___date], order: DESC }
          limit: 1000
        ) {
          edges {
            node {
              fields {
                slug
              }
              frontmatter {
                title
              }
            }
          }
        }
      }
    `
  );

  if (result.errors) {
    throw result.errors;
  }

  // Create blog posts pages.
  const posts = result.data.allMarkdownRemark.edges;

  posts.forEach((post, index) => {
    const previous = index === posts.length - 1 ? null : posts[index + 1].node;
    const next = index === 0 ? null : posts[index - 1].node;

    createPage({
      path: post.node.fields.slug,
      component: blogPost,
      context: {
        slug: post.node.fields.slug,
        previous,
        next,
      },
    });
  });
};
```

Let's assume that our `blog-post.js` looks like this:

```jsx:title=src/templates/blog-post.js
import React from "react";
import { Link, graphql } from "gatsby";

// ...

const BlogPostTemplate = ({ data, pageContext, location }) => {
  const post = data.markdownRemark;
  const siteTitle = data.site.siteMetadata.title;
  const { previous, next } = pageContext;

  return (
    <Layout location={location} title={siteTitle}>
      <SEO
        title={post.frontmatter.title}
        description={post.frontmatter.description || post.excerpt}
      />
      // ...
    </Layout>
  );
};

export default BlogPostTemplate;

export const pageQuery = graphql`
  query BlogPostBySlug($slug: String!) {
    site {
      siteMetadata {
        title
      }
    }
    markdownRemark(fields: { slug: { eq: $slug } }) {
      id
      excerpt(pruneLength: 160)
      html
      fileAbsolutePath
      frontmatter {
        title
        date(formatString: "MMMM DD, YYYY")
        description
      }
    }
  }
`;
```

## Edit Button

Ok, now we need two pieces of information the location of our project on git where our
markdown files are stored. In this example, it's here `https://gitlab.com/hmajid2301/articles`.

First, we need a way to get the file path of the markdown file, well we can do this with using our GraphQL query.
The same query we use to get other information such as title and contents. All we need to add is `fileAbsolutePath`
to the `markdownRemark` part of our query. This will return as the name suggests the absolute path to the file,
i.e. `/home/haseeb/projects/personal/articles/34. Gatsby edit button/source_code/content/blog/hello-world/index.md`..

```js:title=src/templates/blog-post.js
export const pageQuery = graphql`{11}
  query BlogPostBySlug($slug: String!) {
    site {
      siteMetadata {
        title
      }
    }
    markdownRemark(fields: { slug: { eq: $slug } }) {
      id
      excerpt(pruneLength: 160)
      html
      frontmatter {
        title
        date(formatString: "MMMM DD, YYYY")
        description
      }
    }
  }
`;
```

Now we need a way to use this file path into a git URL file path. Since I know that
`articles/` is a git repo, we want to remove `/home/haseeb/projects/personal/articles`
from `/home/haseeb/projects/personal/articles/34. Gatsby edit button/source_code/content/blog/hello-world/index.md`
and then assuming the git URL of our repo, where the markdown files exist is `https://gitlab.com/hmajid2301/articles`. The path to our markdown file on git could be something like
`https://gitlab.com/hmajid2301/articles/-/blob/master/34. Gatsby edit button/source_code/content/blog/hello-world/index.md`.

So let's add logic to our `blog-post.js` file to generate this git URL. After we
update our query. Here the logic below in the `getGitMarkdownUrl()` function.

```jsx:title=src/templates/blog-post.js
const BlogPostTemplate = ({ data, pageContext, location }) => {
  const post = data.markdownRemark;
  const siteTitle = data.site.siteMetadata.title;
  const { previous, next } = pageContext;

  function getGitMarkdownUrl() {
    const pathConst = "/articles/";
    const gitURL = "https://gitlab.com/hmajid2301/articles";
    const sliceIndex =
      post.fileAbsolutePath.indexOf(pathConst) + pathConst.length;
    const markdownFileGitPath = post.fileAbsolutePath.slice(sliceIndex);
    const blogPostOnGit = `${gitURL}/-/blob/master/${markdownFileGitPath}`;
    return blogPostOnGit;
  }

  const gitMarkdownUrl = getGitMarkdownUrl();

  // ....
};
```

> Warn: Don't forget to change the `gitURL` variable in your project!

Where the following two lines remove everything before `/articles/`, so we get
`34. Gatsby edit button/source_code/content/blog/hello-world/index.md`.

```js
const sliceIndex = post.fileAbsolutePath.indexOf(pathConst) + pathConst.length;
const markdownFileGitPath = post.fileAbsolutePath.slice(sliceIndex);
```

Then we combine this with our git URL, to end up with the path to the markdown file, `https://gitlab.com/hmajid2301/articles/-/blob/master/34. Gatsby edit button/source_code/content/blog/hello-world/index.md`.

```js
const blogPostOnGit = `${gitURL}/-/blob/master/${markdownFileGitPath}`;
```

Finally, all we need to do is add the button edit button and have it link to
this `gitMarkdownUrl`. You can do something like this below:

```jsx
<a href={gitMarkdownUrl} rel="noreferrer" target="_blank">
  EDIT THIS POST
</a>
```

If you want to make it look fancier you can use `react-icons` to get a proper
edit icon (as shown in the gif above).

That's it! That's all we needed to do when the user clicks on the edit button it'll
take them to the git repo where the markdown files exist. They can then perhaps fork
the project make their edit and open a new merge or pull request (GitLab vs GitHub)
and add in the changes they want (if approved by you).

## Appendix

- [Source Code](https://gitlab.com/hmajid2301/articles/tree/master/34.%20Gatsby%20edit%20button/source_code)
- [Site in video](https://haseebmajid.dev/)
- [Source code](https://gitlab.com/hmajid2301/portfolio-site) for site in video
