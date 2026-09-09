# ctfg-guidefinder

A lightweight widget that recommends relevant [Civic Tech Field Guide](https://civictech.guide) [categories](https://app.civictech.guide/categories), [issues](https://app.civictech.guide/issues), [communities](https://app.civictech.guide/communities), and locations for any piece of text.

Pass it a paragraph, a page summary, or a theory of change — it returns up to 3 matching entries from the CTFG directory with links to explore further.

---

## Quick start

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/Civic-Tech-Field-Guide/ctfg-guidefinder@1/guidefinder.css">
<script src="https://cdn.jsdelivr.net/gh/Civic-Tech-Field-Guide/ctfg-guidefinder@1/guidefinder.js"></script>

<div id="ctfg-recommendations" hidden></div>

<script>
  GuiFinder.show(
    document.getElementById('ctfg-recommendations'),
    'Your text here — a page summary, a description of your project, etc.'
  );
</script>
```

### Choosing a version

`@1` tracks the newest `v1.x` release, so bug fixes and new result types reach your page without an HTML edit. A `v2.0.0` will never arrive this way, which is what keeps the range safe to follow.

Never use `@main`. It serves whatever was pushed minutes ago, reviewed or not, and jsDelivr tells browsers to hold it for seven days, so it is neither safe nor fast.

To freeze the code instead, pin an exact tag and add an [`integrity`](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) attribute. Both go together: `integrity` only works against a URL whose bytes can never change, so it rules out `@1`.

```html
<script src="https://cdn.jsdelivr.net/gh/Civic-Tech-Field-Guide/ctfg-guidefinder@v1.0.0/guidefinder.js"
        integrity="sha384-..." crossorigin="anonymous"></script>
```

```sh
openssl dgst -sha384 -binary guidefinder.js | openssl base64 -A
```

Updates take up to 12 hours to clear the jsDelivr edge and up to seven days to clear a browser that already has the file. Neither `@1` nor an exact pin is instant.

Or skip the CDN entirely — see [Self-hosting](#self-hosting-the-widget) below.

The container stays hidden until results arrive. If no relevant matches are found, or the daily request cap is reached, it stays hidden.

---

## API

### `GuiFinder.show(container, text [, options])`

| Parameter | Type | Description |
|---|---|---|
| `container` | `Element` | DOM element to render results into |
| `text` | `string` | Text to match against the CTFG directory (max 5000 chars) |
| `options.limit` | `number` | Max results to show, 1–3 (default: 3) |
| `options.heading` | `string` | Override the default heading text |

Results are lazy-loaded when the container scrolls into view (200px margin), so it's safe to call on page load without delaying rendering.

---

## Public API endpoint

The widget calls a public endpoint hosted by the Civic Tech Field Guide:

```
POST https://curator.civictech.guide/api/recommend
Content-Type: application/json

{
  "text": "your text here",
  "limit": 3
}
```

### Response

```json
{
  "categories": [
    {
      "name": "Civic Engagement",
      "description": "Tools and platforms for civic participation...",
      "softrUrl": "https://civictech.guide/...",
      "type": "category"
    }
  ],
  "issues": [...],
  "communities": [...],
  "locations": [
    {
      "name": "Lisbon",
      "description": "Projects, tools, and organizations based in Lisbon.",
      "softrUrl": "https://ctfg.softr.app/place/?recordId=recrdWsd1k74PkZ6Z",
      "type": "location"
    }
  ]
}
```

Each result has `name`, `description`, `softrUrl`, and `type` (`"category"`, `"issue"`, `"community"`, or `"location"`).

### Locations

Locations are country-level and city-level entries from the CTFG Locations directory, linking to that place's page. They are matched more conservatively than the other three types: a location is only returned when the text is substantially about that place, and at most one location appears in a result set. A passing mention, a conference venue, a byline, or a nationality used as an adjective does not qualify. Most texts get no location at all.

If the daily request cap is reached, the response includes `"dailyCapReached": true` and empty arrays.

### Rate limits

- 10 requests per minute per IP
- 400 requests per day total across all callers
- Results are cached for 24 hours, so repeated identical queries are free

---

## Self-hosting the widget

Copy `guidefinder.js` and `guidefinder.css` into your project and update the script/link tags to point to your local copies. No build step required.

---

## Releases

Pushing a change to `guidefinder.js` or `guidefinder.css` on `main` cuts the next patch tag automatically and clears the jsDelivr edge for the floating URLs. See `.github/workflows/tag-release.yml`. Tag a `v2.0.0` by hand for a breaking change, so `@1` embeds stay on the old major.

---

## License

MIT
