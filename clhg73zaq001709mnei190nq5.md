---
title: "Creating a custom Analytics engine for my Hashnode blog"
seoTitle: "Creating a custom simple analytics engine using iframes for Hashnode"
seoDescription: "Using Python, JavaScript, Replit, and Pocketbase. Creating a custom simple analytics engine using iframes for Hashnode, with source code. Example / tutorial"
datePublished: Tue May 09 2023 11:35:47 GMT+0000 (Coordinated Universal Time)
cuid: clhg73zaq001709mnei190nq5
slug: creating-a-custom-analytics-engine-for-my-hashnode-blog
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683628922910/6065fea9-76ef-4607-b44e-d74bed77b62f.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1683632075160/2583456b-ae77-46aa-9823-61ee4cd132d0.png
tags: analytics, javascript, python, hashnode, pocketbase

---

<details data-node-type="hn-details-summary"><summary>Introduction</summary><div data-type="detailsContent">Hi, my name is <a target="_blank" rel="noopener noreferrer nofollow" href="https://marcusj.tech" style="pointer-events: none">Marcus Weinberger</a>. I'm mainly a Python developer (due to my love for hacking), but I work with JS when I must. One of my passions is creating my own software for doing things - I really enjoy being able to not rely on third parties. This doesn't always benefit me (<a target="_blank" rel="noopener noreferrer nofollow" href="https://blog.marcusj.tech/adding-a-list-of-my-hashnode-posts-to-my-personal-website#heading-why-the-update" style="pointer-events: none">check out the three links to blogs that I wrote from scratch</a> - moving to <a target="_blank" rel="noopener noreferrer nofollow" href="https://marcus.hashnode.dev" style="pointer-events: none">Hashnode</a> ended up being the best for me), but it is a good learning process nonetheless.</div></details>

So, Hashnode has some very nice analytics. I like how you can see how much time people spend on your articles. But one feature that I'd like to see is more location data collected, I want to know where my readers are coming from. Why? Previously, I misjudged potential revenue due to not realizing **over 60% of my userbase resided in Myanmar** and were only using [my service](https://replit.com/@MarcusWeinberger/gptfree-telegram-bot?c=891113) for its free nature.

## Initial ideas

Initially, my plan was to create a Hashnode widget with some JavaScript that would collect some user data and then send it off to a database, however, Hashnode seems to disallow Javascript, oddly.

