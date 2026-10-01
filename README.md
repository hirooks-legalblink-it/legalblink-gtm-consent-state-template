# LegalBlink Consent State

A Google Tag Manager variable template that tells your triggers which consent categories a
visitor granted in the [LegalBlink](https://legalblink.it) CMP.

It reads the `lb_csc` consent cookie, the same source the LegalBlink CMP tag template uses, and
returns the granted categories as a comma-separated list, always in this order:

```
necessary,preferences,analytics,marketing
```

## Requirements

- The **LegalBlink CMP** tag template from the Community Template Gallery, on the
  **Consent Initialization - All Pages** trigger, with your License ID.
- The categories you want to use must be visible in your LegalBlink cookie policy. A hidden
  category is never granted.

## Installation

1. In GTM open **Templates → Variable Templates → Search Gallery**, look for
   **LegalBlink Consent State** and add it to your workspace.
2. Open **Variables → New** and choose **LegalBlink Consent State**. There is nothing to
   configure. Name it, for example, `LegalBlink - Consent State`.

## Output

| Visitor's choice | Value |
|---|---|
| No valid choice yet: no cookie, unreadable cookie, banner not answered | `""` (empty string) |
| Rejected everything | `necessary` |
| Analytics only | `necessary,analytics` |
| Marketing only | `necessary,marketing` |
| Accepted everything | `necessary,preferences,analytics,marketing` |

- The tokens are `necessary`, `preferences`, `analytics` and `marketing`. No token is part of
  another, so a "contains" condition is exact. Future versions will keep this rule.
- `necessary` means that the visitor has made a choice. Strictly necessary tags do not depend on
  consent: do not gate them on it.
- In the LegalBlink banner, `preferences` is the category shown as "Personalizzazione Google".

## Using it in triggers

Use **contains** conditions:

- `{{LegalBlink - Consent State}}` contains `marketing`
- `{{LegalBlink - Consent State}}` contains `analytics`
- `{{LegalBlink - Consent State}}` contains `preferences`

Avoid "equals": it stops matching as soon as another category changes.

For a non-Google tag such as the Meta Pixel, one Custom Event trigger covers both returning
visitors and new choices:

- Event name: `^(gtm\.js|legalblink_consent_update)$`, with **Use regex matching**
- This trigger fires on: **Some Custom Events**, `{{LegalBlink - Consent State}}` contains
  `marketing`
- In the tag, **Advanced Settings → Tag firing options: Once per page**

Returning visitors already have the cookie when the page loads (`gtm.js`). After every choice
the LegalBlink CMP pushes `legalblink_consent_update` to the data layer, and the cookie is
already written when that event arrives. The event part requires the default `dataLayer` name.

Google tags (Google tag, GA4, Google Ads) do not need these conditions: they follow Google
Consent Mode, which the LegalBlink CMP tag template sets for them.

When a visitor withdraws consent, the following events no longer satisfy the condition.
Requests sent before the change cannot be undone, and a script already loaded on the page keeps
running until the next page load unless you tell it: for the Meta Pixel,
`fbq('consent', 'revoke')`.

## How the value is computed

- The template reads `lb_csc` with `getCookieValues('lb_csc', false)`. The CMP stores it as raw
  JSON: `{"level":[...],"version":"2"}`.
- A value is valid only if it is a JSON object whose `level` is an array containing
  `necessary_cookies`. Anything else gives `""`: a missing cookie, an empty value, malformed JSON,
  or a `level` that is not an array.
- Unknown entries in `level`, and any other field, are ignored: they never grant a category.
- If the browser exposes more than one `lb_csc` cookie, a category counts only if every cookie
  grants it, and one invalid cookie gives `""`. This can happen after moving from the older
  LegalBlink script on a host with four or more labels. An old cookie can never hide a
  revocation.
- Nothing is cached. The cookie is read every time the variable is evaluated, so a change on the
  same page is seen at the next event.

| `level` entry | token |
|---|---|
| `necessary_cookies` | `necessary` |
| `first_party_tracking_cookies` | `preferences` |
| `third_party_stats_cookies` | `analytics` |
| `third_party_adv_cookies` | `marketing` |

## Permissions

The template reads the value of one cookie, `lb_csc`, and nothing else.

## Support

Report problems on this repository's [Issues](../../issues) page.

## License

Apache License 2.0, see [LICENSE](LICENSE).
