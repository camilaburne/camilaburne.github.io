---
layout: post
categories: "en"
title: "How to locally test your github pages on Mac"
date: 2021-07-03
tags: jekyll blog github

---

I followed this [amazing guide from J McGlone](http://jmcglone.com/guides/github-pages/) to build this simple github pages, and eventually I got tired of making a thousand commits to test different css styles. I then tried to run my page locally but I struggled with Jekyll and Ruby and Gems. After following [this](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll#prerequisites) & [this](https://jekyllrb.com/docs/) & [this](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/?utm_source=stackoverflow#introduction) & [this](https://www.moncefbelyamani.com/making-github-pages-work-with-latest-jekyll/) guidelines, I finally did it! Here's how:

1. To create a simple github pages, clone [this repo](https://github.com/hankquinlan/hankquinlan.github.io) and follow [this guideline to understand the repo structure.](http://jmcglone.com/guides/github-pages/)

2. To run your site locally, install ruby, gems and Jekyll from [this repo](https://github.com/monfresh/install-ruby-on-macos)

3. Create new Jekyll website as shown on [this documentation](https://jekyllrb.com/docs/). Note that when you create this project, you will see two files in the project folder: Gemfile and Gemfile.lock

4. Copy and paste these two Gemfiles into your github pages project folder. While you're on your github pages folder, run jekyll serve on your terminal. Your local site will be running on this server:  Server address: [http://127.0.0.1:4000](http://127.0.0.1:4000) unless you specified any other port in the config yml file or in the jekyll serve statement.


And now I'm able to run my github pages locally 🥳 <br><br>
<img src="/images/local_github_page_test.png">
