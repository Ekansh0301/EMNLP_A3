# Robustness of NLP Evaluation Metrics for English-Hindi Machine Translation

Evaluating how BLEU, chrF++, COMET-DA, and COMET-QE respond to adversarial perturbations in machine translation. We stress-test three MT models across clean, surface-noisy, and distribution-shifted conditions using the IITB English-Hindi Corpus.


**Course:** EMNLP: Evaluation Methods for NLP: Assignment 3

---

## Dataset & Perturbations

**Source:** IITB English-Hindi Corpus (300 sentences, 95% CI: ±2.6 BLEU)

Three variants with identical reference translations:
1. **Original:** Clean English sources
2. **Surface-Perturbed (92.3%):** OCR-like noise—transposition, deletion, insertion, swaps
3. **Distribution-Shifted (78.3%):** Register shifts, code-mixing, gender swaps (M↔F)

---

## Machine Translation Models

Three publicly available English→Hindi systems spanning generalist to Indic-specialized:

| # | Model | Parameters | Training Focus | Role in Study |
|---|-------|-----------|-----------------|---|
| **M1** | `Helsinki-NLP/opus-mt-en-hi` (MarianMT) | 74M | OPUS general-domain parallel data | Baseline; shows high length sensitivity |
| **M2** | `facebook/nllb-200-distilled-600M` (NLLB-200) | 600M | CCNet / FLORES-200 (200 languages) | Strong multilingual baseline |
| **M3** | `ai4bharat/indictrans2-en-indic-dist-200M` (IndicTrans2) | 200M | IndicCorp + Samanantar (Indic-specific) | Language-specialized SOTA |

**Consistent generation parameters across all models:**
- `num_beams = 4`
- `max_new_tokens = 256`

**Key observation:** IndicTrans2 uses IndicTransToolkit for Devanagari pre/post-processing, giving it architectural advantage for Indic morphology.

---

## Metrics

| Metric | Type | Robustness | Best For |
|--------|------|-----------|----------|
| **BLEU** | Surface n-gram | Fragile (15–32% drop) | Reproducibility only |
| **chrF++** | Character n-gram | Moderate (8.5–18.3% drop) | Development cycles |
| **COMET-DA** | Neural semantic | Robust (5.6–12.9% drop) | Final evaluation + σ |
| **COMET-QE** | Reference-free | Fairness evaluation | Bias analysis |

---

## Results

### Surface Fragility Hierarchy
```
BLEU drops 15–32% → chrF++ drops 8.5–18.3% → COMET drops 5.6–12.9%
(consistent across all 3 models)
```
MarianMT loses 32% BLEU vs. 12.9% COMET for identical inputs (2.5× gap).

### Model Rankings
**IndicTrans2** (σ=0.079, most stable) > **NLLB-200** > **MarianMT** (σ=0.157, most erratic)

**Inter-Metric Divergence:** BLEU ranks MarianMT 2.5× below NLLB; COMET ranks it only 1.34× below. BLEU penalizes Devanagari surface-forms COMET absorbs as semantically equivalent.

### Fairness Analysis (COMET-QE)
- **Gender gap:** 0.0015 (p=0.213, non-significant) ✓
- **Code-mixing penalty:** −0.0298 (20× larger than gender gap) ⚠
- **Length bias:** Model-intrinsic; MarianMT most sensitive (r≈−0.30)

---

## Recommendations

- **Development:** Use **chrF++**—robust, fast, Devanagari-appropriate
- **Final evaluation:** Use **COMET-DA** with segment-level σ reporting
- **Reproducibility:** Include BLEU only—never interpret in isolation
- **Fairness:** Add code-mixed (Hinglish) evaluation data—code-mixing penalty is 20× larger than gender bias

---

## Conclusions

1. **Surface fragility hierarchy is consistent:** BLEU (15–32%) >> chrF++ (8.5–18.3%) > COMET (5.6–12.9%)
2. **IndicTrans2 most robust:** Highest scores, lowest variance (σ=0.079)
3. **No gender bias detected:** COMET-QE gap = 0.0015 (p=0.213, non-sig)
4. **Code-mixing is primary fairness risk:** Gap = −0.0298 (20× gender gap, σ=0.054)
5. **Length bias is model-intrinsic:** Stable across perturbations; MarianMT most sensitive (r≈−0.30)

---

## Project Structure

```
├── README.md                     # This file
├── requirements.txt              # Dependencies (pip install -r requirements.txt)
├── report.tex                    # Full academic report
├── notebooks/
│   └── analysis_pipeline.ipynb   # Complete reproducible workflow
├── data/
│   ├── comprehensive_results.csv # 8,100 metric scores
│   ├── summary_statistics.csv
│   ├── modification_statistics.csv
│   └── error_analysis.csv
└── figures/
    ├── 01–10_*.png              # All visualizations (10 plots)
```

---

## Quick Start

```bash
pip install -r requirements.txt
jupyter notebook notebooks/analysis_pipeline.ipynb
```

All 8,100 metric scores reproducible from `data/comprehensive_results.csv` (fixed seeds, deterministic).

---

## Acknowledgments

**Data:** IITB English-Hindi Corpus (Kunchukuttan et al., 2018)  
**Models:** Helsinki-NLP MarianMT, Meta NLLB-200, AI4Bharat IndicTrans2  
**Metrics:** BLEU, chrF++, COMET (Rei et al., 2020)
