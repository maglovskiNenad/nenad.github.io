---
layout: post
title: "GitHub Pages to LinkedIn with Make"
date: 2026-03-24 09:00:00 -0500
categories: linux
tages: github linked in make automate blog
image:
    path: /assets/img/headers/GitHub-Pages.png
---


# GitHub Pages to LinkedIn with Make

A simple step-by-step guide for beginners.

This guide shows how to:

1. Publish a blog post on a GitHub Pages site.
2. Detect the new post with Make.
3. Automatically create a LinkedIn post.

---

## 1. What you need

Before you start, make sure you have:

- a GitHub Pages blog or documentation site
- a working RSS or Atom feed
- a Make account
- a LinkedIn account
- permission to post on LinkedIn

If your site uses Jekyll, the feed is usually:

```text
https://yourname.github.io/feed.xml
```

or, for a project site:

```text
https://yourname.github.io/repository-name/feed.xml
```

`jekyll-feed` generates the feed at `/feed.xml`. It is not usually called `index.xml`. 

---

## 2. Make sure your feed works

Open your feed in the browser.

Example:

```text
https://yourname.github.io/feed.xml
```

If it works, you will probably see a big block of XML text.

That is normal.
You do **not** need a pretty page.
You only need a valid feed URL.

The Make RSS trigger expects the URL of an RSS or Atom feed, not the homepage of your website. 

### Important

Use:

```text
.../feed.xml
```

Do **not** use only:

```text
https://yourname.github.io
```

---

## 3. Create the Make scenario

In Make:

1. Open **Dashboard**.
2. Click **Create a new scenario**.
3. Click the big **+** button.
4. Search for **RSS**.
5. Select **RSS**.
6. Choose **Watch RSS feed items**.

Make describes this module as the trigger that starts when a new RSS feed item is posted. citeturn865443search0

---

## 4. Configure the RSS module

You will see several fields.

### URL

Paste your full feed URL:

```text
https://yourname.github.io/feed.xml
```

### Maximum number of returned items

Set it to:

```text
1
```

Why?
Because it is the easiest and safest option for testing.
It tells Make to return only one item per run.

### Choose the starting point

You may see options like:

- Select the first RSS feed item
- RSS feed items from after a specific date
- All RSS feed items
- From now on

For normal real use, choose:

**From now on**

This means Make will wait for posts that appear **after** this moment.
The RSS watch module is designed to react to new items. citeturn865443search0turn865443search2

### What this means in plain English

If you choose **From now on**, Make will **not** use your old blog posts.
It will only watch for a **new** post in the future.

---

## 5. Save the RSS module

After you fill in the fields:

1. Click **OK** or **Save**.
2. Return to the scenario canvas.

Now you should see your first module on the screen.

---

## 6. Add the LinkedIn module

Now add the second step.

1. Click the **+** on the right side of the RSS module.
2. Search for **LinkedIn**.
3. Select the LinkedIn app.
4. Choose the post creation module.

Depending on your setup, this may be a user post or a company/page post.

---

## 7. Create the LinkedIn connection

When Make asks for a connection, it will ask for a **Connection name**.

This can be any name you want.
It is only a label for you.

Examples:

- `LinkedIn Personal`
- `My LinkedIn`
- `LinkedIn Blog Posts`
- `LinkedIn Company Page`

Then:

1. Click **Save** or **Continue**.
2. Sign in to LinkedIn.
3. Approve the permissions.
4. Return to Make.

---

## 8. Fill in the LinkedIn fields

When you configure the LinkedIn module, you may see fields such as:

- Content
- Mentions
- Media type
- Visibility

Here is what to do.

### Content

This is the main text of your LinkedIn post.

Use a simple template like this:

```text
New blog post is live:

[Title from RSS]

Read it here:
[Link from RSS]

#blog #documentation #linux #sysadmin
```

Inside Make, do not type the RSS values as plain text if you can avoid it.
Instead, click inside the field and insert variables from the RSS module, such as:

- Title
- Link
- Description

A slightly better version is:

```text
New blog post is live:

[Title from RSS]

[Description from RSS]

Read it here:
[Link from RSS]

#blog #documentation #linux #sysadmin
```

### Mentions

Leave this empty for now.

Only use it if you want to tag a person or company.

### Media type

For the first test, leave this as:

- `None`
- or no media

The simplest first test is a plain LinkedIn post with text and a link.

### Visibility

Set this to:

- `PUBLIC`

