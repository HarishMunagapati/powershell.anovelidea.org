---
layout: single
title: "Publish a Post for a Jekyll Site on a Schedule"
excerpt: "Learn how to schedule the publishing of a post using the GitHub Action, Jekyll Publish Drafts, on a GitHub pages hosted site."
date: '2020-05-11 08:00:00 -0600'
header:
  overlay_image: /assets/images/publish-drafts/marketplace-action.png
  overlay_filter: 0.9
comments: true
tags:
  - blog
  - jekyll
  - github
  - github actions
  - github pages
  - ci
category:
  - blog
---

![Jekyll Publish Drafts]({{ site.url }}{{ site.baseurl }}/assets/images/publish-drafts/marketplace-action.png)
{: .full}

## Scheduled Posts

A few months ago, [Jeff Hicks][1]{:target="_blank"} asked me to participate in a **PSBlogWeek** focused on the release of **PowerShell 7**.
I was ecstatic and honored.
The contributors were asked to publish one to two blog posts on a specific topic, at a specific time.

A specific time, for my static blog, meant that I needed to push my blog post at or near that time manually.
I had no mechanism to schedule a post.

Back then, I did a little searching and found a [GitHub Action][2]{:target="_blank"} that seemed like it would do the trick, but I didn't spend the time to work on it.

In this post, I will show you how easy it is to configure a workflow using [Jekyll Publish Drafts][3]{:target="_blank"} to schedule the publishing of a post for a GitHub Pages hosted Jekyll-based site.

## Blog Technical Stack

For this blog, I use several free and open-source technologies.
GitHub and GitHub Pages provides source control and serves the static Jekyll site.
When I commit a new blog post, Travis CI picks up the change and builds the site and pushes it into the `gh-pages` branch.

If you would like to know more, check out my post, [How I Blog][4]{:target="_blank"}, on this site.

What's important for this post is that I did not want to move my site or change the technology stack.

The `Jekyll Publish Drafts` GitHub Action is the best and simplest solution I found.

## GitHub Actions

Originally introduced in 2018, GitHub Actions have evolved to allow users to *build end-to-end continuous integration (CI) and continuous deployment (CD) capabilities directly in your repository*.
GitHub added support for self-hosted runners in November 2019.

They provide a [Marketplace][5]{:target="_blank"} with Applications and Actions.
Currently it has more than 3500 Actions.
Some actions are for those with GitHub Enterprise licensing, while others are offered on a paid basis.

As the time of this writing, the Action Marketplace has 19 *Jekyll* related actions.

The terms `workflow` and `action` are used in this post interchangeably.
Technically, a workflow is essentially a container for one or more actions.

## Jekyll Drafts

Jekyll has been around nearly twelve years and, since 2013, supports the generation of markdown files in the site's `_drafts` folder using the `--drafts` command line argument.
You can use this while working on a post to generate the site locally to check how images display, verify links, and check grammar.

The `_drafts` folder is where you can save your work-in-progress blog posts.
The markdown files in this folder should not be named with the date prefix, like published posts should be.
This is important when we begin scheduling the publication of the drafts.

## Jekyll Publish Drafts Workflow

With those quick introductions out of the way, let's begin setting up the workflow.

### Actions Permissions

By default, your GitHub repository should allow local and third party Actions.
You can check this by going to your repository's Settings > Actions page.

![Actions Permissions](/assets/images/publish-drafts/github-actions-permissions.png "GitHub Actions Permissions")

### GitHub Marketplace

Viewing `Jekyll Publish Drafts` in the Marketplace gives you a green *Use latest version* button.
This will default to **V2** of the action.

![Use Latest Version](/assets/images/publish-drafts/marketplace-action-use-latest.png "Use Latest Version")

Clicking on it will only give you a couple lines that need be included in the workflow yaml file.

### Create Workflow File

At the root of your site, you need to create a `.github\workflows` folder.
In the folder, create a text file with `.yml` extension.
The name is not significant, but I would suggest using lowercase and avoid spaces.

Here are the contents of my `.github\workflows\publish-drafts.yml` file.

