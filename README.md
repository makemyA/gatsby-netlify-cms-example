# Add netlify-cms to gatsby starter blog

## About

A quick tutorial to explain how to manage images you add from the netlify-cms.


## Tutorial

### 1. Installation

Follow [this instruction](https://www.netlifycms.org/docs/gatsby/) to link your gatsby-starter-blog to netlify-cms. Be careful to indentation when you copy/paste the **config.yml file**

### 2. Gatsby-plugins

A quick explanation about gatsby-plugins we will use for the image management. All these infos are copy/paste directly from [gatsby](https://www.gatsbyjs.org)

- gatsby-source-filesystem

A Gatsby source plugin for sourcing data into your Gatsby application from your local filesystem.

The plugin creates File nodes from files. The various “transformer” plugins can transform File nodes into various other types of data e.g. gatsby-transformer-json transforms JSON files into JSON data nodes and gatsby-transformer-remark transforms markdown files into MarkdownRemark nodes from which you can query an HTML representation of the markdown.

- gatsby-transformer-remark

Parses Markdown files using Remark.

- gatsby-images

Speedy, optimized images without the work.

gatsby-image is a React component specially designed to work seamlessly with Gatsby’s GraphQL queries. It combines Gatsby’s native image processing capabilities with advanced image loading techniques to easily and completely optimize image loading for your sites. gatsby-image uses gatsby-plugin-sharp to power its image transformations.

Note: gatsby-image is not a drop-in replacement for <img />. It’s optimized for fixed width/height images and images that stretch the full-width of a container. Some ways you can use <img /> won’t work with gatsby-image.

- gatsby-remark-relative-images

Convert image src(s) in markdown to be relative to their node’s parent directory. This will help gatsby-remark-images match images outside the node folder. For example, use with NetlifyCMS.

- gatsby-remark-images

Processes images in markdown so they can be used in the production build.

In the processing, it makes images responsive by:

Adding an elastic container to hold the size of the image while it loads to avoid layout jumps.
Generating multiple versions of images at different widths and sets the srcset and sizes of the img element so regardless of the width of the device, the correct image is downloaded.
Using the “blur up” technique popularized by Medium and Facebook where a small 20px wide version of the image is shown as a placeholder until the actual image is downloaded.

- gatsby-transformer-sharp

Creates ImageSharp nodes from image types that are supported by the Sharp image processing library and provides fields in their GraphQL types for processing your images in a variety of ways including resizing, cropping, and creating responsive images.

- gatsby-plugin-sharp

Exposes several image processing functions built on the Sharp image processing library. This is a low-level helper plugin generally used by other Gatsby plugins. You generally shouldn’t be using this directly but might find it helpful if doing very custom image processing.

It aims to provide excellent out-of-the box settings for processing common web image formats.

For JPEGs it generates progressive images with a default quality level of 50.

For PNGs it uses pngquant to compress images. By default it uses a quality setting of [50-75]. The pngCompressionSpeed value is a speed/quality trade-off from 1 (brute-force) to 10 (fastest). Speed 10 has 5% lower quality, but is 8 times faster than the default (4). In most cases you should stick with the default, but if you have very large numbers of PNGs then it can significantly reduce build times.

### 3. Install missing plugins and configure gatsby-config.js

All plugins are already include in gatsby-stater-blog except this one.

```js
yarn add gatsby-remark-relative-images
```

Configure your gatsby-config.js

```js
plugins: [
  `gatsby-plugin-netlify-cms`,
  {
    resolve: `gatsby-source-filesystem`, // install and add this plugin to link your blog to netlify cms
    options: {
      path: `${__dirname}/static/img`, //The path where netlify-cms will store all images you add from the cms
      name: "uploads",
    },
  },
  ...{
    resolve: `gatsby-transformer-remark`,
    options: {
      plugins: [
        {
          resolve: `gatsby-remark-relative-images`, // Add our new plugin to current configuration
          options: {
            name: "uploads", // the name should be the same than which one mentionned in gatsby-souce-filesystem
          },
        },

        {
          resolve: `gatsby-remark-images`,
          options: {
            maxWidth: 590,
          },
        },
        {
          resolve: `gatsby-remark-responsive-iframe`,
          options: {
            wrapperStyle: `margin-bottom: 1.0725rem`,
          },
        },
        `gatsby-remark-prismjs`,
        `gatsby-remark-copy-linked-files`,
        `gatsby-remark-smartypants`,
      ],
    },
  },
]
```
### 3. Configure gatsby-node.js

We add this function to have access to our images relative path

```js
const { fmImagesToRelative } = require("gatsby-remark-relative-images")

...

exports.onCreateNode = ({ node, actions, getNode }) => {
  const { createNodeField } = actions
  fmImagesToRelative(node)
  if (node.internal.type === `MarkdownRemark`) {
    const value = createFilePath({ node, getNode })
    createNodeField({
      name: `slug`,
      node,
      value,
    })
  }
}

```

### 5. Manage images uploaded from netlify-cms

- add A new widget "image" in our config.yml to allow users of netlify cms to upload an image in their blog post

```yml
backend:
  name: git-gateway
  branch: master

media_folder: static/img
public_folder: /img

collections:
  - name: "blog"
    label: "Blog"
    folder: "content/blog"
    create: true
    slug: "{{year}}-{{month}}-{{day}}-{{slug}}"
    editor:
      preview: false
    fields:
      - { label: "Title", name: "title", widget: "string" }
      - { label: "Publish Date", name: "date", widget: "datetime" }
      - { label: "Description", name: "description", widget: "string" }
      - { label: "Image", name: "image", widget: "image" } //our new Field
      - { label: "Body", name: "body", widget: "markdown" }
```

- Configure the blog template

In src/templates/blog-posts.js

We import the Image component from gatsby-image, then we add the image field in graphql query.

```js
import Image from "gatsby-image"

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
      frontmatter {
        title
        date(formatString: "MMMM DD, YYYY")
        description
        image {
          childImageSharp {
            fluid {
              ...GatsbyImageSharpFluid
            }
          }
        }
      }
    }
  }
`
```

finally We can use our new Image component in our blog template.

```js
class BlogPostTemplate extends React.Component {
  render() {
    const post = this.props.data.markdownRemark
    ...
    return (
      <Layout>
        ...
        <article>
          <Image
              fluid={post.frontmatter.image.childImageSharp.fluid}
              alt="image"
              style={{
                marginRight: rhythm(1 / 2),
                marginBottom: 0,
                minWidth: 50,
              }}
              imgStyle={{
              }}
            />
            <section dangerouslySetInnerHTML={{ __html: post.html }} />
            <hr
              style={{
                marginBottom: rhythm(1),
              }}
            />
        </article>
        ...
      </Layout>
          )}}

```
