# Image-Tile Reconstruction From a Public Reference Site

A public reference site exposed per-record overlay maps, the metadata behind each overlay, and linked spreadsheets. The maps had to be captured at full quality, not as screenshots.

## What I did
Built a scraper that pulls every record's overlay as a reconstructed image at native quality, captures the per-overlay metadata and bounds, and outputs structured data linked to the spreadsheets.

## Approach
The overlay images are tiled. Instead of capturing the rendered page, I pulled the raw tiles and reassembled them so the output keeps maximum resolution and the associated metadata: bounds, value range, zoom, and projection. The full run took continuous local rendering because reassembling every record's tiles is compute-heavy on local hardware. While scraping I found the site withholds certain values for arbitrary record and location pairs, which is expected given how the underlying data behaves physically, so I documented where those values are and are not available rather than shipping silent gaps. I also separated records that carry an attached sub-table from those that do not, so the secondary scraper's coverage matched reality. A related target was a much larger archive in the millions of pages, where a portal forces a keyword before it returns results. I scoped a way to enumerate the full dataset around that restriction.

## Stack
Python for scraping and tile reassembly, image reconstruction at native resolution, structured metadata and spreadsheet linkage, long-running local renders.

## Result
Full-quality overlay imagery plus structured metadata for the record set, delivered with the data-availability limits documented rather than hidden. The same approach carried to a multi-million-page archive target, where I worked out how to enumerate the full dataset past a forced-keyword search.