{% raw %}
```yml
name: Publish Blog Drafts

on:
  schedule:
    - cron: '30 */4 * * *'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v1
    - name: Jekyll Publish Drafts
      uses: soywiz/github-action-jekyll-publish-drafts@v2
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        jekyll_path: ./
        branch: master
```
{% endraw %}

#### Name

The `name` property is what your workflow will be called and how it will be listed on the *Action* tab in your site's repository.

#### On Schedule Cron

The trigger will for the workflow will be based on the schedule provided in the `cron` property.

If you're unfamiliar with Linux/Unix, cron is a task scheduler.
The values listed are used to determine when the task, or action in this case, runs.

|minute (0-59)|hour (0-23)|day of month (1-31)|month (1-12)|day of week (0-6)|
|:-:|:-:|:-:|:-:|:-:|
| 30 | */4 | * | * | * |

These values will have my workflow run every 4 hours at the half hour mark.
*I didn't see the need to run every hour.*

* The 30 means at the half hour mark.
* The `*/n` means to run for every `nth` interval of time. In this case, every 4 hours.
* The `*` means *every* interval, much like a wildcard.
* The day of week (0-6) starts with Saturday and ends with Sunday.

### All Workflows

After your workflow has run the first time, you should see it on the *Action* tab.

![All Workflow](/assets/images/publish-drafts/workflows-list-with-failures.png "All Workflows")

You will see that I had a couple failed runs.
I was tinkering with the workflow file and had set `jekyll_path` incorrectly.
Use the default of `./` as it will be the root of your cloned repo.

**NOTE:** You will only see your workflow **after** it has run for the first time.
Additionally, at the time of this writing, you cannot remove runs from the list.
{: .notice--warning}

### Workflow Process

After you have a workflow, let's examine a few things in the logs.

Still in the *Action* tab, click on one of the runs then click on *build* on the left-hand side.

![Publish Blog Drafts](/assets/images/publish-drafts/workflow-steps.png "Publish Blog Drafts")

We can see from this there are four main sections for this workflow.

1. Set up job
2. Run actions/checkout@v1
3. Jekyll Publish Drafts
4. Complete job

#### Set Up Job

![Workflow Step - Set up job](/assets/images/publish-drafts/workflow-set-up-job.png "Workflow Step - Set up job")

In this section, you will find details on the environment provided.
It has interesting information, such as the operating system of the system running the action.

At this time, using *ubuntu-latest* means that it uses Ubuntu 18.04.4.
GitHub even provides a link that defines the software that is included in the build.

#### Run Actions

The second section is where your Jekyll site repository is synchronized to the running system.
It contains the most lines but, ironically, has the least amount of interesting information.

#### Jekyll Publish Drafts

This section is named the same as the workflow name and contains the results of the action.

![Workflow Step - Jekyll Publish Drafts](/assets/images/publish-drafts/workflow-jekyll-publish-drafts.png "Workflow Step - Jekyll Publish Drafts")

For this run, you can see that it found 1 file in my site's `_drafts` folder but did not move it to the `_posts` folder.

## Discoveries

This post was published on a schedule using the configured workflow.

Any discoveries that I find after publishing will be added here.

## Summary

Thank you for reading this post.

If you found this post because you use GitHub Pages with your Jekyll-based site and would like to schedule the publishing of a post, I hope you found it helpful.

If you have any questions, find errors (technical, grammatical, or typographical), or have suggestions that can make my module better,
please leave a comment below.

Thank you, again.

## Reference

Feel free to check out the action's [author's blog post][6]{:target="_blank"} on using `Jekyll Publish Drafts`.
Also, if you want want to dig into the action to see how it works, here is the [GitHub repo][7]{:target="_blank"}.

[1]: https://twitter.com/JeffHicks
[2]: https://help.github.com/en/actions/getting-started-with-github-actions/about-github-actions
[3]: https://github.com/marketplace/actions/jekyll-publish-drafts
[4]: https://powershell.anovelidea.org/blog/how-i-blog/
[5]: https://github.com/marketplace?type=actions
[6]: https://soywiz.com/autopublish-jekyll-drafts/
[7]: https://github.com/soywiz/github-action-jekyll-publish-drafts