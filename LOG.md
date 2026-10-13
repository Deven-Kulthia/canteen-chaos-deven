## CC-01 — "The search suggestions are behind everything"

**Reproduced:** I opened the site on localhost:3000 in an incognito window. When i typed "pa" in the menu search box. The suggestions list opened, but only the top row (Pav Bhaji) could be clicked. The middle rows (Paneer Butter Masala with Butter Naan, Bread Pakora) were partly hidden behind the category tabs row, and clicking them did nothing. Happened every time the list was long enough to reach the tabs row (According to the question and i also checked the bug on my own as well ). Then i inspected the site and used Google Dev Tools. 

**Cause:** Then i checked that the suggestions list sits inside .search-wrap (checked when i inspected the site ), which had position: relative and z-index: 1. That layer was too low, so the category tabs row below it was drawn on top and caught the clicks. The click handlers were fine, but the clicks never reached the suggestion buttons. A child cannot rise above its parent's layer, so the list's own styles could not fix it.

**Fix:** Raised the z-index on the existing .search-wrap rule in style.css from 1 to 100. I tested it first in Google DevTools, then moved the same change into style.css.

**Checked:** Now when i type any dish name it come above the tab rows , and now i can click all the suggestions rows which comes in top, middle and bottom as well, also i checked and verified it in both light and dark theme as well. 

**Time:** About 25 mins

## CC-02 — "Can't read anything in dark mode"

**Reproduced:** I opened the site on localhost:3000 in an incognito window. It opened in dark mode straight away, with no toggle or saved theme involved. The dish names and prices were almost invisible, but the grey "Breakfast" line under the name was same. Then i inpected on the website and used Google DevTools and under render tab i used "Emulate CSS media feature prefers-color-scheme" and i checked it in light and dark emulation. 

**Cause:** In style.css, the .dish-body rule had color: #2b2118 typed in directly instead of using the --ink variable. That is the same colour as --ink in light mode, but a typed-in colour never changes when the theme switches. The dish name (color: inherit) and the price both picked it up from there, so they stayed dark brown on a dark theme. The grey line worked because it uses --muted, which the dark theme does override.

**Fix:** Changed color: #2b2118 to color: var(--ink) in .dish-body (style.css, line 294). That rule is where both the name and the price get their colour, so fixing it there repaired both without adding any dark-mode-only rules.

**Checked:** In dark mode the dish names and prices are readable now. In light mode they look the same as before as well.

**Time:** About 2 hours 

## CC-03 — "The menu is wider than my phone"

**Reproduced:** I opened the site on the localhost:3000 in an incognito window. First i inspected and then i used Google Dev Tools and tried a few phone widths. 360, 375 and 414px were fine. At 320px the dish cards were getting stuck out past the right edge and the Add to Cart buttons were cut off. Happened every time at that width.

**Cause:** On small screens the menu grid was set to grid-template-columns: 1fr. A plain 1fr column won’t shrink smaller than what’s inside it, so at 320px the column ended up wider than the phone and pushed the cards off the edge.

**Fix:** Changed that one value to minmax(0, 1fr) in the existing 480px media query.

**Checked:** At 320, 360, 375 and 414px the page is exactly as wide as the screen, nothing scrolls sideways, and every Add to Cart button is fully visible.

**Time:** About 1.5 hours




