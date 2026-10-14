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

## CC-05 — "The category bar scrolls away on my phone"

**Reproduced:** I opened the site on the localhost:3000 in an incognito window. Then i opened the menu in Chrome DevTools with the iPhone SE preset (375 x 667) and scrolled down through the dishes. The category and filter bar scrolled off the top of the screen instead of staying right there. On a wide desktop window the same bar stayed pinned, so it only happened on small screens.

**Cause:** The bar was already at position: sticky, so that part was fine. The problem was in the max-width: 480px block, where .view had overflow-x: hidden. Setting one overflow axis to hidden makes the other one auto, which quietly turned .view into a scroll container. Sticky sticks to the nearest scroll container, and .view never actually scrolls (the page does), so the bar had nothing to stick to. There was also top: 0 on the bar, so even once it stuck it would have slid underneath the sticky header.

**Fix:** Changed overflow-x: hidden to overflow-x: clip on .view. It still cuts off sideways overflow but doesn't create a scroll container. Then changed .filters to top: 93px, so it parks just below them.

**Checked:** Reloaded in the iPhone SE view and other phone models as well and scrolled the menu. The filter and category bar stays pinned under the orange slot bar, and the header and slot bar are still visible above it.

**Time:** About 1 hour

## CC-06 — "I ordered more than they had"

**Reproduced:** I opened the site on the localhost:3000 in an incognito window. I selected a dish from the menu and saw the stock was 2 and it let me order 5 , even though the stock was 2, and the order placed without any error about stock.

**Cause:** Two spots in logic/validation.js. validateLine only checked whether stock was 0 or less, so 5 against a stock of 2 looked fine to it. reserveStock then just subtracted the quantity without checking anything, even though the comment above it says it re-checks every line. So nothing ever stopped the order.

**Fix:** Added a qty > stock check in validateLine so the customer gets a clear "only 2 left, you asked for 5" message. Also made reserveStock check stock before subtracting

**Checked:** Now again i selected the dish with stock 2 and tried to place the order for quantity of 4, now the order didn't placed and the message shown there stock left is 2 only.

**Time:** About 40 Mins

## CC-07 — "Cancelling makes it worse"

**Reproduced:** I opened the site on the localhost:3000 in an incognito window. I placed an order for a dish, then cancelled it. The stock for that dish went down again instead of coming back. Cancel a few orders and the dish drops to 0 and shows as sold out, even though nothing is actually gone. Happens on every cancel, on any dish.

**Cause:** releaseStock in logic/validation.js had the wrong sign. It did dish.stock - line.qty, so a cancel took the plates off a second time instead of putting them back. The cancel route itself was fine, it was calling releaseStock correctly.

**Fix:** Changed the minus to a plus in releaseStock, so it's dish.stock + line.qty.

**Checked:** Again i placed the order for the dish, then cancelled it. This time the stock went up after cancelling instead of going down.

**Time:** About 20 Mins

## CC-08 — "An old coupon still works"

**Reproduced:** I opened the site on the localhost:3000 in an incognito window. I added a dish to the cart and typed FRESHERS24 into the coupon box. Even though the offer expired last year, the code was accepted and the discount was applied.

**Cause:** The expiry date was only checked when building the list of coupons shown as chips. The code that actually validates a coupon (applyCoupon in logic/pricing.js) was handed the current time but never compared it to the coupon's expiry date. So the list hid expired coupons, but anyone typing the code skipped that check, and both the cart preview and checkout went through the unchecked path.

**Fix:** Added an expiry check inside applyCoupon, using the "now" value it already receives, so there is one place that decides whether a coupon is valid. A coupon with a missing or unreadable expiry date is treated as expired, so bad data can't let a coupon through by accident. The cart preview and checkout both use the same function, so one change covers both.

**Checked:** Typing FRESHERS24 now shows "FRESHERS24 has expired", no discount is applied, and the total doesn't change. Lowercase and extra spaces behave the same. Placing an order with the expired code is rejected. BYTE10, FLAT50 and VEGDAY still apply normally.

**Time:** About 35 Mins


## CC-09 — "The menu shows more dishes than it should"

**Reproduced:** I opened the site on the localhost:3000 in an incognito window. Then i opened the menu and all 37 dishes loaded straight away, even though the page asks the server for 5 at a time. Scrolling made it worse. The list grew to 74 because the same 37 dishes got added again. Happened on every load, with no filters on.

**Cause:** The frontend was fine. It asked for page 1 with a limit of 5 and appended more as you scrolled. The problem was on the server. paginate() in logic/search.js cut out the right 5 dishes, but then returned the whole list instead of that slice. The totals and hasMore were worked out correctly, so the frontend kept thinking there were more pages and kept loading the full list again.

**Fix:** Changed items: list to items in the return of paginate().

**Checked:** First load now shows 5 dishes, the count line says total 37, and the category tabs still say All 37. Scrolling loads 5 more at a time and stops at 37 with no repeats. Filters and search start again from the first page.

**Time:** About 1 hour

## CC-10 — "Sorting by price is backwards"

**Reproduced:** I opened the site on the localhost:3000 in an incognito window. I picked “Price: low to high” on the menu and the most expensive dishes came first. Switching to “high to low” gave the cheapest ones first. Happened every time, with or without filters, and on every page of results.

**Cause:** The two price comparators in SORTERS (logic/search.js) had their operands the wrong way round. price-asc was written as b.price - a.price, which sorts biggest first, and price-desc was the mirror of that. The sort worked fine, it just ran in the opposite direction to what the dropdown said.

**Fix:** Swapped a and b in both comparators so price-asc is a.price - b.price and price-desc is b.price - a.price.

**Checked:** Now Low to high starts from cheap and goes up and High to low starts at the most expensive and comes down. The order stays the same as more dishes load on scroll, and it still works with the Veg, Non-Veg filter as well.

**Time:** About 15 min








