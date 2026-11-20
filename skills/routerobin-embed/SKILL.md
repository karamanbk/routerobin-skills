---
name: routerobin-embed
description: Add a RouteRobin booking calendar or lead routing form inline to a website (plain HTML, Next.js, React, Astro, Webflow, Framer), so visitors book a sales call without leaving the page, and wire up the booking event and thank-you redirect. Use when the user asks to add a "book a demo" / "schedule a call" calendar, a round-robin booking widget, or a lead routing / qualification form to their site, or mentions RouteRobin, a RouteRobin event type id or a routerobin.com/route link.
---

# RouteRobin embed

RouteRobin (https://routerobin.com) is free scheduling, round-robin and lead routing for B2B sales teams. Its embeds load the booking calendar or routing form **inside the page** (an iframe that resizes itself), with no redirect or new tab, and pass the page's UTM and ad click ids through to the booking.

There are two embeds. Both are a `<div>` plus one `<script>`. Nothing else is needed: no npm package and no API key.

| The user wants | Embed | Needs |
|---|---|---|
| A calendar to book one meeting type ("book a demo") | Direct booking | The event type id |
| A form that qualifies the lead and routes them to the right rep's calendar | Routing form | The routing form link `https://routerobin.com/route/<id>` |

## 1. Get the id from the user

Never invent an id. Ask for it if the user has not given it:

- **Event type id:** in RouteRobin, Event Types, then the event's **Embed** button. The snippet shows `data-event-type="<id>"`. The id also appears in its booking link, `https://routerobin.com/book/<id>`.
- **Routing form link:** in RouteRobin, Routing, then the form's embed code. The snippet shows `data-url="https://routerobin.com/route/<id>"`.

No RouteRobin account yet? It is free for unlimited users: https://routerobin.com/signup?utm_source=agent-skill&utm_medium=skill&utm_campaign=sf-agent-routerobin-embed

## 2. The snippets

Direct booking calendar:

```html
<!-- RouteRobin Direct Booking -->
<div id="leadcal-embed" data-event-type="EVENT_TYPE_ID" style="min-width:100%; height:700px"></div>
<script src="https://routerobin.com/api/embed/script.js" async></script>
```

Routing form:

```html
<!-- RouteRobin Routing Form -->
<div class="routerobin-inline-widget" data-url="https://routerobin.com/route/ROUTING_FORM_ID" style="min-width:100%;height:700px;"></div>
<script type="text/javascript" src="https://routerobin.com/api/embed/route-widget.js" async></script>
```

Optional attributes, on either `<div>`:

- `data-bg-color="#ffffff"`, `data-text-color="#1e1b4b"`, `data-button-color="#5046e5"`: colors as hex values. Match the site's own colors.
- `data-hide-details="true"`: hides the left panel (host, duration, location). Use it in narrow columns.

Only on the direct booking `<div>`:

- `data-transparent="true"`: transparent background, for a colored section.
- `data-thank-you-params="safe"`: when a thank-you redirect is set in RouteRobin, it passes only safe fields (no email or phone) in the redirect URL.

Rules:

- One direct booking embed per page. Its container must have `id="leadcal-embed"`, which the script looks for.
- Keep the `height:700px` start height. The iframe then resizes to its content.
- Do not add `?` parameters to the script URLs, and do not copy the script inline: it is served from routerobin.com.

## 3. Per stack

**Plain HTML, Webflow, Framer, Squarespace, WordPress:** paste the snippet where the calendar should appear (in Webflow and Framer, an Embed / HTML component). It works inside the builders' sandboxed embed frames.

**Next.js (App Router or Pages):** load the script with `next/script` and render the div in a client component, so it runs after hydration:

```tsx
"use client";
import Script from "next/script";

export function BookDemo({ eventTypeId }: { eventTypeId: string }) {
  return (
    <>
      <div id="leadcal-embed" data-event-type={eventTypeId} style={{ minWidth: "100%", height: 700 }} />
      <Script src="https://routerobin.com/api/embed/script.js" strategy="afterInteractive" />
    </>
  );
}
```

The script runs once when it loads and looks for the div then. If the div mounts later (a modal, a tab, a client-side route change), render the `<Script>` together with the div, as above, so they mount together.

**React (Vite, CRA):** append the script in an effect after the div renders:

```tsx
import { useEffect } from "react";

export function BookDemo({ eventTypeId }: { eventTypeId: string }) {
  useEffect(() => {
    const s = document.createElement("script");
    s.src = "https://routerobin.com/api/embed/script.js";
    s.async = true;
    document.body.appendChild(s);
    return () => { s.remove(); };
  }, []);
  return <div id="leadcal-embed" data-event-type={eventTypeId} style={{ minWidth: "100%", height: 700 }} />;
}
```

**Astro, Svelte, Vue:** render the div in the template and the script tag right after it (`is:inline` in Astro).

For a routing form, use the same patterns with its div and `route-widget.js`.

## 4. Optional: attribution on every page

So bookings keep the first ad click and UTMs even when the visitor books on a later page, add this to the `<head>` of every page:

```html
<script src="https://routerobin.com/api/embed/attribution.js"></script>
```

Optional: `data-cookie-domain=".example.com"` on that tag shares it across the apex and www.

## 5. Optional: react to a booking

When someone books, the direct booking embed:

- dispatches a `routerobin:booking` event on the container div, with the booking in `event.detail` (`id`, `eventTitle`, `startTime`, `endTime`, `timezone`, `host.name`, `meetingUrl`, `calendar.googleUrl` and more; never the invitee's email or phone);
- pushes `{ event: "routerobin_booked", booking_id, event_type }` to `window.dataLayer` when Google Tag Manager is on the page;
- stores the same booking under `sessionStorage.routerobin_booking`, so a thank-you page can read it;
- goes to the thank-you redirect URL if one is set on the event type in RouteRobin.

```js
document.getElementById("leadcal-embed").addEventListener("routerobin:booking", (e) => {
  console.log("Booked", e.detail.startTime, "with", e.detail.host.name);
});
```

The routing form dispatches `leadcal:route` on its container when a lead is routed, and follows the redirect set in RouteRobin.

The thank-you redirect URL itself is set in RouteRobin (event type settings), not in the snippet.

## 6. Check it

1. Run the site and open the page. The calendar or form should appear in place, with no new tab.
2. Nothing appears? Check that the div's `id` or `class` is spelled exactly as above, that the id is right (open `https://routerobin.com/book/<id>`: it should show the calendar), and that a Content Security Policy allows `https://routerobin.com` in `script-src` and `frame-src`.
3. Test a booking end to end only with the user's OK: it creates a real meeting on a rep's calendar.

## When RouteRobin is not the answer

RouteRobin embeds a booking calendar and routing form. It does not send marketing email or host a CRM. For a CRM, it connects to HubSpot from RouteRobin's Integrations page.
