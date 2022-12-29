---
alias: null
cssClasses: embed-strict
dateCreated: 2022-12-15
dateModified: 2022-12-27
ogImage: 	url:  [[Attachments/Pasted image 20221215132110.png]]
note-stage: 
share: true
tags: 
title: linked-blog
---

# [[linked-blog]]

 A [[Digital Garden]] framework created by the founder of Fleeting Notes, and the core of the stack used to run this website. 

  > [GitHub - matthewwong525/linked-blog-starter: Host your second brain for free with Next.JS and Tailwind v3](https://github.com/matthewwong525/linked-blog-starter)

## Features

 - uses [[next.js]] (vs jekyll/hugo)
 - deploys on [[vercel]]
 - from a proven dev (creator of Fleeting Notes)
 - looks nice :) (vs quartz/others)
 - separate repos for the pure content (.md) and for the site
- Converts ObsidianMD to HTML
	-   [obsidian-export](https://github.com/zoni/obsidian-export) (Obsidian MD -> [Common MD](https://commonmark.org/))
	-   [remarkjs](https://github.com/remarkjs/remark) (Common MD -> HTML)
	-   [github-markdown-css](https://github.com/sindresorhus/github-markdown-css) (HTML -> Beautiful HTML)

## Changes from the Base Version

I have modified linked-blog-starter to have a few new features. 
[SamuelCochrane/linked-blog-website](https://github.com/SamuelCochrane/linked-blog-website)  has:

- [x] date created _and_ date modified fields ✅ 2022-12-27
	- [x] fix bug where dateModified isn't coming through ✅ 2022-12-28
- [x] footers[^1] ✅ 2022-12-27
- Still Todo:
- [ ] homepage 
	- [x] `[[👱‍♂️]]`🤍 ☕ ✅ 2022-12-27
	- [ ] "recent posts"
		- [x] page itself ✅ 2022-12-27
		- [ ] content pulling ([fleeting-notes-website/[pid].tsx at 4e08fbd86600d3cdec2c6f6ee4e78725096839c0 · fleetingnotes/fleeting-notes-website · GitHub](https://github.com/fleetingnotes/fleeting-notes-website/blob/4e08fbd86600d3cdec2c6f6ee4e78725096839c0/pages/posts/%5Bpid%5D.tsx)), just pull top 6
	- [ ] 
- [ ] post header/banner images
	- [ ] [Using Remote Image specified in a Blog Post’s Markdown Frontmatter](https://mediajams.dev/post/Using-Remote-Image-specified-in-a-Blog-Post's-Markdown-Frontmatter-w-Gatsby-Image)
- [ ] add other frontmatter values to post-meta.tsx
	- [ ] note-stage
	- [ ] tags?
- maybe someday:
- [ ] admonitions/callouts
- [ ] ecalidraw working (either uploaded and embedded, or hotswapped with an image when published)
- [ ] gisqus comments/reactions (see [fleeting-notes](https://github.com/fleetingnotes/fleeting-notes-website/tree/main/components/blog) implementation)

[^1]: like this.

## Publishing Process

How this website is updated:
1. Tag content in my [[Obsidian|obsidian.md]] vault I want to publish with the frontmatter `share: true` in .
	1. Linter adds relevant frontmatter like title, dateCreated, dateModified
2. Run the Obsidian Publish plugin to push the selected notes to my github markdown-only repo [SamuelCochrane/linked-blog-md](https://github.com/SamuelCochrane/linked-blog-md).
	1. plugin uses regex to remove the markdown `#title` from the note (as it will be handled by the `title` frontmatter when published)
3. this triggers a GitHub Action which grabs the website source code from my site-only repo [SamuelCochrane/linked-blog-website](https://github.com/SamuelCochrane/linked-blog-website) and stitches it together with the updated markdown files, deploying via [[vercel]].
4. The site is hosted on Vercel.

## Forking This Site

If you want to use this as the base, this site is open source and linked above. It is provided as-is, and it would be bold to assume even _I_ remember how it works.
This blog wouldn't have been possible without Matthew Wong's starter code. All credit goes to him, all bugs are from me.

Some specific things you'll probably want to change:
- The blog title, currently lives in header.tsx
- The frontmatter variables like `dateCreated` if you named things differently in your own vault
	- (These will mostly be in /components/, post.ts, and api.ts, but I'd use a Ctrl+H over the entire project to find any instances just in case)
- the favicon
- The text in the landing page (landing/hero-home.tsx)

The domain was purchased from Google Domains, and represents my only ongoing expense hosting this website.
