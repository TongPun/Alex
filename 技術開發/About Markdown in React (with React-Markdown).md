---
title: "About Markdown in React (with React-Markdown)"
date: "2026-06-03"
description: "A comprehensive guide to implementing and customizing react-markdown in React applications—covering core setup, syntax highlighting, inline HTML extensions, custom annotations, and advanced rendering workflows."
---
# About Markdown in React (with React-Markdown)
# 🎯 Why React-Markdown?
**react-markdown** is a robust, secure alternative to raw HTML/markdown renderers for React. It parses markdown into 
React elements (no **dangerouslySetInnerHTML**!) and supports extensibility via remark/rehype plugins—making it ideal for 
blog platforms, documentation sites, or any app needing rich markdown support.

## ⚡ Core Setup
First, install the foundational package to render markdown content in your React app:

```bash
npm install react-markdown
# For TypeScript projects (optional but recommended)
npm install -D @types/react-markdown
```
Fetch & Parse Markdown Files (From GitHub)
A production-ready function to fetch raw markdown files from GitHub (with caching and error handling):

```js
// According to file name (slug) to get raw Markdown
// types.ts (add type safety first)
export type PostData = {
    slug: string;
    title: string;
    date: string;
    description?: string;
    content: string;
    [key: string]: unknown; // For custom frontmatter fields
};

// constants.ts (centralize config)
export const REPO_OWNER = "your-username";
export const REPO_NAME = "your-repo";
export const BRANCH = "main";
export const GITHUB_TOKEN = process.env.NEXT_PUBLIC_GITHUB_TOKEN; // For rate limit bypass

// markdown-utils.ts
import matter from "gray-matter";
import { PostData } from "./types";
import { REPO_OWNER, REPO_NAME, BRANCH, GITHUB_TOKEN } from "./constants";

/**
 * Fetch and parse a markdown post from GitHub by slug
 * @param slug - Filename (without .md extension)
 * @returns Parsed post data (frontmatter + content) or null on failure
 */
export async function getPostBySlug(slug: string): Promise<PostData | null> {
    const rawUrl = `https://raw.githubusercontent.com/${REPO_OWNER}/${REPO_NAME}/${BRANCH}/${slug}.md`;

    try {
        const response = await fetch(rawUrl, {
            headers: GITHUB_TOKEN ? { Authorization: `token ${GITHUB_TOKEN}` } : {},
            next: { revalidate: 3600 }, // Cache for 1 hour (prevents GitHub rate limits)
        });

        if (!response.ok) {
            console.error(`Failed to fetch post: ${slug} (Status: ${response.status})`);
            return null;
        }

        const rawMarkdown = await response.text();
        const { data, content } = matter(rawMarkdown); // Parse YAML frontmatter

        // Validate required fields
        return {
            slug,
            title: data.title || "Untitled Post",
            date: data.date || new Date().toISOString().split("T")[0],
            content,
            ...data,
        };
    } catch (error) {
        console.error(`Error processing post ${slug}:`, error);
        return null;
    }
}
```

## 🚨 Advanced Code Block Highlighting

Add syntax highlighting to code blocks with react-syntax-highlighter (supports 100+ languages):

~~~bash
# Install dependencies
npm install react-syntax-highlighter @types/react-syntax-highlighter
# Optional: Pre-built themes (dark/light)
npm install highlight.js
~~~
Implementation (React Component)
```tsx
    import React from "react";
import ReactMarkdown from "react-markdown";
import rehypeRaw from "rehype-raw"; // Support inline HTML
import { SyntaxHighlighter } from "react-syntax-highlighter";
import { dracula as darkTheme } from "react-syntax-highlighter/dist/esm/styles/hljs"; // Dark theme
import { github as lightTheme } from "react-syntax-highlighter/dist/esm/styles/hljs"; // Light theme
import { PostData } from "./types";

type MarkdownRendererProps = {
  post: PostData;
  isDarkMode?: boolean;
};

/**
 * Reusable markdown renderer with syntax highlighting
 */
