---
title: Setting Up Your Own Website Using Github Pages
permalink: /technology/github-pages
tags: ["technology", "github pages", "github"]
---

## What is GitHub Pages?

I've wanted to create my own website for a very long time. A few friends who already had cybersecurity blogs ([NoSecurity](https://nosecurity.blog) & [blauersec](https://blauersec.com/)) told me they used GitHub Pages, and convinced me to do the same. Today, I want to show you how easy it is to create your own website using GitHub Pages. There are many websites hosting services that allow you to create your own website easily, but you have to pay for them. There are affordable options available, but I don't want to pay for any of these services and I doubt you do either. After going through it myself, I am going to teach you how to host your own website using GitHub Pages.

GitHub Pages is a alternative to all these paid services, and handles everything for you. It turns a repository into a website, and is completely free. You don't need any developer experience to get your own website up and running either.

## Prerequisites

There are a few things you have to know/learn before using GitHub Pages.

- Git
- Jekyll
- Basic Linux commands
- Virtual machine

I won't be covering everything in depth, but I will try my best to get you up to speed. I didn't know anything about Git or Jekyll before creating my website, but it is really easy to learn. I actually just dived into making my website without knowing a single thing about Git or Jekyll, and created my website by the end of the day. You can do this.

## What is Git?

Git is a version control system that allows you to track the changes of a project over time. It acts as a "save" button. Every change made is tracked and managed in a central repository. When you commit a change, Git gives that snapshot an ID and it becomes an official checkpoint of your project.

### Downloading Git

