# CephalonTools.com


## Background

This is a website that I made to solve a friction problem for the game Warframe.
Warframe has a robust community trading system, with players being able to trade
premium currency and most in-game items with each other. While the in-game interface 
for this leaves much to be desired, several community tools have sprung up throughout 
the years to much success. Out of these, https://warframe.market (WFM for short) has 
seen the most use. It lets users create listings to buy and sell every tradable item. 
This was a big step forward, but it has a few issues, the most glaring being the complete
lack of wider informational views.


Here's an overview of the steps a user can take to answer the questions WFM struggles with:

	**"Out of a large amount of items, which are the most valuable?"**: 
	- Individually look up every item they have
	- Write down the prices of each item 

	**"How should I most efficiently spend my time to get a certain item?"**:
	- Individually search every revenue source they have
	- Search the desired items
	- Search how to get the desired items (takes user off-site)
	- Compare the efficiency of buying vs directly obtaining the item

Obviously, this is a ridiculous amount of tasks for things that should be trivial.
Luckily, WFM offers [an API](https://docs.warframe.market/docs/intro/) to fetch almost anything on the site,
so I used this to make a complementing tool for WFM.

The large portion of the items affected by the above questions fall into two categories:
**prime parts** and **Arcanes**. For the initial launch of the site, I just focused on 
prime parts and their sources, with arcanes to follow in a later update.

### An Example

To give a concrete example of what the base process looks like using solely WFM, let's
walk through getting all the parts for Zephyr Prime. It's common knowledge among players that
Warframes (which Zephyr is) have 4 parts: Blueprint, Systems, Neuroptics, and Chassis.
In this example, let's say the player has 0 platinum to start (abbreviated as p or plat).
This is the premium currency of the game and the one used universally in trading.

A WFM search reveals the prices of the components the player needs. It also has the option to
buy them as a full set as a separate listing:

- Blueprint  			- 16p
- Systems    			- 40p
- Neuroptics 			-  4p
- Chassis    			-  5p
- Individual total cost - 65p
- Set        			- 79p

At these prices, we would need to get 65p before we can buy all the parts. To do this, the 
player can open some of the relics that they already own -- relics contain prime parts, although
not necessarily the ones that we want. Relics have 6 items that they can turn into, with the odds 
varying across item rarity and other currency investment. The player *could* individually look up
all 6 items that each relic contains: across all ~100 relics that they own. But, this is clearly
an absurd amount of time to spend *not* playing the game. Instead, they settle on something
inefficient: trying to luck into each of the parts they need. They settle on repeatedly 
opening 2 relics that have the Neuroptics and Chassis (Axi O3 & Axi O4). For getting the 
part they want, it's between a 11-25% chance on each run, and a 2-10% chance at getting the best
drop valued at 18p. Assuming the best odds from (non plat) currency investment, the expected
value of each run is 6.0-6.3p. Including the parts we want to keep for Zephyr, this means an
**expected ~11.3 runs** to get the 56 remaining platinum needed.

### Example using CephalonTools (CT)

In the same example as above, the user could drastically cut down the amount of time needed to 
reach their goal using my website. The core feature of CT is a sortable & searchable table of 
all of the parts in all of your relics. While the user does have to manually enter their owned 
relics, this turns out to be a very fast process thanks to a few small details:

- The relics have a structured name: <ERA> <LETTER> <1-2 DIGIT NUMBER>
	- Era becomes a selectable drop-down menu, relics are sorted in-game by alphanumeric order,
	  so this only has to change 3-4 times in total when entering all
	- The Letter and 2 Digit number are very fast to type
	- Enter is a shortcut instead of pressing the "add" button
	- We can quickly verify that the Letter & Digit combo corresponds to a real relic
- I save this data in the user's local storage, and they have the option to import/export 

In my case, I have roughly 125 relics, and it took me only a minute to enter them all.

After spending the time to enter their relics, the user now has access to the previously mentioned
table. Simply put, it's almost always worth the time to use the relics with the highest output
and trade for the wanted parts rather than try to get them directly. The table sorts by highest
part value by default, and there's also an option to sort by relic EV calculation in a separate 
tab. Assuming the user has been playing Warframe for a while, they most likely have many relics with 
an EV > 3x what they using previously. Consequently, this means the user can shorten the time 
to get all of the parts by the same amount. A relic EV of 20p is only **3.2 runs** instead of the
earlier 11.3.

Of course, this is not the only use case for the site. I've shared it with a few Warframe players
since I released the initial version, and user feedback suggests that it's reaching a wide variety
of players. With so much of the game revolving around the economy and market, a resource that 
provides a way to quickly see what's "easy money" is in high demand.

## Development

Making the initial launch of the site took about 4 days. Because I was not handling any sensitive 
informational and because I was starting from scratch, this project was a great fit for heavy
AI use. Aside from AI coding tools, this project features a Go backend, JS frontend, a SQL database,
and automated CI/CD.

### AI Usage

This project features AI use at a level that is typically higher than I prefer. I like to think of
LLMs as more of a tool rather than a primary driver in the dev process. Pure AI code tends to be
sloppy (in both senses), directionless, and naive. This can be allieviated through strong direction,
strict guidelines, and a deeper understanding of your intended structure. On the flip side, AI tends
to be really good at searching, quick scripts, and pointed queries.

My AI of choice for this project was Claude Opus 5. I start by writing -- or taking chunks from a 
previous -- CLAUDE.md. This includes the typical: 

- Ask, don't assume
- Be concise; skip framing
- Don't silently pick options when writing code
- Use standard library solutions; avoid writing bespoke code
- Don't get anchored on specific wording; intention over wording
- Coding style to use
- Before trying fallback solutions, ask the user and try to solve the root issue

This is in shorthand, and it's more verbose in the document. I've found that including these directions
generally help avoid possible mistakes before they have a chance to happen.

In addition to the AGENTS/CLAUDE file, I also prepare a project document that details: the overall
goals of the project; specific choices that I made; past, current, and future steps; architecture; 
specific component behavior; etc. In my opinion, something like this document is absoultely essential 
to good agent use. It serves as a handoff document to save on token use, provides necessary context, 
and it prevents the agent from making the same mistakes by giving reasoning as to why certain 
decisions were made.

A final step of configuration is restricting available terminal commands that the agent can use. I
do not allow any commands or tools that affect remote environments. No git commands that edit,  
wrangler, gcloud, etc.

Only after all of this is done do I start the agent CLI. Most of the time, I use it in manual mode.
While it can be faster in auto, it makes frequent mistakes that could be caught otherwise.

### Go Backend










