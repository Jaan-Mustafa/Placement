# How Users Hit CDN URLs

The user doesn't know — your server/code tells them the CDN URL.

## How It Works

When your server sends back an HTML page or API response, it includes CDN URLs directly in the response.

### Example

```html
<!-- Instead of this (your origin server) -->
<img src="https://myserver.com/images/photo.jpg">

<!-- Server returns this (CDN URL) -->
<img src="https://cdn.cloudflare.com/mysite/images/photo.jpg">
```

User's browser sees the CDN URL → hits CDN directly → gets the image.

## Who Sets the CDN URL?

Developers configure it. When a file is uploaded, the code stores the CDN URL in the DB.

```
Developer uploads photo.jpg
  → stored on CDN (e.g. Cloudflare)
  → CDN gives back a URL: cdn.cloudflare.com/photo.jpg
  → you save that URL in your DB

User requests a post
  → server fetches post from DB (includes cdn.cloudflare.com/photo.jpg)
  → server sends that URL in the response
  → browser hits CDN directly for the image
```

## Real World Example — YouTube

```json
{
  "videoId": "abc123",
  "title": "My Video",
  "thumbnail": "https://i.ytimg.com/vi/abc123/thumbnail.jpg"
}
```

`i.ytimg.com` is YouTube's CDN — user's browser hits that directly, never YouTube's main server.

**TL;DR:** User doesn't know or care — your server embeds the CDN URL in the response. Browser just follows whatever URL it's given.
