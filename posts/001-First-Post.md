# Markdown Notebook
## Or: make the visitors do the work

Markdown is super easy to use and is kind of the Official Language of the Internet now. HTML is old and busted. So when I decided that I was going to put up a collection of technical notebooks as a sort-of-blog, I had to do it in Markdown.

Unfortunately, all of the Markdown blogging systems I found in my 5 minutes of searching were these opinionated beasts.

**Jekyll](https://jekyllrb.com/)** is a Ruby-based system that doesn't, officially, run on Windows. It integrates well with Github Pages, but that integration is through ["A simple Ruby Gem to bootstrap dependencies for setting up and maintaining a local Jekyll environment in sync with GitHub Pages"](https://github.com/github/pages-gem), which seems to want you to turn your blog *itself* into a Ruby gem. It also comes with a command-line tool.

I don't want to write a Ruby gem; I want to write a blog. I thought I could just decide to blog with Jekyll on Github Pages and upload my Jekyll-styled Markdown and have Github build the pages for me. Maybe you can, but I couldn't figure out how to do it. All the tutorials start out with "install some Ruby gems and run this command to make a file structure to put your blog in".

I've never been sold on the "command to stamp out a new project" paradigm. It might be easier when you're making your 27th new Node module or Jekyll blog or whatever, but when I make my first one it's important to me to understand each piece of the project structure and why it has to be there.

**[Octopress](http://octopress.org/)** has on its front page a big writeup about how it's really just Jekyll, and what's more is a massive git repo of Jekyll that you clone and add your own stuff to, with the expectation that you just rebase onto updates from upstream. It's an interesting idea, which might get me out of having to run a magic new blog creation command, but the writeup is all very discouraging about this whole way of doing things, and the new version isn't out yet. It doesn't sound like a software ship I want to sail in right now. Next!

**[Hexo](https://hexo.io/)** was examined long enough for me to read its tagline: "Hexo is a fast, simple & powerful blog framework powered by Node.js". In my experience, Node.js is indeed very powerful, but it's anything but simple. You need to make sure you have the right version of `node` itself. You need to make sure that `npm` has access to a working C/C++ compiler, because somewhere in the long chain of module dependencies that are required by the module you want, there may be native code. You need to make sure that that C/C++ compiler knows about node so it can find the headers. You need to [hope that all those dependencies are still there when you need them](https://www.reddit.com/r/programming/comments/4bjss2/an_11_line_npm_package_called_leftpad_with_only/).

[Here's the dependency graph for Hexo](http://npm.anvaka.com/#/view/2d/hexo). I got tired of watching it after it loaded a couple hundred nodes. Beautiful? Sure! Powerful? Probably! Simple? No.

So, I decided to put together my own solution.