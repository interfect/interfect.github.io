# About

## Blog

This blog functions as a repository for my personal project notebooks.

I often find myself wanting to do things that are difficult with existing tools, and some of the time I manage to bang my way through to a solution that didn't exist before. Those solutions live here.

[You can clone this blog on Github.](https://github.com/interfect/interfect.github.io)

## Markdown Notebook

This blog is rendered with Markdown Notebook.

Everything is written in Markdown, and rendered with a client-side Markdown viewer. The viewer is originally based on the [IPFS Markdown Viewer example](https://github.com/ipfs/examples/tree/master/webapps/markdown-viewer), but has since been greatly expanded and improved.

All rendering is done client-side. Unlike other static-blogging tools, no site generation step is needed, so there's no code to run to start blogging, and no [Ruby](https://jekyllrb.com/), [Go](http://gohugo.io/), or [Node.js](http://hexo.io/) to install. This makes the authoring [completely platform-independent](https://jekyllrb.com/docs/windows/). Anything with a text editor and a file upload tool is now a blogging device.

Posts and other pages are vanilla Markdown, with no extra header text. Links to other posts or pages are directly to the Markdown file; the links are automatically rewritten to load the file in the markdown renderer.

On the downside, the lack of any generation step, and the lack of any way to enumerate a directory in JavaScript, make keeping a list of recent posts up to date very difficult. Right now, to make your blog a real blog, with a front page of recent posts, you have to do all the schlepping of text yourself. In the future, there will be some kind of post auto-enumerator, either driven by a hand-maintained list of posts or by a predictable naming scheme.
