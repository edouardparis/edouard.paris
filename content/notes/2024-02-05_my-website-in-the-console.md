+++
title = "My website in the console"
date = "2024-02-05"
slug = "my-website-in-the-console"
topics = ["caddy", "hugo"]
+++

# My website in the console

I like ascii art, I like hacker zine like [Phrack](http://phrack.org/) or
[tmpout.sh](https://tmpout.sh). The first version of my website was some sort
of terminal window in the browser and I wanted my personal website to
have this kind of text design with ASCII art and diagrams.
Naturally I thought it would be nice to have a version of the website readable
in the console, like a `man(1)` page.

My static site generator is [Hugo](https://gohugo.io/). It exists for quite
some time and keeps evolving with a lot of functionalities. Content and
templates are two core parts of the architecture of Hugo. The content you wrote
is parsed from markdown files and written in HTML page through usage of
templates. But it is also possible to change the format of the final generated
file. For example, from a list of blog posts Hugo is able to create for each
post a HTML page and for the list a HTML index page with a RSS feed.
The outputs formats are **HTML** and **RSS (xml)** but can be **JSON** or
plain text.

Here a small guide to use this Hugo feature with Caddy to have blog posts
readable in the console.


 ## 1. Configuring Hugo for Plain Text Output

First define the plain text format as an output format in the theme's `config.toml`
and add it to the list of outputs for the content pages.

{{< code lang="toml" >}}
[outputFormats.Txt]
name = "txt"
baseName = "index"
mediaType = "text/plain"
isPlainText = true

[outputs]
home = ["HTML", "txt"]
page = ["HTML", "txt"]
section = ["HTML", "txt", "RSS"]
{{</ code >}}


## 2. Creating TXT Templates in Hugo

Then, create **TXT** templates following the
[lookup order of Hugo](https://gohugo.io/templates/lookup-order/).
With a minimal configuration, it requires at least the `layouts/index.txt`
for the home page, a `layouts/_default/section.txt` and
`layouts/_default/single.txt` templates.

Hugo uses Go programming language’s `html/template` and `text/template`
libraries as the basis  for the templating. Plain text is de facto
supported with all the nesting features and variables support.

In order to display the post content and keep the markdown elements, use
the template variable `{{ .RawContent }}` or `{{ .RenderShortcodes }}` inside the templates.

Here an example of `layouts/_default/section.txt`:
{{< code lang="html" >}}
{{ partial "head.txt" . }}
{{ .RawContent }}
{{ range .Pages.GroupByDate "2006" }}[ {{ .Key }} ]
    {{ range .Pages }}
        {{ .Date.Format "Jan 02" }} {{ .Title }}
            ↪ {{ .RelPermalink }}
    {{ end }}
{{ end }}
{{ partial "foot.txt" . }}
{{</ code >}}

Add this piece of code to inform visitor of the nerdy feature:
{{< code lang="html" >}}
<p>Read this page in your terminal with the command:</p>
<code>$ curl -L example.com{{ strings.TrimSuffix "/" (.RelPermalink) }} | less</code>
{{</ code >}}

After running `hugo` command, you can see in the `public` folder the generated
pages in each output formats.

{{< code lang="text" >}}
public/
├── blog
│   ├── first-post
│   │   ├── index.html
│   │   └── index.txt
│   ├── index.html
│   ├── index.txt
│   ├── index.xml
...
{{</ code >}}

Now, the last thing to do is to redirect visitors to the plain text page when they
are using [Curl](https://curl.se) in the terminal to read the post.

## 3. Serving Plain Text Pages with Caddy

I personally use [Caddy](https://caddyserver.com), a HTTP/2 web server with
automatic HTTPS.  Configuration is quite easy and the service installation
on a NixOS running server can be done in few lines.
The Caddy file supports request matching and rewrite directives.
We tell it to rewrite the URL of any request to our website that has Curl in the
`User-Agent` field to point to the plain text version of the post.

{{< code lang="caddyfile" >}}
example.com {
    @isCurl header User-Agent *curl*
    handle @isCurl {
        rewrite * {path}/index.txt
    }

    root * /var/www/example.com
    file_server
}
{{</ code >}}

That's it. Is it useful or more readable ? Maybe not. Is it fun ? Yes.
