# Table of Content

- [Table of Content](#table-of-content)
- [Introduction](#introduction)
- [Status of this Document](#status-of-this-document)
- [Examples](#examples)
- [Goal](#goal)
- [JSON Structure](#json-structure)
  - [Top-level](#top-level)
  - [Stories](#stories)
- [MIME Type](#mime-type)
- [Extensibility model](#extensibility-model)
- [Recommendations](#recommendations)
  - [Number of stories](#number-of-stories)
  - [Conditional GET](#conditional-get)
  - [Cross-origin resource sharing (CORS)](#cross-origin-resource-sharing-cors)
  - [Discovery](#discovery)
- [Relation with RSS/ATOM/JSONFeed](#relation-with-rssatomjsonfeed)
- [Credits](#credits)
- [License](#license)

# Introduction

This document proposes a new syndication format ([Yet to be named][0]), similar - but **NOT** replacement - to [RSS][1], [Atom][2], and [JSONFeed][3].

# Status of this Document

This proposal is still in a very early stage and should be considered as _draft_. Before reaching version 1.0, changes may be introduced without prior notice.

# Examples

Here’s a simple example:

```json
{
  "version": 1.0,
  "link": "https://example.org/",
  "feed_url": "https://example.org/stories.json",
  "language": "en",
  "author": {
    "name": "John Due",
    "avatar": "https://example.org/avatar.png"
  },
  "stories": [
    {
      "id": "987654321",
      "text": "Look at this cool photo of me",
      "attachments": [
        {
          "url": "https://example.org/second-item/images/me.png",
          "type": "image/png"
        }
      ],
      "link": "https://example.org/second-item"
    },
    {
      "id": "987654322",
      "text": "Hello world",
      "link": "https://example.org/initial-post",
      "alternates": ["https://twitter.com/username/status?id=1231232"]
    }
  ]
}
```

# Goal

Our goal is to support authors to publish short posts anywhere without being locked in a single platform while providing an easy way for others interested to follow their publishing.

Typically we aim to support these kinds of content:

- Short (more or less a tweet-size ), plain text only, untitled, posts.
- Posts with only images or videos.
- A mix of above.

# JSON Structure

## Top-level

**Required:**

- `version` (number) is the version number of the format the feed uses. This should appear at the very top, though we recognize that not all JSON generators allow for ordering.

- `feed_url` (string) is the absolute URL of the feed and serves as the unique identifier for the feed.

- `author` (object) specifies the feed author:

  - `name` (string) is the author’s name. This attribute is **required**.

  - `avatar` (string) is the URL for an image for the author. It should be square and relatively large — such as 512 x 512 — and should use transparency where appropriate, since it may be rendered on a non-white background. This attribute is **optional**, but highly recommended.

- `language` (string) An [RFC 5646][10] string that specifies the language that all or the majority of stories are written in. It should be used as a default value when the `language` attribute of a story is missing.

**Optional:**

- `link` (string) is the absolute URL of the resource that the feed describes. This resource may or may not actually be a “home” page, but it should be an HTML page.

* `next_url` (string) is the absolute URL of a feed that provides the next n items (for pagination), where n is determined by the publisher. `next_url` must not be the same as `feed_url`, and it must not be the same as a previous `next_url` (to avoid infinite loops).

## Stories

`stories` is an array of stories, and it's required. A story includes:

**Required:**

- `id` (string) is unique for that story for that feed over time. If a story is ever updated, the `id` should be unchanged. New stories should never use a previously-used `id`. Ideally, the `id` is the full URL of the resource described by the feed, or a UNIX timestamp for the time of creation since both options make great unique identifiers.

- `published` (string) specifies the date in [ISO 8601][4] format. (Example: `20190214T102052Z`.)
- `attachments` (array) lists one or more attachements:

  - `url` (string) specifies the absolute URL of the attachment. This attribute is **required**

  - `type` (string) specifies the type of the attachment, such as “audio/mpeg.” This attribute is **required**

  - `title` (string) a title of the attachment. We present, it should be used to enhance accessibility (e.g. `aria-label`). This attribute is **optional**.

  - `size` (number) specifies the size of the file in bytes. This attribute is **optional**.

  - `duration` (number) specifies how long it takes to listen to or watch when played at normal speed. Duration should be specified in seconds. This attribute is **optional**.

**Optional:**

- `link` (string) is the absolute URL of the resource described by the story.

- `text` (string) is a plain text (i.e. non-HTML) representation of the story. This attribute is considered **required** when no attachment is included.

- `updated` (string) specifies the modification date in [ISO 8601][4] format.

- `until` (string) specifies the expiry date in [ISO 8601][4] format in case the story is expirable. When a story is expired, the reader software should assume that the URLs specified in `link` and `alternates` are no longer accessible.

- `language` (string) specifies the language of this story in [RFC 5646][10] format. It's only needed if the value is different than the Top-level `language` attribute.
- `alternates` (array) an array of strings specifies alternatives absolute URLs for the same story in a different platform. For example, a publisher may republish some of their notes (originally on their website, as indicated by `link`) to Twitter, in this case, they include a link of that specific tweet so people can reach and engage with them.

# MIME Type

The [Yet to be named][0] feed must always be served using `application/[PREFIX]+json` MIME type.

# Extensibility model

TODO . See [#2](https://github.com/z0al/naming-is-hard/issues/2)

# Recommendations

## Number of stories

The specification doesn't specify a maximum number of stories in a feed, it's up to the publisher to determine the limit. If you need to provide older items, consider using pagination via `next_url`.

## Conditional GET

Publishers are highly recommended to support [Conditional GET][5], to limit the impact on their system when feed readers poll for changes. For static websites, services like [Netlify][6] support this out of the box.

## Cross-origin resource sharing (CORS)

It's highly recommended to enable [CORS][9] on feed hosts so that it can be consumed from JavaScript hosted on a static website. E.g:

```http
Access-Control-Allow-Origin: *
```

## Discovery

A web page should include a `link` tag that specifies the location of the [Yet to be named][0] Feed. Like this:

```html
<link
  rel="alternate"
  title="My stories"
  type="application/[PREFIX]+json"
  href="https://example.org/stories.json"
/>
```

This is the same as the feed discovery mechanism used for RSS, Atom, and JSONFeed.

The `title` attribute is optional but can be useful to differentiate multiple feeds — for instance, there might be a feed for all posts and a feed for a specific category (e.g. Tech, Politics)

# Relation with RSS/ATOM/JSONFeed

Again, [Yet to be named][0] doesn't intend to replace any of RSS, ATOM, or JSONFeed. It's meant to serve very specific cases where the use of RSS, ATOM or JSONFeed is either too much or just doesn't fit well.

# Credits

Inspired by the awesome work of [RSS][1], [ATOM][2], and [JSONFeed][3].

Authored by:

1. Ahmed T. Ali (https://ahmed.sd)

Reviewed by:

1. TODO

# License

![Creative Commons Attribution 4.0 International][8]

Except where otherwise noted, This work is licensed under a [Creative Commons Attribution 4.0 International][7] license.

[0]: https://github.com/z0al/naming-is-hard/issues/1
[1]: http://cyber.harvard.edu/rss/rss.html
[2]: https://tools.ietf.org/html/rfc4287
[3]: https://jsonfeed.org/version/1
[4]: https://en.wikipedia.org/wiki/ISO_8601
[5]: https://fishbowl.pastiche.org/2002/10/21/http_conditional_get_for_rss_hackers/
[6]: https://netlify.com
[7]: http://creativecommons.org/licenses/by/4.0/
[8]: https://i.creativecommons.org/l/by/4.0/88x31.png
[9]: https://en.wikipedia.org/wiki/Cross-origin_resource_sharing
[10]: https://tools.ietf.org/html/rfc5646
