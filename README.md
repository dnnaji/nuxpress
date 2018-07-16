# nuxpress: Minimalist Plain Text Blogging

**nuxpress** is the result of me reading through [VuePress][1]'s source code for
a week. It doesn't have blogging support yet, so I set out to try and cook 
something up with it. I learned a lot reading through Evan's code, but my 
feeling is that VuePress goes to great lengths to replicate [Nuxt][2]'s 
functionality for automatically setting up and launching a Vue app.

[1]: https://vuepress.vuejs.org/
[2]: http://nuxtjs.org/

While I see the value of having `vuepress` as a standalone CLI tool and 
everything it does that Nuxt doesn't (e.g., all SEO-friendly publishing tweaks), 
for me really its **golden feature** is streamlining Markdown in Vue files.

Now, I **really like** Nuxt's code organization standards. Having a blog as
a Nuxt app makes sense, as it would give me total freedom for customization.
VuePress's `eject` command gives you a similar functionality, but you miss
the multitude of plugins and Nuxt-oriented solutions out there.

So instead of trying to add blogging support to VuePress, I decided to add 
Markdown blogging support to a Nuxt app with the minimum amount of code 
and conventions I could possibly manage.

## Where the magic happens

<table>
<tr>
<td><b>nuxt.config.js</b></td>
<td>Supercharged with functions that load <code>.entry</code> files from a 
<code>entries/</code> directory. Generates RSS and Atom feeds using
<code>.template</code> files under <code>feeds/</code>.</td>
</tr>
<tr>
<td><b>plugins/entryLoader.js</b></td>
<td>Uses an index <code>entries.json</code> file (generated by 
<code>nuxt.config.js</code>) to apply the Markdown transformation and inject 
<code>$entries</code> and <code>$permalinks</code> in Nuxt's <code>app</code>.</td>
</tr>
<tr>
<td><b>pages/index.vue</b></td>
<td><code>asyncData()</code> makes <code>$entries</code> available to 
the template. Entries are automatically ordered by dates descending.</td>
</tr>
<tr>
<td><b>pages/entry.vue</b></td>
<td><code>asyncData()</code> makes <code>article</code> available to 
the template, accessed via slug in the <code>$permalinks</code> hash.</td>
</tr>
</table>

If the loader encounters a directory under `entries`, it checks inside for an
`.entry` file and copies all other files to `/static/entries/`, so you can 
reference any assets in your `.entry` files

## Files that need customization

The `domain` constant in `nuxt.config.js`, then:

<table>
<tr>
<td><b>pages/*</b></td>
<td>Where <code>index.vue</code>, <code>entry.vue</code> and 
<code>archive.vue</code> live. As long as you understand and preserve the logic,
layout and CSS can be modified as you like.</td>
</tr>
<tr>
<td><b>feeds/*</b></td>
<td>Lodash templates for RSS and Atom feeds.</td>
</tr>
<tr>
<td><b>layouts/default.vue</b></td>
<td>Nuxt's default layout, set up to use <code>components/LeftColumn.vue</code></td>
</tr>
<tr>
<td><b>components/LeftColumn.vue</b></td>
<td>The left column in the default two-column layout. This can be removed 
altogether and replaced by whatever layout structure you're using.</td>
</tr>
</table>

## Improved Markdown links

I love Markdown, but the fact that you need to use a unique identifier for 
link references makes things to hard to maintain.

Here's an example:

```
To use it you need to [write a locustfile][1], which is just a Python file with
at least one [Locust subclass][2]. Each Locust object specifies a list of tasks
to be performed during the test, represented by a [`TaskSet`][3] class.

[1]: http://docs.locust.io/en/latest/writing-a-locustfile.html
[2]: http://docs.locust.io/en/latest/api.html#locust-class
[3]: http://docs.locust.io/en/latest/api.html#taskset-class
```

if you remove the second link, you get an aesthetically unpleasing unordered 
list. Plus you have to keep track of references, even if you use slugs as link 
identifiers instead of numbers. In `nuxpress`, all you have to do is:

```
To use it you need to [write a locustfile][], which is just a Python file with
at least one [Locust subclass][]. Each Locust object specifies a list of tasks
to be performed during the test, represented by a [`TaskSet`][] class.

[]: http://docs.locust.io/en/latest/writing-a-locustfile.html
[]: http://docs.locust.io/en/latest/api.html#locust-class
[]: http://docs.locust.io/en/latest/api.html#taskset-class
```

And it will automatically link ordered references. Use this if you're ok with
not ever having two references to the same link in paragraphs, and want a 
streamlined way of adding and editing them. Otherwise, regular Markdown link
references will work as expected.

