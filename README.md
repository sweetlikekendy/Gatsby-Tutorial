# Kendy's GatsbyJS Notes

Based off the official tutorial from [Gatsby's documentation](https://www.gatsbyjs.org/tutorial/).

## File Structure

Typical file structure for a Gatsby project will look like this:

```
.
+-- src
|   +-- components
    |   +-- layout.js
|   +-- images
|   +-- pages
    |   +-- about.js
    |   +-- index.js
    |   +-- some-markdown.md
|   +-- templates
    |   +-- blog-post.js
|   +-- utils
+-- public
+-- gatsby-config.js
+-- gatsby-node.js
```

Continued explanation will follow this file structure from the top down.

## Components

Example relative path for a component:

```
src > components > layout.js
```

A layout component is used for sections you want to use throughout multiple pages (e.g. header & footer).

When used with GraphQL, the query must be in a StaticQuery component. This is imported through the gatsby library. The query is a property of the StaticQuery component.

```
src > components > layout.js // example layout without styling and query
```

```javascript
import { StaticQuery } from "gatsby";

export default ({ children }) => (
  <StaticQuery
    query={graphql`
      query {
        site {
          siteMetadata {
            title
          }
        }
      }
    `}
    render={data => (
      <div>
        <Link to={`/`}>
          <h3>{data.site.siteMetadata.title}</h3>
        </Link>
        <Link to={`/about/`}>About</Link>
        {children}
      </div>
    )}
  />
);
```

## Images

Images will go here

## Pages

All pages in site will go here. Import your layout to use on all your pages from layout.js. Page queries live outside of the component and go at the end of the file. It **DOES NOT** use the StaticQuery component like your layouts.

```
src > pages > about.js
```

```javascript
import React from "react";
import { graphql } from "gatsby";
import Layout from "../components/layout";

export default ({ data }) => (
  <Layout>
    <h1>About {data.site.siteMetadata.title}</h1>
    <p>
      We're the only site running on your computer dedicated to showing the best
      photos and videos of pandas eating lots of food.
    </p>
  </Layout>
);

export const query = graphql`
  query {
    site {
      siteMetadata {
        title
      }
    }
  }
`;
```

## Templates

When programmatically creating pages with [createPages](https://www.gatsbyjs.org/docs/node-apis/#createPages) API in gatsby-node.js, you will need a template to reference

## Utils

Plugins will use files in here for configurations.

Example of a [Typography.js](https://kyleamathews.github.io/typography.js/) plugin:

```javascript
import Typography from "typography";
import kirkhamTheme from "typography-theme-kirkham";

const typography = new Typography(kirkhamTheme);

export default typography;
export const rhythm = typography.rhythm;
```

## Gatsby-config.js

This file will enable plugins & hold site metadata.

```javascript
module.exports = {
  siteMetadata: {
    // site metadata goes here
    title: `some title`
  },
  plugins: [
    // plugins go here
  ]
};
```

## Gatsby-node.js

Get file path to create a slug field with GraphQl query. Create page from new slug field.

```javascript
const path = require(`path`);
const { createFilePath } = require(`gatsby-source-filesystem`);

exports.onCreateNode = ({ node, getNode, actions }) => {
  const { createNodeField } = actions;
  if (node.internal.type === `MarkdownRemark`) {
    const slug = createFilePath({ node, getNode, basePath: `pages` });
    createNodeField({
      node,
      name: `slug`,
      value: slug
    });
  }
};

exports.createPages = ({ graphql, actions }) => {
  const { createPage } = actions;
  return graphql(`
    {
      allMarkdownRemark {
        edges {
          node {
            fields {
              slug
            }
          }
        }
      }
    }
  `).then(result => {
    result.data.allMarkdownRemark.edges.forEach(({ node }) => {
      createPage({
        path: node.fields.slug,
        component: path.resolve(`./src/templates/blog-post.js`),
        context: {
          // Data passed to context is available
          // in page queries as GraphQL variables.
          slug: node.fields.slug
        }
      });
    });
  });
};
```
