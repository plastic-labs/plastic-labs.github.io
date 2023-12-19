TL;DR: rather than general eval benchmarks, how can we assess performance on an individual basis?

The first step is defining the application development around a user so managing their context is paramount.

Then you can begin to play around with intermediate prompts that seek to reason about the user's needs, store important facts, and use a combination of that reasoning + derived facts to add very specific context to your actual service prompt.

Once that's done, you need some way of collecting feedback. i see two options here...
1. passive: make predictions about the user and update your model of them based on the delta between prediction and reality
2. active: ask for some minimally-invasive feedback to update your model

"Model" in this sense is defined quite loosely. In the current VoE implementation, this model is stored as facts in a vector DB. But I imagine we can get quite complex here.

The novel thing to do here is offload this feedback task to the language model. You can't trust users to provide authentic feedback and theory of mind in large language models is good enough to handle this.

Jacob had a really good point here -- that this is best viewed through a reinforcement learning lens. Through that lens, people are the "environment" and the "agents" (AIs) are trying to optimize their reward model (psychological coherence). 