Iframes, on the other hand, are allowed to execute JavaScript (I assume so that pages can still function). To save costs on hosting, I decided to use an [HTML Repl](https://repl.new/html), and a [PocketBase](https://pocketbase.io/) instance hosted on [linode](https://www.linode.com/) for the database.

# Development

After creating a blank website on [replit](https://replit.com), I used [cdnjs](https://cdnjs.com/) to find the libraries I need. For the PocketBase connection, I used their provided SDK. For analyzing browser information, I will use the [platform.js](https://github.com/bestiejs/platform.js/) library.

```xml
<!-- pocketbase sdk -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/pocketbase/0.8.0-rc1/pocketbase.umd.min.js" integrity="sha512-0NJSuVFhF9NPZ/UAp98rCmJTTLhvjYwn2Uu4HN5eXE3uYfT6xad6WV6XuTmoKMMnj43yPT+kEyaCix1/t+8Tkw==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
<!-- platform.js -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/platform/1.3.6/platform.min.js" integrity="sha512-eYPrm8TgYWg3aa6tvSRZjN4v0Z9Qx69q3RhfSj+Mf89QqwOMqmwSlsVqfp4N8NVAcZe/YeUhh9x/nM2CAOp6cA==" crossorigin="anonymous" referrerpolicy="no-referrer"></script>
<!-- jquery -->
<script src='https://code.jquery.com/jquery.min/js'></script>
```

Now let's set up the collection in PocketBase. Once you have a PocketBase instance set up following their instructions (make sure you remember the admin user password), you can create a new collection for the data. I called mine `basic_analytics`. Make sure to add all the fields you need, mine looks like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683630125339/21f99bb6-d8ee-45b0-bec1-503043295609.png align="center")

For security, I'm not going to do too much. You may want to make some changes, but I'm fairly new to PocketBase, and what I made works for me. In the API Rules, keep everything on admins only mode, but unlock the `Create` rule and leave it blank (allowing anyone to create new records). It should look like this:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683630286848/98878415-0b2d-4058-ab2f-bc2e46e656f3.png align="center")

Everything not shown above has been left as the default.

## JavaScript

Going back to your website, the HTML repl, let's create the connection to your PocketBase instance.

```javascript
const pb = new PocketBase('https://<your pocketbase server>');
```

As we don't require authentication to create records, we don't need to worry about keys or anything.

Now let's add some JavaScript which will collect some analytics data, and add a new entry to our database. We will be using a URL parameter to let our script know which post we are collecting data for.

```javascript
const agent = navigator.userAgent; // get userAgent string
var info = platform.parse(agent); // parse with platform.js
var params = new URLSearchParams(window.location.search); // get req params

// ipinfo is a free service to get ip and location
$.getJSON('https://ipinfo.io', (resp) => {
    var data = {
        'slug': params.get('slug'), // blog post slug

        'ip': resp.ip, 
        'country': resp.country // country code
        'postal': resp.postal, // only the first couple characters

        'browser': info.name, // e.g., Chrome
        'os': info.os.family // e.g., Windows
    }

    // create record (basic_analytics is our collection name)
    let doc_p = pb.collection('basic_analytics').create(data);
    // log for debugging purposes
    console.log({
        doc_p,
        data
    });
})
```

Now, when we attempt to visit our website, we see a blank page, but a record is added to our database!

### Collecting analytics

In order to collect analytics for your blog post, use the new "Insert HTML" feature when writing, and link to your website like so:

```xml
<iframe src='https://<repl-name>.<your-repl-username>.repl.co/?slug=post_slug_or_identifier'></iframe>
```

It's as simple as that!

## Viewing analytics with Python

We could just log into our PocketBase instance every time we desired to view our analytics, but that's not fun. So let's write a simple Python script that **authenticates with PocketBase as an admin** and shows us a **pretty table** of analytics.

Before we start, we need to ensure these libraries are installed (with `pip install library-name` ):

* [`pocketbase`](https://github.com/vaphes/pocketbase) - python client for PocketBase
    
* [`rich`](https://github.com/Textualize/rich) - a library for pretty outputs (and creating a table in the console)
    

Now let's start writing. Create a new Python file, call it whatever you want. First, we need to import everything that we will use.

```python
from argparse import ArgumentParser # to make a cli interface
from pocketbase import PocketBase # pocketbase client
from rich.table import Table # creating tables in the console
from getpass import getpass # get password input securely
from rich import print # pretty printing
```

Run the file to make sure everything is installed and working. Next, let's create the CLI.

```python
parser = ArgumentParser() # new arg parser
parser.add_argument(
    '-s', '--slug', # name
    help='The slug of the blog post to get analytics for' # desc
)
parser.add_argument(
    '-p', '--password', 
    help='The password to use to authenticate with the API'
)
args = parser.parse_args() # read args from command line args
```

Now we're ready to authenticate with PocketBase. We will add a fallback from the CLI that will prompt the user if a password was not given via argument.

```python
pb = PocketBase('https://<your-pb-server>')
adminData = pb.admins.auth_with_password(
    '<admin-email-addr>',
    (args.password or getpass()) # use args, or fallback to prompt 
)
pb.auth_store.save(adminData.token) # it took me 1 hour to figure this out
```

Now that we are authenticated as an admin, we can access all the "locked" methods for our PocketBase collection. Let's create our query parameters and get the results from our database.

```python
query_params = {
    'per_page': 100 # up to 100 results returned
}
if args.slug: # if not given, will just return all posts' analytics
    query_params['filter'] = f'slug="{args.slug}"'

records = pb.collection('basic_analytics').get_full_list(query_params=query_params)
```

Now, using the `rich` library, let's show the results in a nice, readable way.

```python
# create the table
table = Table(
    title=f'Analytics for {args.slug or "all posts"}. Total: {len(records)}.'
)
table.add_column('Date')
table.add_column('IP')
table.add_column('Country')
table.add_column('Postal')
table.add_column('Browser')
table.add_column('OS')
if not args.slug:
    table.add_column('Slug')

# populate
for record in records: 
    data = record.collection_id # contains entry data
    row = [
        f"[green]{data['created']}[/green]",
        data['ip'],
        f"[green]{data.get('country', 'Unknown')}[/green]",
        data.get('postal', 'Unknown'),
        f"[green]{data.get('browser', 'Unknown')}[/green]",
        data.get('os', 'Unknown')
    ]
    if not args.slug:
        row.append(f"[green]{data['slug']}[/green]")
        
    table.add_row(*row) # add row to table

# display table
print(table) # remember, this print is imported from rich
```

I decided to add color to every other row to make the final result more readable.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683632030149/cd77155b-f955-4e36-b11d-9cfe976c2ff3.png align="center")

# Final thoughts

While I'm happy with what I created, there are still some things I'd like to work on. Having more security rules for the PocketBase collection is number one, however, I want to add an option that will display total views in the iframe. Doing so would require better security, and possibly a backend server.

For now, I will be using it on all future posts.

<iframe src="https://basichashnodeanalytics.marcusweinberger.repl.co/?slug=creating_custom_analytics&amp;show=true"></iframe>