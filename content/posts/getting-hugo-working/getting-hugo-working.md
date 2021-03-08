---
title: "Getting Hugo Working"
date: 2021-03-05T15:47:06+01:00
draft: false
categories: []
tags: ["hugo", "Github actions"]
---

Recently I remembered that I made a website using github pages.
This website was never updated after initially creating it. 
It was my business card while applying for my very first job in development.
Join me on my journey discovering static site generators, github actions and hugo.

<!--more-->
# Why I used hugo

When I decided to create a new website, this time using a static site generator I only knew [Jekyll](https://jekyllrb.com/) by name. When checking out Jekyll it turned out I first needed to install Ruby, RubyGems as well as GCC and Make.

I started looking for alternatives which specifically have less dependencies. The reason is simple, I was lazy. Enter google, looking over a few possibilities I settled on [Hugo](https://gohugo.io).

The main reasons are nicely summarized on the [Hugo website](https://gohugo.io/about/what-is-hugo/#who-should-use-hugo)

>Hugo is for people that prefer writing in a text editor over a browser.
>
>Hugo is for people who want to hand code their own website without worrying about setting up complicated runtimes, dependencies and databases.
>
>Hugo is for people building a blog, a company site, a portfolio site, documentation, a single landing page, or a website with thousands of pages.

# Setting up hugo

The first steps were setting up hugo ([the official guide](https://gohugo.io/getting-started/installing/)). I used Chocolatey which made the whole process as simple as opening a command shell and executing a single command.

The following step was preparing the repository hosting my website using github-pages. The first thing I did was creating a new branch to ensure my old website remained available. It turns out this was definitely the right choice as will become clear in another section.

After cloning the repository, as I said I hadn't changed anything in 4 years, I started setting up hugo. The first attempt was following the hugo quick start guide.
Overall this worked however it felt *clunky* to have both the source used to generate the site and the generated files in the same place.

While I succeeded in displaying a newly generated website some things felt off. This is one of my strengths/flaws in development. I do not only focus on the codebase and product but I get hung up on tooling. For me it is important that the correct tool is used correctly. That is I did not like the source and the result existing in the same place.

## RTFM, or how Hugo already addresses this

While playing some more with the website and its content I also kept reading selected pages in the documentation. While not highlighted as much as it could have been, the answer was already there.

> Publish your hugo files and generated content into [two different repositories](https://gohugo.io/hosting-and-deployment/hosting-on-github/#github-user-or-organization-pages)

The longer I'm working with Hugo, the clearer it becomes that their documentation contains most answers. There hasn't been a Hugo-related issue (yet) for which I needed information not found on the website.

Furthermore the page linked above also introduces an example of a Github action which deploys the website automagically.

# 'Struggling' with Github actions

Since the documentation already gives an example of using github actions to deploy the generated files, I went with the [default action](https://gohugo.io/hosting-and-deployment/hosting-on-github/#build-hugo-with-github-action).

This action has 4 steps

1. [`actions/checkout@v2`](https://github.com/marketplace/actions/checkout): Download the latest version in the virtual environment
2. [`peaceiris/actions-hugo@v2`](https://github.com/marketplace/actions/hugo-setup): Setup hugo in the environment
3. `run: hugo --minify`: Executes Hugo in the current directory.
4. [`peaceiris/actions-gh-pages@v3`](https://github.com/marketplace/actions/github-pages-action): Deploy the output of step 3

When first setting up this action the build step failed. 

![The github action failed chief](/images/posts/getting-hugo-working/failed-action.png)

Somehow I made a mistake adding the theme as a submodule. I still haven't got an idea what went wrong. After deleting the submodule and re-adding it, things started working. I'll write a follow-up on this issue later. There are 2 aspects of my specific situation I'll look into there, the hugo repository was private at that time and I added the themes directory to the `.gitignore` file.

## Deploying to another repository

Deploying the generated code requires authentication. Otherwise it would be simple to change code in repositories without any kinds of checks. I imagine this would be a dream for malicious actors, inject a piece of code without anyone knowing and you're golden. A security breach like this would be an enormous issue.

Getting back on topic, the example given in the documentation  I chose to use deploy keys. These are (in my opinion) the better way. They only work for specific repositories whereas a personal access token would allow write access to each and every repository I, as a user, can write to.

After fixing the build step, the action got stuck when deploying. I started looking at the documentation provided for the deploy action [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages). The first issue I encountered was due to the way I generated the key.

I copy-pasted the following command directly into powershell

```bash
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
```

Powershell will complain about the value passed for the `-N` option which is used to set the `N`ew passphrase. Without thinking anymore I fixed this by setting a password. Set up the deploy key and lets go, right?

Nope, looking back it is quite obvious that this will not work. When setting up the SSH connection we obviously need the pass phrase if the key is generated with a non-empty pass-phrase. I couldn't figure this out because even after 15 minutes the action remained active.

I figured why not use a personal access token, that way I would be able to make sure it could actually work. And to my surprise and joy it just worked (after I changed the parameter from `deploy_key` to `personal_token` in the action definition).

![First succesful run](/images/posts/getting-hugo-working/success-with-personal-token.png)

So now I know for a fact that it should work but why didn't it before? I went back to one of the runs that I cancelled. This time there was output available in the deploy step. One glance at the output made it clear that I should've created a key without passphrase.

![I found some clues](/images/posts/getting-hugo-working/deploy-key-clues.png)

This was about the time I considered my first attempt at using a static site generator and github actions a success. I'd set up an automatic deploy action, my first ever usage of github actions. I replaced my old website with a new placeholder. And I had loads of fun.

My final and annotated version of the action is available in the following gist. This might be another topic for me to look into, can I update the gist when the action is changed?

{{< gist labiej 67650fa15285cd4249d03e8c36326b89 >}}

Next time I'll be looking into customization of a hugo theme.