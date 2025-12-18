# SleepNav Zone 1 - Sleep Health Check Logic

## Overview
A patient self-triage kiosk for community pharmacy sleep services that generates clinical handover codes based on symptoms, risk factors, and patient demographics.

---

## Assessment Flow

### **Step 1: Demographics**
Collect:
- Age (used to determine elderly status: ≥65 years)
- Gender (M/F/Other)
- Height & Weight (Metric: cm/kg OR Imperial: ft/in, st/lbs)
- Calculate **BMI** = weight(kg) / height(m)²

### **Step 2: Primary Triage**
Patient selects main complaint:

1. **"Can't Sleep"** → Insomnia pathway
2. **"Falling Asleep / Snoring"** → OSA pathway
3. **"Restless Legs / Other"** → Direct to `AMBER-3-OTHER`

---

## **Pathway A: INSOMNIA**

### Question:
*"How long have you had trouble sleeping?"*

### Logic:
```
IF duration < 3 months (ACUTE):
    IF patient_age >= 65:
        → AMBER-1-INSOMNIA (Safety Warning - Do not sell Diphenhydramine)
    ELSE:
        → GREEN-1-INSOMNIA (Safe Supply - Short-term Medication Available)

IF duration >= 3 months (CHRONIC):
    → RED-1-INSOMNIA (Refer to CBT-i)
```

---

## **Pathway B: OSA (Obstructive Sleep Apnea)**

### Part 1: Clinical Red Flags (YES/NO toggles)
1. Has anyone seen you **stop breathing** while asleep?
2. Do you have **high blood pressure**?
3. Do you **snore loudly**?
4. Is your **collar size over 17 inches** (43cm)?

### Part 2: Epworth Sleepiness Scale (pESS)
8 scenarios rated 0-3:
- 0 = Never doze
- 1 = Slight chance
- 2 = Moderate chance
- 3 = High chance

**Total Score:** 0-24

### OSA Risk Calculation Logic:
```
HIGH RISK if ANY of these are true:
1. Apnea flag = TRUE (witnessed breathing cessation)
2. pESS > 10 (severe excessive daytime sleepiness)
3. Snore = TRUE AND BP = TRUE
4. Snore = TRUE AND BMI > 30
5. Neck size > 17" AND pESS > 10

IF HIGH RISK:
    → RED-2-OSA (Recommend HSAT - Home Sleep Test)

ELSE IF Snore = TRUE (but not high risk):
    → AMBER-2-OSA (Dentist Referral for Snoring Assessment)

ELSE:
    → GREEN-2-OSA (Reassurance - No Immediate Concerns)
```

---

## Final Handover Screen

### Code Format:
`#[COLOR]-[NUMBER]-[COMPLAINT]`

Examples:
- `#RED-1-INSOMNIA`
- `#AMBER-2-OSA`
- `#GREEN-1-INSOMNIA`

### Display Logic:

**RED/AMBER Codes:**
- Icon: 🛑 (Stop sign)
- Title: "Medication may not be the right answer."
- Body: "Based on your profile, simple sleeping pills may be ineffective or unsafe. Please show the code below to the Pharmacist for a clinical assessment."

**GREEN Codes:**
- Icon: ✅ (Check mark)
- Title: "Short-term relief available."
- Body: "Please speak to the counter staff for product advice."

### Clinical Summary Box:
- Age
- BMI
- Elderly Patient (≥65): Yes/No
- Sleepiness Score (if OSA): X/24
- Recommended Action (depends on code)

### Action Mapping:
```
RED-1   → "Refer to CBT-i (Cognitive Behavioral Therapy)"
RED-2   → "Recommend HSAT (Home Sleep Test)"
AMBER-1 → "Safety Warning - Do not sell Diphenhydramine"
AMBER-2 → "Dentist Referral for Snoring Assessment"
AMBER-3 → "GP Referral Required"
GREEN-1 → "Safe Supply - Short-term Medication Available"
GREEN-2 → "Reassurance - No Immediate Concerns"
```

---

## Complete Code Matrix

| Code | Condition | Action |
|------|-----------|--------|
| **RED-1-INSOMNIA** | Chronic insomnia (≥3 months) | CBT-i referral |
| **RED-2-OSA** | High OSA risk | Home sleep test |
| **AMBER-1-INSOMNIA** | Acute insomnia + elderly (≥65) | No diphenhydramine |
| **AMBER-2-OSA** | Snoring only (low risk) | Dentist referral |
| **AMBER-3-OTHER** | Restless legs/other complaints | GP referral |
| **GREEN-1-INSOMNIA** | Acute insomnia + non-elderly | OTC sleep aid OK |
| **GREEN-2-OSA** | No significant OSA risk | Reassurance |

---

## Additional Features

- **Inactivity timeout:** 5 minutes → returns to idle screen
- **Start Over:** Reset button available on all screens
- **Unit switching:** Metric ↔ Imperial for height/weight
- **PWA:** Installable app with offline functionality
