# Mathematics & Reference Values - Doctor Review Document

> **Purpose:** This document lists every formula, constant, and medical reference range used
> in the critical tools suite. Please review for clinical accuracy and flag anything that
> needs correcting. Any corrections can be given back as a simple list
> (e.g. "Section X, value Y should be Z") and TDW will update the code.

---

## Table of Contents

1. [Dose Calculator](#1-dose-calculator)
2. [Peptide Reconstitution Calculator](#2-peptide-reconstitution-calculator)
3. [Half-Life / Plasma Concentration Curve](#3-half-life--plasma-concentration-curve)
4. [Estimated 1-Rep Max (Workout Logger)](#4-estimated-1-rep-max-workout-logger)
5. [Training Volume (Workout Logger)](#5-training-volume-workout-logger)
6. [Nutrition Tracker - Macro Ring & Averages](#6-nutrition-tracker--macro-ring--averages)
7. [Bloodwork Reference Ranges](#7-bloodwork-reference-ranges)
8. [Bloodwork Status Classification Logic](#8-bloodwork-status-classification-logic)
9. [Unit Conversions](#9-unit-conversions)
10. [Compound Database - Half-Lives & Default Doses](#10-compound-database--half-lives--default-doses)
11. [Frequency Definitions](#11-frequency-definitions)

---

## 1. Dose Calculator

### What it does
Takes a compound, the user's body weight (kg), and experience level to suggest a per-dose range.

### Formula

```
per_dose_min = dose_per_kg_low  ×  body_weight_kg
per_dose_max = dose_per_kg_high ×  body_weight_kg
```

Results are rounded to the nearest whole number.

### Dose-Per-Kg Ranges

| Category   | Experience   | Low (unit/kg) | High (unit/kg) |
|------------|-------------|---------------|----------------|
| Injectable | Beginner     | 3             | 5              |
| Injectable | Intermediate | 5             | 8              |
| Injectable | Advanced     | 8             | 12             |
| Oral       | Beginner     | 0.3           | 0.5            |
| Oral       | Intermediate | 0.5           | 0.8            |
| Oral       | Advanced     | 0.8           | 1.2            |
| Peptide    | Beginner     | 2             | 4              |
| Peptide    | Intermediate | 4             | 6              |
| Peptide    | Advanced     | 6             | 10             |

### Weekly total extrapolation

```
if frequency == "ed"   → weekly = per_dose × 7
if frequency == "eod"  → weekly = per_dose × 3.5
if frequency == "e3.5d"→ weekly = per_dose × 2
otherwise              → weekly = per_dose  (assumed once per week)
```

### Reference dose
Each compound also has a stored `defaultDoseMg` which is shown as a static "Ref. Dose" for comparison. This value is NOT weight-adjusted - it is the compound's commonly cited standard dose.

### Questions for the doctor
- Are the mg/kg ranges clinically reasonable across all three categories?
- The "Oral" range (0.3–1.2 mg/kg) is deliberately lower than injectables. Does this make sense given typical oral bioavailability?
- The "Peptide" range uses mcg/kg. Are 2–10 mcg/kg appropriate for common peptides (BPC-157, Ipamorelin, etc.)?

---

## 2. Peptide Reconstitution Calculator

### What it does
Tells the user how many insulin syringe units to draw for a desired mcg dose, given the vial size and bacteriostatic water volume.

### Formulas

```
concentration_per_ml = (vial_mg × 1000) / bac_water_ml
    (result in mcg/ml)

ml_needed = desired_dose_mcg / concentration_per_ml

syringe_units = ml_needed × 100
    (standard insulin syringe: 1 ml = 100 units)
```

### Example
- Vial: 5 mg, BAC water: 2 ml, Desired dose: 250 mcg
- Concentration = (5 × 1000) / 2 = 2500 mcg/ml
- ml needed = 250 / 2500 = 0.1 ml
- Syringe units = 0.1 × 100 = 10 units

### Questions for the doctor
- Is the `1 ml = 100 units` insulin syringe assumption correct? (We assume a standard U-100 syringe.)
- Any concern with the `mg × 1000 → mcg` conversion being applied universally?

---

## 3. Half-Life / Plasma Concentration Curve

### What it does
Models plasma concentration over time for repeated dosing of a compound, using exponential decay.

### Core decay formula (per dose)

```
concentration_at_time_t = dose × 0.5 ^ ((t - dose_time) / half_life_hours)
```

This is the standard first-order elimination half-life equation.

### Superposition for repeated doses

```
total_concentration(t) = Σ  dose × 0.5 ^ ((t - tᵢ) / t½)
                         for all dose times tᵢ where t ≥ tᵢ
```

Each dose is administered at intervals defined by the chosen frequency (see Section 11). All prior doses contribute residual concentration at time `t`.

### Chart sampling
- Points are computed every 6 hours from t=0 to the end of the chosen duration.
- The x-axis is `t / 24` (converted to days).
- Concentration values are rounded to 2 decimal places for display.

### Assumptions / limitations
- Assumes 100% bioavailability (no absorption lag, no first-pass metabolism).
- Assumes single-compartment pharmacokinetics.
- Does not model ester cleavage time for prodrugs (e.g. enanthate/decanoate esters).
- Does not account for saturation kinetics or non-linear clearance.

### Questions for the doctor
- Is first-order single-compartment decay an acceptable simplification for user education purposes?
- Should I add a lag time (Tmax) for ester-based injectables before decay begins?
- The chart shows steady-state buildup correctly - does the visual match expectations?

---

## 4. Estimated 1-Rep Max (Workout Logger)

### What it does
Estimates a user's one-rep max from a set performed at a given weight and rep count.

### Formula (Epley formula)

```
if reps == 1:
    estimated_1RM = weight

otherwise:
    estimated_1RM = weight × (1 + reps / 30)
```

This is the **Epley formula**, one of the most commonly used 1RM estimation methods.

### Example
- 100 kg × 8 reps → 1RM = 100 × (1 + 8/30) = 100 × 1.267 = **127 kg**

### PR detection
When a new workout is submitted, each exercise's best estimated 1RM is compared against the same exercise in all previous workouts. If the new 1RM exceeds the previous best, a "New PR" notification is displayed.

### Questions for the doctor
- The Epley formula is widely used. Is there a preferred alternative for this population?
- It tends to overestimate at very high rep ranges (15+). Should I cap the rep input?

---

## 5. Training Volume (Workout Logger)

### What it does
Calculates total training volume for a workout session.

### Formula

```
exercise_volume = Σ (reps × weight)  for each set in the exercise

session_volume = Σ exercise_volume   for all exercises in the workout
```

Units are in kg (total weight moved).

### Example
- Bench Press: 3×10 @ 80 kg → 3 × 10 × 80 = 2,400 kg
- Rows: 3×8 @ 70 kg → 3 × 8 × 70 = 1,680 kg
- Session volume = 4,080 kg

---

## 6. Nutrition Tracker - Macro Ring & Averages

### Default daily targets (hardcoded)

| Macro    | Target |
|----------|--------|
| Calories | 2,800  |
| Protein  | 200 g  |
| Carbs    | 350 g  |
| Fats     | 80 g   |

### Macro ring percentage

```
percentage = min(current / target, 1.0) × 100
```

Capped at 100% for the visual ring. Actual values are shown as `current / target`.

### Rolling averages

```
7-day average  = sum(last 7 entries) / count(last 7 entries)
30-day average = sum(last 30 entries) / count(last 30 entries)
```

All averages are rounded to the nearest whole number.

### Questions for the doctor
- Default targets are 2,800 kcal / 200g P / 350g C / 80g F. Are these reasonable defaults for a male bodybuilder?
- Should I make targets configurable per user?

---

## 7. Bloodwork Reference Ranges

All ranges are for **adult males**. Units follow UK/EU conventions.

### Hormones

| Marker               | Min    | Max    | Unit    | Warning High |
|----------------------|--------|--------|---------|-------------|
| Total Testosterone   | 8.64   | 29     | nmol/L  | -           |
| Free Testosterone    | 0.2    | 0.62   | nmol/L  | -           |
| Oestradiol (E2)      | 41     | 159    | pmol/L  | 200         |
| SHBG                 | 18     | 54     | nmol/L  | -           |
| Prolactin            | 86     | 324    | mIU/L   | 400         |
| LH                   | 1.7    | 8.6    | IU/L    | -           |
| FSH                  | 1.5    | 12.4   | IU/L    | -           |
| DHEA Sulphate        | 4.34   | 12.2   | umol/L  | -           |
| IGF-1                | 83     | 382    | ng/mL   | -           |
| Cortisol (AM)        | 166    | 507    | nmol/L  | -           |

### Liver

| Marker               | Min | Max | Unit   | Warning High |
|----------------------|-----|-----|--------|-------------|
| ALT (SGPT)           | 0   | 41  | U/L    | 60          |
| AST (SGOT)           | 0   | 40  | U/L    | 55          |
| Gamma GT             | 0   | 60  | U/L    | 80          |
| Alkaline Phosphatase | 30  | 130 | U/L    | -           |
| Total Bilirubin      | 0   | 21  | umol/L | -           |
| Albumin              | 35  | 50  | g/L    | -           |
| Total Protein        | 60  | 80  | g/L    | -           |

### Lipids

| Marker              | Min | Max | Unit    | Warning Low | Warning High |
|---------------------|-----|-----|---------|-------------|-------------|
| Total Cholesterol   | 0   | 5.0 | mmol/L  | -           | 6.2         |
| LDL Cholesterol     | 0   | 3.0 | mmol/L  | -           | 4.0         |
| HDL Cholesterol     | 1.0 | 100 | mmol/L  | 0.9         | -           |
| Triglycerides       | 0   | 1.7 | mmol/L  | -           | 2.3         |
| Cholesterol/HDL Ratio| 0  | 4.5 | (ratio) | -           | -           |

### Kidney

| Marker     | Min  | Max  | Unit    | Warning Low |
|------------|------|------|---------|-------------|
| Creatinine | 59   | 104  | umol/L  | -           |
| Urea (BUN) | 2.5  | 7.1  | mmol/L  | -           |
| eGFR       | 90   | 999  | mL/min  | 60          |
| Uric Acid  | 200  | 430  | umol/L  | -           |
| Sodium     | 136  | 145  | mmol/L  | -           |
| Potassium  | 3.5  | 5.1  | mmol/L  | -           |

### Thyroid

| Marker   | Min  | Max | Unit   |
|----------|------|-----|--------|
| TSH      | 0.27 | 4.2 | mIU/L  |
| Free T3  | 3.1  | 6.8 | pmol/L |
| Free T4  | 12   | 22  | pmol/L |

### Haematology

| Marker             | Min  | Max  | Unit       | Warning High |
|--------------------|------|------|------------|-------------|
| Haematocrit        | 0.40 | 0.54 | L/L        | 0.55        |
| Haemoglobin        | 130  | 180  | g/L        | 185         |
| Red Blood Cells    | 4.5  | 6.5  | ×10¹²/L   | 6.8         |
| White Blood Cells  | 3.5  | 11.0 | ×10⁹/L    | -           |
| Platelets          | 150  | 400  | ×10⁹/L    | -           |
| Mean Cell Volume   | 80   | 100  | fL         | -           |
| Mean Cell Hb       | 27   | 32   | pg         | -           |
| Ferritin           | 30   | 400  | ug/L       | -           |

### Other

| Marker           | Min  | Max   | Unit      | Warning Low | Warning High |
|------------------|------|-------|-----------|-------------|-------------|
| Fasting Glucose  | 3.9  | 5.5   | mmol/L    | -           | 6.9         |
| HbA1c            | 20   | 42    | mmol/mol  | -           | 48          |
| PSA              | 0    | 2.5   | ng/mL     | -           | 4.0         |
| Vitamin D        | 50   | 175   | nmol/L    | 30          | -           |
| Vitamin B12      | 187  | 883   | ng/L      | 150         | -           |
| Folate           | 3.9  | 26.8  | ug/L      | -           | -           |
| C-Reactive Protein| 0   | 5     | mg/L      | -           | 10          |
| Fasting Insulin  | 2.6  | 24.9  | mU/L      | -           | -           |

### Questions for the doctor
- Are these ranges appropriate for **adult males aged 20–50**?
- Testosterone range (8.64–29 nmol/L) - is this sufficiently wide for this demographic?
- Haematocrit warning at 0.55 L/L - is this the right threshold for flagging polycythaemia risk in AAS users?
- HDL max is set to 100 mmol/L (effectively no upper limit). Should there be a realistic cap?
- eGFR max is 999 - used because any value ≥ 90 is considered normal. Is this appropriate?
- Should creatinine ranges be wider for heavily muscled individuals?

---

## 8. Bloodwork Status Classification Logic

### How markers are classified as Normal / Warning / Danger

```
if value < min OR value > max → "danger"  (red)

if warningLow is defined AND value < warningLow → "warning"  (amber)
if warningHigh is defined AND value > warningHigh → "warning"  (amber)

if value is within 10% of min or max (edge of range) → "warning"  (amber)
    margin = (max - min) × 0.10
    if value < min + margin  → "warning"
    if value > max - margin  → "warning"

otherwise → "normal"  (green)
```

### Edge-of-range logic explained
Even if a value is technically within range, if it's in the bottom or top 10% of the reference range, it gets flagged as "warning" to draw attention. For example, for ALT (range 0–41 U/L), a value of 38 would be flagged because 41 - (41 × 0.10) = 36.9, and 38 > 36.9.

### Questions for the doctor
- Is the 10% edge-of-range warning appropriate, or is it causing unnecessary alarm?
- The logic checks warningLow/warningHigh first, then the 10% edge rule. Is this priority order correct?

---

## 9. Unit Conversions

| From | To   | Factor     |
|------|------|-----------|
| mg   | mcg  | × 1,000   |
| g    | mg   | × 1,000   |
| ml   | cc   | × 1 (identical) |
| oz   | ml   | × 29.5735 |
| kg   | lbs  | × 2.20462 |
| lbs  | kg   | × 0.453592|

These are standard SI/imperial conversions. No medical review needed unless any factor is incorrect.

---

## 10. Compound Database - Half-Lives & Default Doses

All compounds stored in the database with their pharmacokinetic properties.

### Injectables

| Compound                       | Half-Life (h) | Default Dose | Unit | Default Frequency |
|-------------------------------|---------------|-------------|------|------------------|
| Testosterone Enanthate         | 108           | 250         | mg   | e3.5d            |
| Testosterone Cypionate         | 120           | 250         | mg   | e3.5d            |
| Testosterone Propionate        | 20            | 100         | mg   | eod              |
| Testosterone Suspension        | 2             | 50          | mg   | ed               |
| Sustanon 250                   | 96            | 250         | mg   | e3.5d            |
| Nandrolone Decanoate (Deca)    | 360           | 300         | mg   | weekly           |
| Nandrolone Phenylpropionate    | 60            | 100         | mg   | eod              |
| Trenbolone Acetate             | 48            | 75          | mg   | eod              |
| Trenbolone Enanthate           | 168           | 200         | mg   | e3.5d            |
| Boldenone Undecylenate (EQ)    | 336           | 400         | mg   | weekly           |
| Masteron Propionate            | 48            | 100         | mg   | eod              |
| Masteron Enanthate             | 168           | 200         | mg   | e3.5d            |
| Primobolan (Meth. Enanthate)   | 240           | 400         | mg   | weekly           |
| DHB (Dihydroboldenone)         | 168           | 200         | mg   | e3.5d            |
| MENT (Trestolone Acetate)      | 8             | 10          | mg   | ed               |

### Orals

| Compound                       | Half-Life (h) | Default Dose | Unit | Default Frequency |
|-------------------------------|---------------|-------------|------|------------------|
| Anavar (Oxandrolone)           | 10            | 50          | mg   | ed               |
| Winstrol (Stanozolol)          | 9             | 50          | mg   | ed               |
| Dianabol (Methandrostenolone)  | 5             | 30          | mg   | ed               |
| Anadrol (Oxymetholone)         | 9             | 50          | mg   | ed               |
| Superdrol (Methyldrostanolone) | 8             | 10          | mg   | ed               |
| Turinabol                      | 16            | 40          | mg   | ed               |
| Halotestin (Fluoxymesterone)   | 6             | 20          | mg   | ed               |
| Proviron (Mesterolone)         | 12            | 50          | mg   | ed               |

### Peptides

| Compound                       | Half-Life (h) | Default Dose | Unit | Default Frequency |
|-------------------------------|---------------|-------------|------|------------------|
| BPC-157                        | 4             | 250         | mcg  | ed               |
| TB-500 (Thymosin Beta-4)       | 168           | 5           | mg   | e3.5d            |
| CJC-1295 DAC                   | 192           | 2           | mg   | weekly           |
| CJC-1295 no DAC (Mod GRF)     | 0.5           | 100         | mcg  | ed               |
| Ipamorelin                     | 2             | 200         | mcg  | ed               |
| GHRP-6                         | 2             | 200         | mcg  | ed               |
| GHRP-2                         | 1             | 200         | mcg  | ed               |
| PT-141 (Bremelanotide)         | 4             | 2           | mg   | eod              |
| Melanotan II                   | 1             | 250         | mcg  | ed               |

### HGH & Insulin

| Compound                       | Half-Life (h) | Default Dose | Unit | Default Frequency |
|-------------------------------|---------------|-------------|------|------------------|
| Human Growth Hormone (HGH)     | 3.5           | 4           | iu   | ed               |
| MK-677 (Ibutamoren)           | 24            | 25          | mg   | ed               |
| Insulin (Humalog/Rapid)        | 1             | 10          | iu   | ed               |

### PCT

| Compound                       | Half-Life (h) | Default Dose | Unit | Default Frequency |
|-------------------------------|---------------|-------------|------|------------------|
| Clomiphene (Clomid)           | 120           | 25          | mg   | ed               |
| Tamoxifen (Nolvadex)          | 168           | 20          | mg   | ed               |
| HCG                            | 36            | 500         | iu   | e3.5d            |
| Enclomiphene                   | 10            | 12.5        | mg   | ed               |

### Ancillaries

| Compound                       | Half-Life (h) | Default Dose | Unit | Default Frequency |
|-------------------------------|---------------|-------------|------|------------------|
| Anastrozole (Arimidex)        | 46            | 0.5         | mg   | e3.5d            |
| Exemestane (Aromasin)          | 24            | 12.5        | mg   | eod              |
| Letrozole (Femara)             | 48            | 0.5         | mg   | eod              |
| Cabergoline (Dostinex)         | 1,560 (65d)   | 0.25        | mg   | e3.5d            |
| Raloxifene                     | 28            | 60          | mg   | ed               |
| TUDCA                          | 4             | 500         | mg   | ed               |
| NAC                            | 5             | 600         | mg   | ed               |

### Fat-Burners / Thyroid

| Compound                       | Half-Life (h) | Default Dose | Unit | Default Frequency |
|-------------------------------|---------------|-------------|------|------------------|
| Clenbuterol                    | 36            | 40          | mcg  | ed               |
| T3 (Liothyronine / Cytomel)   | 24            | 50          | mcg  | ed               |
| T4 (Levothyroxine)            | 168           | 100         | mcg  | ed               |
| Cardarine (GW-501516)          | 20            | 20          | mg   | ed               |

### Questions for the doctor
- Are the half-life values correct for each compound?
- Are the default doses in a reasonable/commonly cited range?
- Are the default frequencies appropriate given the half-lives?
- Any compounds missing that should be added?
- Any compounds listed that should NOT be in a user-facing tool?

---

## 11. Frequency Definitions

| Code     | Meaning               | Hours Between Doses |
|----------|-----------------------|--------------------|
| ed       | Every day             | 24                 |
| eod      | Every other day       | 48                 |
| e3d      | Every 3 days          | 72                 |
| e3.5d    | Every 3.5 days (2×/wk)| 84                 |
| weekly   | Once per week         | 168                |
| biweekly | Once every 2 weeks    | 336                |

---

## How to Submit Corrections

Please review each section and return corrections in this format:

```
Section: [section number or name]
Item: [specific value, formula, or range]
Current value: [what it currently says]
Correct value: [what it should be]
Reason: [brief explanation]
```

Example:
```
Section: 10 - Compound Database
Item: Testosterone Enanthate half-life
Current value: 108 hours
Correct value: 96 hours
Reason: Literature consensus for TE is approximately 4–4.5 days
```

I will apply all corrections directly to the codebase.
