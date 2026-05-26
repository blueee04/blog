+++
title = "Tiny Worms and Large Graphs"
date = 2026-05-24
summary = "So, I Started Working on OpenWorm"
tags = ["Deep Learning", "C. elegans", "graph-neural-networks", "OpenWorm", "GSOC", "data-preprocessing"]
+++

A few months ago, I somehow ended up spending my evenings reading about worms, dynamical systems, graph learning, and embryogenesis.

And now I’m here.

This summer, I’ll be working with the OpenWorm Foundation as a GSoC contributor on the project:

[**Explainable Spatio-Temporal Graph Evolution for Developmental Neuroscience**](https://summerofcode.withgoogle.com/programs/2026/projects/A289WZd0)

Honestly, when I first came across developmental biology and computational neuroscience, I never expected it to connect this naturally with graph learning and deep learning. But somewhere between lineage trees, cell trajectories, and attention mechanisms, things started becoming *very* interesting.

The first week was mostly me getting to know more about the OpenWorm community and what are they striving to do. I got to meet [Dr. Bradley Alicea](https://github.com/balicea), [Jesse Parent](https://jesparent.com/) and [Mehul Arora](https://www.linkedin.com/in/mehular0ra/). They gave me a warm welcome into the community.

This blog post is mostly a small recap of what the past few months looked like and my community bonding period

## The Last Few Months

Before the official coding period even began, I spent a lot of time attending discussions, meetings, paper reading sessions, and falling into random rabbit holes around developmental biology. But the interesting part was that it never felt limited to just neuroscience or research. A huge part of the experience was also about learning how to communicate ideas better presenting papers, explaining complicated concepts simply, storytelling, managing time, and learning how to work openly with a community. Honestly, those are the kinds of skills that don’t get talked about enough in technical spaces, but end up mattering just as much.

Most of my recent weeks have looked something like this:

- reading papers at 2 AM
- trying to understand why cells move the way they do
- opening Napari documentation and staring at embryogenesis visualizations for unhealthy amounts of time
- pretending I fully understand dynamical systems
- discovering that biology people casually use words that sound like boss fights

Some of the papers and topics I explored during this period included:

- The Markovian Worm [lecture](https://www.youtube.com/watch?v=zMbSzsS9t98&t=2680s)
- Cell motility through eigenvalues and dynamical systems [lecture](https://www.youtube.com/watch?v=-m0qovQaMuc&t=2s), [github](https://github.com/blueee04/Behaviour-Of-C.-elegans_Markovian_Embryo-based)
- Lineage-specific dynamics and learning on graphs using attention mechanisms for developmental learning [lecture](https://www.youtube.com/watch?v=GuEXhesoDkg&t=150s)
- Dynamic relation learning for C. elegans embryogenesis [lecture](https://www.youtube.com/watch?v=MzTCzQPT0Js&t=0s), [github](https://github.com/blueee04/GazeGraph)

I spent some time building preprocessing pipelines and dataset structures for handling temporal embryogenesis data.

The dataset itself contains things like:

1. cell coordinates across time
2. lineage information
3. edge creation timestamps
4. alive/dead cell states
5. behavioral and spatial metadata

All of this in one .npz (Numpy Array) file.

You can find the dataset [here](https://github.com/blueee04/Spatio-Temporal-Evolution)

I wrote a dedicated technical blog about this pipeline because there’s a lot to unpack there.

Get that [here](https://barshan.is-a.dev/posts/2026-04-15-epic-data-preprocessing-documentation/) buddy...

## What Next?

The coding period officially begins soon.

I’m genuinely excited about where this project could go over the next few months. There’s still a lot I don’t know yet, but honestly that’s probably the best part.

I’ll keep posting detailed updates and technical breakdowns in my blogpost

https://barshan.is-a.dev/

And hopefully by the end of the summer, we’ll have something really interesting growing out of this tiny worm.