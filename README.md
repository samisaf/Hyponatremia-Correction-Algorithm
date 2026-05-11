# Hyponatremia Correction Algorithm

A mathematical tool for calculating intravenous fluid (IVF) rates or predicting serum sodium changes in patients with dysnatremia, with explicit modeling of renal electrolyte and fluid losses.

## Overview

This project derives equations relating IVF infusion parameters to changes in serum sodium concentration, accounting for urinary electrolyte and fluid losses. It extends the widely used Adrogué-Madias equation by treating the body as an **open system** rather than a closed one.

## Variables

| Symbol | Description |
|--------|-------------|
| `NaStart` | Starting serum sodium (mEq/L) |
| `NaEnd` | Target serum sodium (mEq/L) |
| `TBW` | Total body water (L) |
| `IVFConc` | Sodium concentration of IVF (mEq/L) |
| `IVFRate` | IVF infusion rate (L/hr) |
| `Time` | Duration of infusion (hr) |
| `UNa` | Urine sodium concentration (mEq/L) |
| `UK` | Urine potassium concentration (mEq/L) |
| `URate` | Urine output rate (L/hr) |

## Derived Equations

### IVF Rate to Achieve Target Sodium

$$IVFRate = \frac{NaEnd \times TBW - NaEnd \times Time \times URate - NaStart \times TBW + Time \times UK \times URate + Time \times UNa \times URate}{Time \times (IVFConc - NaEnd)}$$

### Predicted Sodium After Infusion

$$NaEnd = \frac{IVFConc \times IVFRate \times Time + NaStart \times TBW - Time \times UK \times URate - Time \times UNa \times URate}{IVFRate \times Time + TBW - Time \times URate}$$

## Derivation

The equations are derived from conservation of sodium (total exchangeable sodium, `TES = NaStart × TBW`), incorporating:

- Sodium gained from IVF infusion: `IVFConc × IVFRate × Time`
- Sodium lost in urine: `(UNa + UK) × URate × Time`
- Volume changes from IVF infusion and urine output

The derivation is performed symbolically using [SymPy](https://www.sympy.org/) and documented in [`formula_derivation.ipynb`](formula_derivation.ipynb) ([HTML version](formula_derivation.html)).

---

## Comparison with the Adrogué-Madias Equation

### The Adrogué-Madias Formula

The Adrogué-Madias equation (1997) estimates the expected change in serum sodium from **1 liter** of any given infusate:

$$\Delta[\text{Na}^+]_s = \frac{[\text{Na}^+]_{\text{infusate}} - [\text{Na}^+]_{\text{serum}}}{\text{TBW} + 1}$$

Where the `+1` accounts for the 1 L of infusate added to the body water pool. For solutions containing both Na⁺ and K⁺ (e.g., Ringer's lactate), the infusate cation concentration is the sum Na⁺ + K⁺. To plan therapy, the volume of infusate needed is:

$$\text{Volume needed (L)} = \frac{\text{Desired } \Delta[\text{Na}^+]_s}{\Delta[\text{Na}^+]_s \text{ per 1 L}}$$

**Example:** A 70 kg young male (TBW = 42 L) with serum Na⁺ of 120 mEq/L receiving 3% saline (513 mEq/L):

$$\Delta[\text{Na}^+]_s = \frac{513 - 120}{42 + 1} = \frac{393}{43} \approx 9.1 \text{ mEq/L per liter}$$

To raise sodium by 6 mEq/L, approximately 0.66 L (660 mL) of 3% saline would be needed.

### Side-by-Side Comparison

| Feature | Adrogué-Madias | This Algorithm |
|---------|---------------|----------------|
| **Urine output** | Not modeled | Explicitly included (`URate`) |
| **Urine electrolytes** | Not modeled | Explicitly included (`UNa`, `UK`) |
| **Infusion duration** | Not modeled (per-liter estimate) | Explicitly included (`Time`) |
| **Infusion rate** | Not modeled | Explicitly included (`IVFRate`) |
| **System assumption** | Closed system | Open system |
| **Primary output** | ΔNa per 1 L infusate | IVFRate needed, or NaEnd after infusion |
| **Clinical use** | Quick bedside estimate | Continuous infusion planning |

### Key Distinction: Open vs. Closed System

The Adrogué-Madias equation treats the body as a **closed system** — it does not account for ongoing renal or extrarenal water and electrolyte losses. This is a recognized limitation: in SIADH, where patients excrete concentrated urine rich in Na⁺ and K⁺, the Adrogué-Madias equation can substantially underestimate the rate of sodium correction, raising the risk of overcorrection and osmotic demyelination syndrome.

The equations derived here model the body as an **open system**: urinary losses of both volume (`URate`) and electrolytes (`UNa`, `UK`) are incorporated directly. This mirrors the approach of the Voets equation, which has been shown to be superior to Adrogué-Madias in SIADH (r = 0.94 vs. r = 0.49).

### Clinical Implications

- **Use Adrogué-Madias** for a rapid bedside estimate of sodium change per liter of infusate, particularly when urine data are unavailable.
- **Use this algorithm** when urine output and urine electrolyte concentrations are known (e.g., from a spot urine or 24-hour collection), for continuous infusion planning, or in conditions like SIADH where renal electrolyte handling significantly affects the correction trajectory.
- Regardless of which formula is used, **frequent monitoring of serum sodium** remains essential. Correction limits of ≤10 mEq/L per 24 hours (≤8 mEq/L in high-risk patients) should not be exceeded.

### References

1. Adrogué HJ, Madias NE. Aiding fluid prescription for the dysnatremias. *Intensive Care Med.* 1997.
2. Liamis G, et al. Therapeutic approach in patients with dysnatraemias. *Nephrol Dial Transplant.* 2006.
3. Sterns RH. Disorders of plasma sodium. *N Engl J Med.* 2015.
4. Voets PJGM, et al. Comparing the Voets equation and the Adrogué-Madias equation for predicting plasma sodium response in SIADH. *PLoS One.* 2021.
5. Cherukuri VKR, et al. Comparison of Voets and Adrogué-Madias equations in children with SIAD. *Pediatr Nephrol.* 2026.
6. Adrogué HJ, Tucker BM, Madias NE. Diagnosis and management of hyponatremia: a review. *JAMA.* 2022.

---

## Calculator Implementations

Two implementations of the algorithm are provided:

### HTML / Preact (current) — `preact/`

A single-file web app built with [Preact](https://preactjs.com/) and [KaTeX](https://katex.org/). No build step or server required — open `preact/index.html` directly in any browser.

- Live result updates as you type
- Two modes: calculate required IVF rate, or predict resulting serum [Na]
- IVF solution presets (3% NaCl, NS, LR, ½NS, D5W, custom)
- TBW calculation from weight and patient demographics
- Color-coded safety warnings (green / amber / red) based on correction rate per 24 h
- Collapsible background section with rendered equations

### Java Swing (legacy) — `java/`

A desktop GUI application built with Java Swing (NetBeans project). Implements the same open-system formulas in a native window.

- Entry point: `java/src/nacalc/Main.java`
- Build: `ant -f java/build.xml`
- Requires Java 8+

---

## Formula Derivation

- `formula_derivation.ipynb` — Jupyter notebook with symbolic derivation (SymPy)
- `formula_derivation.html` — Rendered HTML version of the notebook