if you want other people to see your post.

---

## 9. Why nothing happens sometimes

This is the most common beginner problem.

If you selected **From now on**, then Make waits for a **new** post.
If you do not publish a new post, nothing happens.

That is normal.

### Example

If you created the scenario today, but your latest blog post was published yesterday, Make will ignore that old post.
It will only react when a new item appears in the feed.

---

## 10. How to test the automation

You have two good testing methods.

### Method A: Best test for real usage

1. Keep **From now on** selected.
2. Create a brand new test blog post.
3. Push it to GitHub.
4. Wait until it appears on your site.
5. Open `feed.xml` and confirm the new post is there.
6. Go back to Make.
7. Click **Run once**.

If everything is correct, Make should detect the new RSS item and send it to LinkedIn.

### Method B: Fast test with an old post

If you want to test immediately with an existing post:

1. Open the RSS module again.
2. Change the starting option from **From now on** to:
   - **Select the first RSS feed item**
   - or **RSS feed items from after a specific date**
3. Save the module.
4. Click **Run once**.

This lets Make use a post that already exists in the feed.

---

## 11. Suggested test post

You can create a very small test post like this:

**Title:**

```text
Test Post
```

**Body:**

```text
This is a test post for LinkedIn automation.
```

After publishing it:

1. check that it is visible on your blog
2. check that it appears in `feed.xml`
3. run the Make scenario

---

## 12. Recommended beginner settings

Use these values for your first setup:

### RSS module

- **URL**: your full `feed.xml` URL
- **Maximum number of returned items**: `1`
- **Start option**: `From now on`

### LinkedIn module

- **Connection name**: anything clear, for example `LinkedIn Personal`
- **Content**: title + link + hashtags
- **Mentions**: empty
- **Media type**: `None`
- **Visibility**: `PUBLIC`

---

## 13. Troubleshooting

### Problem: I used my blog homepage URL

**Fix:** use the feed URL instead.

Correct:

```text
https://yourname.github.io/feed.xml
```

Wrong:

```text
https://yourname.github.io
```

The RSS module expects a feed URL.

### Problem: I cannot find `index.xml`

**Fix:** check `feed.xml`.

With Jekyll and `jekyll-feed`, the usual path is `/feed.xml`. 

### Problem: I only see a huge text page

**Fix:** that is fine.

That huge text page is usually the XML feed.
You only need the URL.

### Problem: I do not see any post on LinkedIn

Possible reasons:

- you selected **From now on** and there is no new post yet
- the new post is not visible in `feed.xml` yet
- the LinkedIn module is not connected correctly
- the scenario was not run
- the scenario is saved but not turned on for future runs

### Problem: I do not know what to write in Content

Use this:

```text
New blog post is live:

[Title from RSS]

Read it here:
[Link from RSS]

#blog #documentation #linux #sysadmin
```

### Problem: I do not know what to put in Connection name

Use any simple label, for example:

```text
LinkedIn Personal
```

### Problem: I do not know what to put in Mentions, Media type, or Visibility

Use:

- **Mentions**: empty
- **Media type**: `None`
- **Visibility**: `PUBLIC`

---

## 14. Final workflow summary

Your final automation should look like this:

```text
GitHub Pages blog -> feed.xml -> Make RSS trigger -> LinkedIn post
```

In Make, the scenario should look like this:

```text
RSS: Watch RSS feed items -> LinkedIn: Create post
```

---

## 15. Beginner checklist

Use this quick checklist before testing:

- my GitHub Pages site is online
- my `feed.xml` opens in the browser
- I used the full `feed.xml` URL in Make
- maximum returned items is set to `1`
- I understand what **From now on** means
- I connected LinkedIn successfully
- I wrote a simple Content template
- Mentions is empty
- Media type is `None`
- Visibility is `PUBLIC`
- I created a new test post or changed the RSS starting option for testing

---

## 16. Simple example of the full process

1. I publish a new post on my GitHub Pages blog.
2. The post appears in `feed.xml`.
3. Make checks the feed.
4. Make detects the new RSS item.
5. Make sends the post title and link to LinkedIn.
6. LinkedIn publishes the post.

---

## 17. Best practice for beginners

Start simple.
Do not add images, mentions, or complex formatting in the first version.

First make this work:

- title
- link
- a few hashtags

After that, improve the text later.

---

## 18. References

- Make RSS integration and RSS trigger documentation. 
- Jekyll Feed plugin documentation for `/feed.xml`. 