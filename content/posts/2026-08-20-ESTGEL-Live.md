+++
title = "ESTGEL Is Live: Go Click a Cell"
date = 2026-08-20
summary = "The cell-fate model is off my GPU and on the internet. An interactive 3D embryo you can scrub through, and a Space where you can upload your own."
tags = ["GSOC", "OpenWorm", "ESTGEL", "C. elegans", "DevoLearn", "Demo"]
+++

**New here? The one paragraph version.** I'm spending Google Summer of Code with [OpenWorm / DevoWorm](https://github.com/DevoLearn/DevoGraph) working on *C. elegans*, a millimetre-long worm whose embryo is so predictable that biologists have mapped every single cell division it makes. I'm training a graph neural network to watch that embryo develop and predict what each cell grows up to become: a neuron, muscle, gut, skin. It gets to see where a cell sits in 3D, how big it is, when it appears and which cells it's touching. It never gets to see the cell's name, because in this worm the name *is* the answer.

The model stopped living on my GPU. It's on the internet now, and you can poke it.

<div style="position:relative;width:100%;height:min(80vh,720px);margin:1.5rem 0;border:1px solid rgba(128,128,128,.28);border-radius:10px;overflow:hidden;">
  <iframe src="https://barshan.is-a.dev/estgel/" title="ESTGEL cell-fate dashboard" loading="lazy" style="width:100%;height:100%;border:0;"></iframe>
</div>

If that panel didn't load, it lives at **[barshan.is-a.dev/estgel](https://barshan.is-a.dev/estgel/)**.

Every dot is a cell, coloured by the tissue my network thinks it becomes. Drag to rotate, scroll to zoom, drag the timeline to play development forward and watch cells divide. Click any cell and the inspector tells you what it was predicted to be, what it actually becomes, and how confident the call was.

Three real embryos are loaded in as samples. Or feed it your own EPIC CSV, as long as it has `cell, time, x, y, z, size, blot` and somewhere between 12 and 1200 cells.

## Upload your own

The prediction itself runs in a Space, which the dashboard calls behind the scenes. You can also use it directly:

<div style="position:relative;width:100%;height:min(75vh,700px);margin:1.5rem 0;border:1px solid rgba(128,128,128,.28);border-radius:10px;overflow:hidden;">
  <iframe src="https://bluebarshan-estgel-cell-fate.hf.space" title="ESTGEL Cell Fate Space" loading="lazy" style="width:100%;height:100%;border:0;"></iframe>
</div>

Direct link: **[huggingface.co/spaces/bluebarshan/estgel-cell-fate](https://huggingface.co/spaces/bluebarshan/estgel-cell-fate)**

## The honest numbers

63% accuracy on held-out embryos, macro-F1 0.34. Pharynx, hypodermis and muscle land around 72 to 80% recall. Glia sits at 4%, and the genuinely rare tissues are at zero, because there are only a handful of them in the whole dataset and the model has never had a real chance to learn them. Those are on the dashboard too. I'd rather you see them than not.

The result I actually care about is this one:

![Accuracy drops from 50% to 31% and 43% of predictions flip when contact edges are removed](/previewimg/graph_ablation.png)

I never show the model a cell's name, because in *C. elegans* the name gives away the answer. So the only way it can be right is by reading position, size, timing and who a cell is touching. Delete every contact edge and rerun, and 43% of the calls change. The graph is not decoration.

The dashboard measures importance this way throughout, by perturbation rather than attention weights, because the attention weights saturate and stop being informative.

## One footnote for anyone deploying

The whole classifier is **80,556 parameters**. It runs on CPU in a couple of seconds, no GPU anywhere.

Which made deployment mildly absurd: the Space runs on ZeroGPU, and ZeroGPU refuses to start unless at least one GPU-decorated function exists in your app. So there's a function in there called `warm_up` that is decorated, does nothing, returns `"ok"`, and is never called by anything. It exists purely to satisfy a startup check. No GPU quota is ever consumed.

Anyway. Go break it, and tell me what happens.
