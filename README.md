# tilt-book

A 3D book that tilts toward your cursor. The motion is lifted from `FlippableTiltCard` in
[`nextjs-css-agent-components`](https://github.com/internet-development/nextjs-css-agent-components)
(this project uses that repo as its base: Next.js pages router, vanilla CSS Modules with native
nesting, the same `global.css` tokens and path aliases, no Tailwind / SASS / CSS-in-JS).

The cover art is fetched at render time from the **Open Library Covers API** — the demo book is
_The Singularity Is Near_ by Ray Kurzweil.

## Run

```bash
npm install
npm run dev
```

Dev server on [http://localhost:10001](http://localhost:10001) (port 10001 so it can run alongside
the component reference repo on 10000).

- `npm run build` — production build, typechecked (`ignoreBuildErrors: false`)
- `npm run format` — Prettier across `**/*.{tsx,ts,js,css,json,md}`

## Layout

```
common/open-library.ts            Covers API URL builder + the Singularity Is Near record
elements/visuals/TiltBook.tsx     The component (Tier 1: imports no other in-repo component)
elements/visuals/TiltBook.module.css
pages/index.tsx                   Demo page
global.css / animations.css       Copied from the base repo (themes, type scale, fonts)
```

## How the tilt works

Same shape as `FlippableTiltCard`: a `mousemove` listener maps the cursor's position inside the
element's bounding box to a rotation, kills the transition while tracking so the book follows the
cursor 1:1, then restores a `400ms ease` transition on `mouseleave` for the ease-out back to rest.

Two things differ from the card:

1. The rotation drives a `transform-style: preserve-3d` box with **six faces** (front cover, back
   cover, spine, fore edge, head, tail) instead of a flat card, so the book has real thickness.
2. It rests at `rotateX(4deg) rotateY(18deg)` rather than flat, so the spine stays visible — a book
   dead-on to the camera reads as a poster.

## Component API

`<TiltBook />` takes all-optional props:

| Prop             | Default                                  | Notes                                      |
| ---------------- | ---------------------------------------- | ------------------------------------------ |
| `title`          | `The Singularity Is Near`                | Also used on the spine and the alt text    |
| `author`         | `Ray Kurzweil`                           | Spine + alt text                           |
| `coverSrc`       | Open Library cover id `400518`, size `L` | Any image URL                              |
| `tilt`           | `20`                                     | Max degrees of rotation away from rest     |
| `restingRotateX` | `4`                                      | Resting pitch                              |
| `restingRotateY` | `18`                                     | Resting yaw — positive keeps the spine out |
| `scale`          | `1.06`                                   | Scale applied while the cursor is over it  |
| `style`          | —                                        | Spread onto the outer wrapper              |

Book dimensions are CSS custom properties on `.spacing` (`--book-width`, `--book-height`,
`--book-depth`), overridden at the 768px breakpoint. The defaults (300 × 455) match the 330 × 500
aspect of the Open Library cover.

## Open Library Covers API

```
https://covers.openlibrary.org/b/$key/$value-$size.jpg
```

`$key` is `id`, `olid`, `isbn`, `oclc`, or `lccn`; `$size` is `S`, `M`, or `L`. Lookups by `id` and
`olid` are not rate limited — the others are capped at 100 requests per IP per 5 minutes, so
`common/open-library.ts` defaults to the cover id.

The Singularity Is Near: cover id `400518`, edition `OL7641290M`, ISBN `0670033847`,
work <https://openlibrary.org/works/OL1958850W>.

If the request fails, `TiltBook` swaps in a typeset fallback cover instead of a broken image.

## Dropping this on a website

`TiltBook.tsx` + `TiltBook.module.css` are self-contained — copy both files, plus
`common/open-library.ts` if you want the URL helper. The only outside dependencies are the
`--theme-*`, `--type-scale-*`, and `--font-family-mono` custom properties from `global.css`.
