---
layout: post
title:  "How to create a blog"
date:   2017-02-03 18:20:19
category: Tips & Hacks
---

I want to create a blog.

Who would want to hear what I have to say? Good Question.

However, I see this not only as a means of communicating my work efforts with others, but also a way to document what I have done so I personally can keep track of the thought processes and decisions behind my various projects. If people have a little entertainment and learn somthing along the way then great.

So as a first post, here's how I went about creating this blog...

First off - I am cheap. I don't want to pay for someone to host my blog. Neither do I want unnecessary adverts clogging up the pages. Therefore ideally I wanted to do something free, at the cost of getting my hands a little dirtier than usual. I don't need any fancy dynamic stuff here, just static pages, so after a little digging I found that [Github](https://github.com/) is an ideal hosting platform for such a site. Setting up a site on Github is as easy as creating a new repository, and giving it a certain name.. i.e. [Your Username].github.io. 

Easy!

Obviously you need to add some content though. I didn't go down this route, but Github does offer a number of templates which you can just deploy out of the box. None of them took my fancy, and also I didn't really want something that *everyone* else had. 

Now, I`m not about to start creating a load of html and css. This project is for writing english, not code. So therefore I needed some out-of-the-box infrastructure to deploy to my new github repo. Another bout of digging and I stumbled upon [Jekyll](https://jekyllrb.com), which seems to be widely used for this sort of thing. There are heaps of sites based on Github/Jekyll. 

After working through the install instructions, I decided that Jekyll was still too much like hard work.

A third bout of digging revealed that there is a whole community of hard working people who have create Jekyll themes. These are essentially entire jekyll-based sites, complete with advanced features like comments, social media integration, etc, all with pretty css - just waiting to be forked from their github repos. All you therefore have to do from that point on, is write posts in a markdown file. 

Simple!

I`m picky about my colour schemes, but finally decided on [Paper Theme](https://github.com/dbtek/paper). 

# Summary

The complete set of instructions to start blogging RIGHT NOW are :

- Fork the [repository](https://github.com/dbtek/paper).
- Rename forked repo as `username.github.io`.
- Add your blog name and details by editing `_config.yml`.
- Write some posts!

First Post. Done.
