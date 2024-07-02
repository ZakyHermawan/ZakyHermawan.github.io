---
layout: post
title:  "Week 5: GRC Separation milestones"
author: zaky
categories: [ gsoc ]
image: assets/images/gnuradio_repositories.png
---

So in order to do GRC separation, we split it into 3 milestones.

# First Milestone
The first milestone would be to just make grc into separate submodule for gnuradio, and change nothing. In order to do this, We need to have gnuradio/grc repo, then I will make PR into gnuradio/grc to move newest grc folder from gnuradio/gnuradio into its separate repo (gnuradio/grc), so I can point the grc submodule of gnuradio into that repo, then I will make PR into gnuradio/gnuradio repo just to make grc as a submodule. 

So the result of this first milestone would be:
The content of gnuradio/grc repo will have 0 change, and the changes in gnuradio/gnuradio only just to make grc folder as a submodule.

# Second Milestone
Second milestone would be the heaviest work, where we do folder restructuring, then make the pip as one of options to install grc (the other one is to install grc via cmake).

# Third Milestone
The third milestone would be to make pip install as a default way to install grc.

# Current Status of GRC Separation
I currently working on second milestone, to make grc can be installed via pip. Currently, I have make it work on grc gtk version, but there is still some problem in GRC GTK, and already did the folder restructuring into src folder structure.
The first step could not be done because we still don't have gnuradio/grc repo, so I can't make a PR into gnuradio/grc repo.

## Conclusions
* GRC separation is devided into 3 milestones
* Steps 2 is currently in progress
* Steps 1 is still waiting until we have gnuradio/grc repo
