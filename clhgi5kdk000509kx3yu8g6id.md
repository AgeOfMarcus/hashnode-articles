---
title: "Custom Analytics for Hashnode - Part 2"
seoDescription: "Part 2 of custom lightweight Analytics with JavaScript and PocketBase, including creating a view collection and displaying view count securely in PocketBase"
datePublished: Tue May 09 2023 16:44:57 GMT+0000 (Coordinated Universal Time)
cuid: clhgi5kdk000509kx3yu8g6id
slug: custom-analytics-for-hashnode-part-2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683648197612/f37551dc-a9bf-4b67-b2a6-3dd52f69192b.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1683650598976/7a1d740f-34e2-4cd0-a00b-dc1c3d733c6f.png
tags: analytics, javascript, hashnode, pocketbase

---

*This is a brief update, continuing from* [*my previous post*](https://marcus.hashnode.dev/creating-a-custom-analytics-engine-for-my-hashnode-blog) *where I built a basic, custom analytics service to use on* [*my Hashnode blog*](https://marcus.hashnode.dev)*.*

In my final thoughts, I said:

> While I'm happy with what I created, there are still some things I'd like to work on. Having more security rules for the PocketBase collection is number one, however, I want to add an option that will display total views in the iframe. Doing so would require better security, and possibly a backend server.

Thankfully (for my budget) I was wrong about needing a backend server. [PocketBase](https://pocketbase.io/), alone, is enough to accomplish everything.

### Goals

* Include an option (like `show=true`) in the URL that will display total views.
    
* Make it so - for security - only the number of views (aka, entries in my PocketBase collection) can be queried, not any of the analytics data.
    

After a rough start trying to experiment with PocketBase rules, I asked the community for help with my problem. Below, is the reply from [@ganigeorgiev](https://github.com/ganigeorgiev) which saved me from resorting to creating a backend server.

%[https://github.com/pocketbase/pocketbase/discussions/2449#discussioncomment-5848751] 

## Updating our PocketBase

Turns out, in order to only allow viewing the number of results and not the content of said results, another collection is needed. However, not the same type of collection. I didn't know about this, but PocketBase has a ["view" collection type](https://pocketbase.io/docs/collections#view-collection) which is a collection that's content reflects another collection's (or multiple), with filters applied.

With a little modification to group counts by our `slug` (title of blog post), I ended up with this:

```sql
SELECT id, slug, COUNT(*) as total
FROM `basic_analytics`
GROUP BY slug
```

Your new collection settings in PocketBase should look something like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683649216812/86806f63-f150-4e44-8a16-cee1eaf6de6b.png align="center")

But there's still one more change to make, in the **API Rules** section, unlock both methods. This will allow **anyone with your PocketBase URL** to view ***only*** the number of views for each post.

After saving the new collection, if you have records in your `basic_analytics` collection, your new collection will look like this (I've named mine `view_analytics`):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683649376558/2ac43722-2c24-4677-b4c2-df71ca1e9480.png align="center")

Mine has some genuine, and some test data. It took me a while to work out all the issues.

## Updating our HTML/JavaScript

As our website will be embedded in Hashnode posts, I'm not going to focus on styling. <s>Also because I despise CSS</s>. So let's create a simple `div` , hidden at first, where we can display the number of views.

```xml
<div id='views' style='display: none'>
    <h1>Total Views: <span id='viewnum'>...</span></h1>
</div>
```

Next, we need a function to query the new collection and update the element. You may ask, **why am I doing this in the same function?** Well, the PocketBase SDK actually uses asynchronous functions. Previously, I didn't use `await` as our functions were synchronous - I just assume the promise will eventually resolve (which it seems to do so just fine).

This time, I'll write an `async` function, so we can get the value from PocketBase and then use it. However, the function still won't be awaited properly, and we will assume it resolves again. If someone would like to leave a comment with fully asynchronous code I would appreciate it, for now I think I'll stick to *"if it ain't broke, don't fix it"*.

### Getting view count in JavaScript

```javascript
async function getTotalViews(span_id) {
    // get matches for slug from URL params
    const count = await pb.collection('view_analytics').getFirstListItem(
        `slug="${params.get('slug')}"`
    );
    // update the element with the view count
    document.getElementById(span_id).innerText = count.total;
}
```

**If you're confused**, you really should [read my first post](https://blog.marcusj.tech/creating-a-custom-analytics-engine-for-my-hashnode-blog), which explains how we got to this point.

Next, let's update our main function - where we create the entry in our collection. Right after our `console.log` where we log the new record, and the data it contains, we'll add a simple `if` statement which will toggle the visibility of our view count `div`, and call our `getTotalViews` function.

```javascript
if (params.get('show')) {
    document.getElementById('views').style.display = 'block';
    getTotalViews('viewnum');
}
```

As I mentioned earlier, we can't use `await` as the function containing this code is synchronous. But, **that's it!**

At the end of this post, I have inserted some HTML with Hasnode, to use our analytics and display the view count. Here is the code:

```xml
<iframe src='https://<your-repl>.<your-replit-username>.repl.co/?slug=custom_analytics_pt2&show=true'></iframe>
```

### See it in action

<iframe src="https://basichashnodeanalytics.marcusweinberger.repl.co/?slug=custom_analytics_pt2&amp;show=true"></iframe>