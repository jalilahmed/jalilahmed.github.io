---
title: "What Does a Deep Learning Model Really Learn from Chest X-rays?" 
date: 2025-01-24
author: ["Jalil Ahmed"]
description: "A comprehensive guide analyzing limitations of classification model" 
summary: "A comprehensive guide analyzing limitations of a disease classification model" 
tags: ["AI in Healthcare", "Deep Learning", "Medical AI", "Responsible AI", "Computer Vision"]
---

When we train a neural network to diagnose diseases from medical images, we often celebrate high accuracy scores and move on. Lets analyze a ResNet-50 on chest X-ray classification to reveal a more nuanced story—one that every AI practitioner in healthcare should understand.

## The Deceptive Simplicity of "Good Performance"

Training a deep learning model for medical image classification seems straightforward: get data, train model, achieve high metrics, deploy. But this workflow obscures critical questions: What has the model actually learned? When will its predictions fail? Can we trust it with real patients?

I analyzed the performance of a standard ResNet-50 on the NIH ChestX-ray14 dataset—112,120 chest X-ray images labeled with 14 different pathologies to understand these questions better. The goal wasn't to achieve state-of-the-art performance, but rather to understand what a baseline model learns and, crucially, what we can and cannot infer from its behavior.

## The Setup: Deliberately Minimal

The training approach was intentionally simple:

- Standard ResNet-50 architecture with random initialization (no ImageNet pretraining)
- Basic Adam optimizer with fixed learning rate
- No fancy data augmentation or hyperparameter tuning
- Single training run on one GPU

Why so minimal? Because extensive tuning can obscure which design choices actually matter and make results harder to reproduce. The focus here was on rigorous evaluation, not leaderboard competition.

## Performance Isn't a Single Number

The model achieved a macro-average AUROC of 0.744 across all pathologies. But that single number hides variation across different pathologies:

**High performers (AUROC > 0.80):**
- Cardiomegaly: 0.852
- Edema: 0.825
- Pneumothorax: 0.816

**Low performers (AUROC < 0.70):**
- Nodule: 0.642
- Pneumonia: 0.668
- Infiltration: 0.684

Why such a spread? The high performers share something in common: they produce clear, high-contrast visual patterns. An enlarged heart creates an obvious change in cardiac silhouette, Pulmonary edema shows widespread white opacities and Pneumothorax displays sharp lung edge demarcation.

The low performers? They're subtle, variable, and easily confused with other conditions. Nodules can be tiny and hidden behind ribs, Pneumonia manifests in diverse patterns that overlap with other diseases, and Infiltration is notoriously ambiguous even for radiologists.

**The key insight**: Even with identical architecture and training, different diseases have drastically different learnability. "Model performance" is not a single scalar but it's a distribution that reflects task complexity.

## Looking Inside: The Grad-CAM Revelations

To understand what the model was actually looking at, I used Grad-CAM visualizations—heatmaps showing which image regions most influenced each prediction.

### The Good News

For high-performing pathologies, attention patterns looked anatomically plausible:

- **Cardiomegaly**: Model focused on the central thoracic region where the heart is located
- **Edema**: Diffuse attention spanning bilateral lung fields, matching the widespread nature of edema
- **Pneumothorax**: Concentrated attention in upper chest regions

This seems encouraging and the model appears to be looking at the right places.

### The Concerning News

For low-performing pathologies, attention patterns were problematic:

- **Infiltration**: Heavy attention on image corners and edges—likely learning from artifacts rather than pathology
- **Pneumonia**: Extrathoracic bias, potentially relying on hospital-specific tags or positioning artifacts
- **Nodule**: Central mediastinal focus instead of localized lung regions, suggesting the model hasn't learned what a nodule actually is

## The Interpretability Trap

Here's where things get philosophically interesting. We should be warned against what is called the "Interpretability Trap"—a flawed reasoning chain that goes:

1. Model produces plausible-looking attention map
2. Human observer recognizes anatomically relevant region
3. Observer concludes model learned "correct" reasoning
4. Observer's confidence in model increases ← **The Trap**

The problem? Plausible attention is necessary but not sufficient for valid reasoning. A model might highlight the correct anatomy while actually exploiting spurious correlations invisible to human observers. It might be detecting the presence of medical equipment, patient positioning, or hospital-specific imaging protocols rather than the disease itself.

Therefore, it is important to note that Grad-CAM shows correlation, not causation. It reveals what the model associates with labels in the training distribution, not necessarily the biological cause of pathology.

## What We Still Don't Know

We should always be explicit about our work's limitations:

**Not evaluated:**
- Performance on other chest X-ray datasets
- Behavior across patient demographics
- Robustness to adversarial examples or distribution shifts
- Agreement with radiologist interpretations
- Temporal stability as medical practices evolve

**Cannot conclude:**
- The model learned clinically valid reasoning
- Performance will generalize to new hospitals or populations
- Grad-CAM validates model correctness
- The model is safe for clinical deployment


## The Real Lesson

This work isn't just about chest X-rays or ResNet-50. It's a template for how we should approach AI in high-stakes domains:

1. **Be explicit about methodology**: Document every choice and its rationale
2. **Report heterogeneous performance**: Aggregate metrics hide crucial variation
3. **Use explanations diagnostically, not defensively**: Grad-CAM helps identify failures, not justify deployment
4. **Define the scope of valid inference**: State clearly what you can and cannot conclude
5. **Prioritize robustness over optimization**: Real-world reliability matters more than benchmark numbers

## Moving Forward

The path from "model works in lab" to "model safe for patients" requires much more than good test metrics:

- Rigorous out-of-distribution testing across multiple datasets
- Subgroup analysis to detect performance disparities
- Temporal validation to ensure stability over time
- Clinical validation with radiologist gold standards
- Adversarial testing to probe failure modes

High AUROC scores are necessary but far from sufficient. The hard questions aren't about accuracy—they're about robustness, fairness, interpretability, and ultimately trust.

## Conclusion

The aim of this work is to reveals how much we still don't understand about what our models have learned and when they'll fail.

The most valuable contribution isn't the highest performance metric. It's the framework for rigorous evaluation and honest uncertainty quantification. In healthcare AI, knowing the boundaries of our knowledge isn't just good science—it's an ethical imperative. Therefore, before deploying any AI system in clinical settings, we must answer not just "Does it work?" but "How does it work?", "When does it fail?", and "How do we know?"

---

**Read the full technical report**: [What a Standard ResNet Learns on NIH Chest X-rays—and What We Can (and Can't) Infer From It](./report.pdf)