export const MarkdownRenderer: React.FC<MarkdownRendererProps> = ({
  post,
  isDarkMode = true,
}) => {
  return (
    <ReactMarkdown
      className="prose max-w-none" // Add Tailwind typography (optional)
      rehypePlugins={[rehypeRaw]} // Enable inline HTML rendering
      components={{
        // Custom code block renderer
        code({ className, children, ...rest }) {
          // Extract language from className (e.g., "language-js" → "js")
          const match = /language-(\w+)/.exec(className || "");
          const language = match?.[1] || "text";

          return match ? (
            <SyntaxHighlighter
              PreTag="div"
              language={language}
              style={isDarkMode ? darkTheme : lightTheme}
              showLineNumbers={true} // Optional: Add line numbers
              wrapLines={true} // Prevent horizontal overflow
              {...rest}
            >
              {String(children).replace(/\n$/, "")} {/* Clean trailing newlines */}
            </SyntaxHighlighter>
          ) : (
            // Fallback for inline code
            <code
              {...rest}
              className={`${className} bg-gray-100 dark:bg-gray-800 px-1 rounded`}
            >
              {children}
            </code>
          );
        },
      }}
    >
      {post.content}
    </ReactMarkdown>
  );
};
```

## 🎨 Inline HTML Extensions (Rich Formatting)
Leverage inline HTML to add custom styling (colors, underlines, etc.)—supported via rehypeRaw:
More detail [MiMO](https://mimo.org/glossary/html/font-color).

```html
Syntax
<ins>Underlined text</ins><br/>
<font color="#ef4444">Red text</font><br/>
<font color="#3b82f6">Blue text</font><br/>
This is a [link](https://github.com/remarkjs/react-markdown)
```
Output
<ins>Underlined text</ins><br/>
<font color="#ef4444">Red text</font><br/>
<font color="#3b82f6">Blue text</font><br/>
This is a [link](https://github.com/remarkjs/react-markdown)
## 📌 Custom Annotation Blocks (Warning/Note/Tip)
Add visually distinct annotation blocks (with emoji and styling):
```Markdown
> :warning: **Warning:** Do not push the big red button.

> :memo: **Note:** Sunrises are beautiful.

> :bulb: **Tip:** Remember to appreciate the little things in life.
```
Rendered Output:
> :warning: **Warning:** Do not push the big red button.

> :memo: **Note:** Sunrises are beautiful.

> :bulb: **Tip:** Remember to appreciate the little things in life.

## ✅ Task Lists & Tables
```Markdown
* [ ] Set up `react-markdown`
* [x] Add syntax highlighting
* [x] Implement GitHub markdown fetch
* [ ] Add dark/light theme support
```
Rendered Output:
* [ ] Set up `react-markdown`
* [x] Add syntax highlighting
* [x] Implement GitHub markdown fetch
* [ ] Add dark/light theme support

Tables (Clean Formatting)
```Markdown
| Feature                | React-Markdown | Raw Markdown |
|------------------------|----------------|--------------|
| Security (no XSS)      | ✅ Yes         | ❌ No        |
| React Element Output   | ✅ Yes         | ❌ No        |
| Plugin Extensibility   | ✅ Yes         | ❌ No        |
| Inline HTML Support    | ✅ (via rehypeRaw) | ✅ (unsafe) |
```
| Feature                | React-Markdown | Raw Markdown |
|------------------------|----------------|--------------|
| Security (no XSS)      | ✅ Yes         | ❌ No        |
| React Element Output   | ✅ Yes         | ❌ No        |
| Plugin Extensibility   | ✅ Yes         | ❌ No        |
| Inline HTML Support    | ✅ (via rehypeRaw) | ✅ (unsafe) |

# 🧠 How React-Markdown Works (Under the Hood)
The rendering pipeline ensures secure, flexible conversion from markdown to React elements:
```map
                                                           react-markdown
         +----------------------------------------------------------------------------------------------------------------+
         |                                                                                                                |
         |  +----------+        +----------------+        +---------------+       +----------------+       +------------+ |
         |  |          |        |                |        |               |       |                |       |            | |
    markdown-+->+  remark  +-mdast->+ remark plugins +-mdast->+ remark-rehype +-hast->+ rehype plugins +-hast->+ components +-+->react elements
         |  |          |        |                |        |               |       |                |       |            | |
         |  +----------+        +----------------+        +---------------+       +----------------+       +------------+ |
         |                                                                                                                |
         +----------------------------------------------------------------------------------------------------------------+
```
* remark: Parses markdown into MDAST (Markdown Abstract Syntax Tree)
* remark-rehype: Converts MDAST to HAST (HTML Abstract Syntax Tree)
* rehype plugins: Modify HAST (e.g., rehypeRaw for inline HTML)
* Components: Maps HAST nodes to React elements (customizable!)
