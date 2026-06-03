---
title: "About Markdown in React"
date: "2026-06-03"
description: "It is a markdown example with react-markdown. It will record how to use react-markdown with different 
markdown syntax such as Color, Code block, Inline HTML, Colored Text, Link, Blockquote, Warning, Note, Tip, and Bulb."
---

# 🚀 About Markdown in React
This article is about how to use react-markdown with different markdown syntax. I want to add some function on my blog website.
I will learn about **Code Block** and **Inline HTML**.

## Markdown
We need to install a required package **react-markdown** so that my blog website can display markdown(.md) files.

```bash
npm install react-markdown
```
I create function to get Markdown file from GitHub.
```js
// According to file name (slug) to get raw Markdown
export async function getPostBySlug(slug: string): Promise<PostData | null> {
    const rawUrl = `https://raw.githubusercontent.com/${REPO_OWNER}/${REPO_NAME}/${BRANCH}/${slug}.md`;

	try {
    const response = await fetch(rawUrl, {
    headers: GITHUB_TOKEN ? { Authorization: `token ${GITHUB_TOKEN}` } : {},
    next: { revalidate: 3600 } // 快取 1 小時，避免頻繁請求觸發 GitHub Rate Limit
    });

    if (!response.ok) return null;

    const rawMarkdown = await response.text();

    // 解析 YAML Frontmatter 和主要內文
    const { data, content } = matter(rawMarkdown);

    return {
    slug,
    title: data.title || '無標題',
    date: data.date || '',
    content,
    ...data,
    };
    } catch (error) {
    console.error('抓取文章失敗:', error);
    return null;
    }
    }
```

## Code Block

When you want to add a code block, you can use SyntaxHighlighter. It can highlight code in many languages.

~~~bash
npm install react-syntax-highlighter @types/react-syntax-highlighter
~~~
and then set the code as following style.
```reactjs
    <ReactMarkdown rehypePlugins={[rehypeRaw]} components={{
                   code({ className, children, ...rest }) {
                   const match = /language-(\w+)/.exec(className || "");
    return match ? (
    <SyntaxHighlighter
            PreTag="div"
    language={(match[1] || 'text') as string}
    style={dark}
    {...(rest as Record<string, unknown>)}
    >
    {String(children)}
    </SyntaxHighlighter>
    ) : (
    <code {...rest} className={className}>
        {children}
    </code>
    );
    },
    }}>{post.content}</ReactMarkdown>
```

## 📝 Inline HTML
Inline HTML can be used to add colors, links, and other elements to your Markdown.
More detail [MiMO](https://mimo.org/glossary/html/font-color).

```html
Some of these words <ins>will be underlined</ins>.
<font color="red">This text is red!</font>
This is a [link](https://github.com/remarkjs/react-markdown)
```
Some of these words <ins>will be underlined</ins>.<br/>
<font color="red">This text is red!</font><br/>
This is a [link](https://github.com/remarkjs/react-markdown)
## Tags
using some Icon to do the notifcation
> :warning: **Warning:** Do not push the big red button.

> :memo: **Note:** Sunrises are beautiful.

> :bulb: **Tip:** Remember to appreciate the little things in life.

## Lists
* [ ] todo
* [x] done

## table

| a | b |
| - | - |

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
