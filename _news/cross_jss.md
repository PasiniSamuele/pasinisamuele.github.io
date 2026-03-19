---
layout: post
date: 2026-03-19 11:00:00-0400
inline: false
related_posts: false
title: New publication in JSS!
---

Our paper, “Cross-Site Scripting Adversarial Attacks Based on Deep Reinforcement Learning: Evaluation and Extension Study,” has been accepted for publication in the Journal of Systems and Software! (JSS)!

---

This work was made possible thanks to my co-authors Gianluca Maragliano, Jinhan Kim, and Professor Paolo Tonella.

Key Takeaways:
- AI in security-critical systems requires caution: While Deep Learning and Language Models are increasingly used in software security, our work shows that their evaluation can be misleading if not carefully designed.
- Adversarial testing pitfalls: We demonstrate how common evaluation pipelines can introduce hidden flaws (e.g., preprocessing artifacts) that make attacks appear more effective than they actually are.
- From “how to” to “how not to”: By replicating and stress-testing prior work, we highlight incorrect practices in adversarial evaluation and propose a more reliable methodology.
- Oracle-based validation: We introduce an XSS Oracle to ensure that adversarial examples remain semantically valid, leading to a more trustworthy assessment of robustness.

Core message: When deploying AI—especially LLMs and Deep Learning—in security-sensitive components, we must rigorously validate both the models and the evaluation pipelines. Otherwise, we risk building systems that appear robust but fail in practice.