You could download Git for Windows, or use a Linux virtual machine (I'll explain later).

**Windows**: [https://git-scm.com/downloads](https://git-scm.com/downloads)

**Linux**: `sudo apt install git`

### Basic Git Commands

#### Basic Commands

I only used 3-4 commands when working on my website, but these are basic Git commands to get you started with creating your website.

- `git init`: initializes a repository

- `git clone <link/to/GitHub/repo>`: clones a repository to your local machine
  - `git clone <https://github.com/<username>/<username.github.io>` will clone your website's repository to a local machine

- `git branch <branch-name>`: creates a new branch
  - `git checkout <branch-name>`: switch to another branch

#### Pushing Commits

- `git status`: checks the changes made to the repository

- `git add <file>`:  stages file(s) to be added to a commit

- `git commit -m "<message>"`: snapshots the current changes of your repository

- `git push`: pushes the changes in a local repository to a central repository
  - `git push -u origin main`: pushes the commit to the main branch of your repository

### Resources

Here are some extra resources to help you get a better understanding of Git.

#### Websites

- [https://www.notion.so/Introduction-to-Git-ac396a0697704709a12b6a0e545db049](https://www.notion.so/Introduction-to-Git-ac396a0697704709a12b6a0e545db049)
- [https://www.atlassian.com/git/tutorials/what-is-version-control](https://www.atlassian.com/git/tutorials/what-is-version-control)

#### Videos

- [Learn Git in 15 Minutes](https://youtu.be/USjZcfj8yxE)
- [Git & GitHub Crash Course For Beginners](https://youtu.be/SWYqp7iY_Tc)

#### Cheat Sheets

- [https://www.atlassian.com/git/tutorials/atlassian-git-cheatsheet](https://www.atlassian.com/git/tutorials/atlassian-git-cheatsheet)
- [https://education.github.com/git-cheat-sheet-education.pdf](https://education.github.com/git-cheat-sheet-education.pdf)

## What is Jekyll?

Jekyll is a static website generator that makes it easier to create websites without needing any web developer experience. Basically, Jekyll converts your files to what is needed to host your website. All you have to do is create the contents of your website.

### Jekyll Commands

`jekyll new .`: makes a new Jekyll site with default content (directory needs to be empty)

`bundle exec jekyll serve`: initial command to host a local website on `localhost:4000`

- This command will check the current directory and build the website. You can use this command to see how your website will look before pushing any official changes to your repository.

- `jekyll serve`: you can just type this after first time to host the local website on port 4000

### Frontmatter

**Frontmatter**: gives information about each page in YAML or JSON format

- Author, date, title, layout
- You can also create custom frontmatter variables

```yaml
---
layout: post
title:  "Welcome to Jekyll!"
date:   2021-03-10 11:30:09 -0800
categories: jekyll update
---
```

Frontmatter is also metadata that determines how files will be read/stored on your website.

The link to the default post is `localhost:4000/jekyll/update/2021/3/10/welcome-to-jekyll.html`. Notice that the `categories` section determines the address of the blog post. Right after `/jekyll/update`, you also see the date the blog was created followed by the title of the blog. This is the default naming scheme of your posts.

`categories: jekyll update`

- This will create the directories within the `_site` directory that the blog will be stored in.

`YYYY-MM-DD-<title-of-post>`

- This is the default naming scheme when making the markdown files that will be our blogs. The `YYYY-MM-DD` and dashes in between words are necessary when creating a file. There should be no spaces in your file name.

### Blog Post Syntax

Naming convention: `YYYY-MM-DD-<title-of-post>`

File type: markdown or HTML files

#### Frontmatter

First thing to include on the post is frontmatter

- `layout: post`: defines the template to be a post
- `tite: <title>`: overrides the default naming convention

#### Organizing Your Blogs

You don't have to follow the default naming scheme. I suggest you to come up with a way to store your blogs. You can make subdirectories within `_posts` to organize your blog posts. It doesn't change the permalink of the post.

- `_drafts`: directory to store unfinished posts that aren't going to be posted yet
  - `_drafts` is located in the home directory, *NOT* in _posts
  - Run `jekyll serve --draft` to show drafts on your website

#### Permalinks

Permalinks give permanent links to your blog posts. This is the way I organize my blogs since I don't like unnecessary directories and dates on my link. I came up with several overarching "directories" that categorize my posts.

- `/technology` is the permalink for my posts relating to technology
- `/htb` is the permalink for my posts relating to Hack the Box

`permalink: <"my-new-url">` sets the permanent link for the post

##### Sample Blog Post

These are 2 ways to format your the address to your blog. You can use categories and the current date, or give the post a permanent link. I recommend sticking with the markdown format when creating your blog posts.

```markdown
---
layout: post
title:  "Welcome to Jekyll!"
date:   2021-03-10 11:30:09 -0800
categories: jekyll update
---
Begin writing your blog post here...
```

```markdown
---
layout: post
title:  "Welcome to Jekyll!"
date:   2021-03-10 11:30:09 -0800
permalink: /directory/you/want
---
Begin writing your blog post here...
```

### Pages

So far, we've been working with posts. The `layout` category determines what type of page you want on your website. Pages are anything on our website that aren't posts (About, Contact).

- They are just regular markdown/html files on the home directory
- Use `layout: page` to define the page layout
- If pages are in directories, the address is going to be `localhost:4000/<directory>/<page>`

### Frontmatter Defaults

You can set frontmatter defaults in the `config.yml` file so it appears automagically when you create a post. `_config.yml` is the file to define the default frontmatter

- Make sure to restart the server after any changes to reflect any changes on your website

```yaml
defaults:
  -
    scope:
      path: "" # an empty string here means all files in the project
      type: "posts"
    values:
      layout: "post"
      author: Robinson
```

`path` specifies to the directories that default frontmatter will apply to

`type` specifies what type of pages (page, post) default frontmatter will apply to

`layout` defines what type of layout will be used

- So without `type`, all files will default to the post layout

### Directory Structure

```
<username>.github.io/
├─ _drafts
└──── draft1
└──── draft2
├─ _posts
└──── post1
├─ _site
├─ .gitignore
├─ 404.html
├─ _config.yml
├─ about.md
├─ Gemfile
├─ index.md
├─ README.md
```

**_drafts**: You can create this directory to store your unfinished blog posts. It will not show on your website.

**_posts**: Directory that stores all your published posts

**_site**: Holds finished product of the website (Don't need to change anything in this directory).

**404.html**: the error file that will show when something isn't reachable on your website

**_config.yml**: Settings for your Jekyll website

**Gemfile**: Stores website dependencies

**about.md**: The about page of our website

**index.md**: The home page of our website

### Themes

Links to find Jekyll Themes:

- [RubyGems](https://rubygems.org/search?query=jekyll-themes)
- [jekyll-themes](https://jekyll-themes.com/free/)
- [jekyllthemes.io](https://jekyllthemes.io/)

There are a lot of Jekyll themes online that you can use to make your website nicer. I'm currently using [Minima](https://github.com/jekyll/minima). I really like the simple UI for my blog. It's also really easy to navigate the directory structure, and create new blog posts using this theme.

### Resources

I learned everything I needed to know about Jekyll from Mike Dane's [tutorial](https://www.youtube.com/playlist?list=PLLAZ4kZ9dFpOPV5C5Ay0pHaa0RJFhcmcB) on YouTube. I also used Jekyll's [documentation](https://jekyllrb.com/docs/).

## Getting Started

1. Create a GitHub account
2. Create a new repository as `<username>.github.io`
3. Install Git & Jekyll
4. Clone the repository to your VM
5. Start blogging

### Workflow

> You could download Git for Windows, or use a Linux virtual machine (I'll explain later).

I recommend creating a virtual machine to initially create the site. Jekyll is more compatible with Linux than Windows, so we are going to create a Linux virtual machine to make life easier. This is also a great way to get started with Linux as well! After the initial Jekyll setup, you can stick with this workflow or install Git on Windows and work on your website from your host machine. Here is some [documentation](https://jekyllrb.com/docs/installation/windows/) on how to get Jekyll to work on Windows. I will be going over how to install Jekyll on a Linux VM.

### Installing Git & Jekyll

After setting up your VM, the first thing to do is install Git and Jekyll. I followed the installation process for Ubuntu from Jekyll's [documentation](https://jekyllrb.com/docs/installation/ubuntu/).

```bash
sudo apt install git ruby-full build-essential zlib1g-dev

echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

gem install jekyll bundler
```

Jekyll requires a empty directory, so we can create the website and then move it into our repository.

```bash
mkdir </new/directory>
cd </new/directory>
jekyll new .
mv * /path/to/repository
```

From here, you can create a new markdown file in `_posts` to make your first blog. Congratulations!
