---
layout: post
title: Jekyll Deployment Notes
---

So I have my repository for Github Pages.   I decided - for no really good reason - to initial jekyll in a blog subfolder.

Using the usual git commands:

```
git add blog/
git commit
git push
```

When I deployed I had the issue that I got this error from the good folks at Github:

```
The page build failed with the following error:

A file was included in blog/about.md that is a symlink or does not exist in your _includes directory. For more information, see https://help.github.com/articles/page-build-failed-file-is-a-symlink.
```

So I've checked out the page.   It seems to imply the `_includes` folder should be in the root directory of the website.  

In the end, I fixed it by blowing away the entire folder and restarting with a fresh jekyll deployment in the root folder.



