# SleepNav Zone 1 - Modification Brief

## Version 2.0 Development Specification

**Date:** December 2025
**Author:** Adrian Zacher, PhD (Candidate), BSPSS
**Document Type:** Developer Brief
**Priority:** Medium (Pre-Pilot Refinement)

## Executive Summary

This brief specifies modifications to SleepNav Zone 1 (v1.0) to reduce patient burden, mitigate litigation
risk, and improve throughput while maintaining clinical safety standards. Changes affect two core areas:
age capture methodology and OSA sleepiness assessment.

**Key Changes:** 1. Replace exact age input with age bracket selection (3 options) 2. Replace 8-item
pictorial Epworth Sleepiness Scale (pESS) with 2-question validated screening approach 3. Update OSA
risk logic to accommodate simplified sleepiness assessment 4. Preserve all other functionality (BMI
calculation, Beers Criteria, insomnia pathway, handover codes)

**Estimated Development Time:** 4-6 hours
**Testing Required:** Full pathway validation, clinical logic verification

## Section 1: Clinical Rationale

### 1.1 Problem Statement

**Current v1.0 Implementation Issues:**

**Age Capture:** - Numeric entry feels invasive to patients at point of first interaction - Creates
psychological barrier at start of journey - Only two clinical decision points required (under-18 exclusion,
≥65 Beers flag) - Exact age provides minimal additional clinical value to pharmacist triage decision

**Sleepiness Assessment:** - 8-item pESS requires 60-90 seconds of 2-minute total assessment time - High
patient burden increases abandonment risk - Pictorial format (Ghiassi et al.) creates potential IP/licensing
exposure - Use of formal validated instrument increases liability if scoring errors occur - Situational
granularity (8 scenarios) exceeds triage requirements

### 1.2 Evidence Base for Changes

**Age Bracket Approach:** - Pharmacy triage requires binary decisions, not continuous variables - Under-
18 exclusion = legal/ethical requirement (not clinical judgment call) - Beers Criteria threshold = fixed at
65 years (American Geriatrics Society, 2023) - Age bracket disclosure shown to reduce form
abandonment in health screenings (UX research)

**Simplified Sleepiness Assessment:** - Core OSA symptom = Excessive Daytime Sleepiness (EDS), not
situation-specific drowsiness - PHQ-9 frequency language validated across 50+ million primary care
encounters - Two-question screening maintains sensitivity while reducing burden - NICE Clinical
Guideline [NG202] supports symptom-based OSA screening in non-specialist settings - Professional


standard: pharmacist triage ≠ diagnostic assessment

### 1.3 Risk Mitigation

**Litigation Protection:** - Generic screening language (non-proprietary) eliminates IP exposure -
Simplified tool = fewer opportunities for administration error - Positioned as “initial screening” not
“clinical assessment” - Pharmacist clinical override explicitly retained - All handover codes still trigger
appropriate escalation

**Clinical Safety:** - OSA risk logic maintains sensitivity for high-risk cases - Beers Criteria protection
unchanged - Witnessed apnea still triggers immediate RED-2 (unchanged) - Multi-factor risk assessment
preserved (anatomical + symptomatic)

## Section 2: Technical Specifications

### 2.1 Age Capture Modifications

**REMOVE:** - Screen 2 numeric age input field - Validation logic for age range (18-120) - Age display as
exact number on handover

**REPLACE WITH:**

**Screen 2: Age Bracket Selection**

**Question Text:**
“Which age group are you in?”

**UI Element:** Three large tap buttons (vertical stack, minimum 60px height each)

**Options:** 1. “Under 18” 2. “18-64” 3. “65 or older”

**Button Styling:** - Pharmacy green (#00A651) border when selected - Clear tap feedback - Single-select
only (radio button behavior) - Minimum touch target: 44px × 280px

**Logic:**

// Selection handling
function selectAgeGroup(bracket) {
state.age_bracket = bracket;

if (bracket === 'under-18') {
// Immediate exit - no further assessment
showExitScreen();
} else if (bracket === '18-64') {
state.is_elderly = false;
continueToNextScreen();
} else if (bracket === '65-plus') {
state.is_elderly = true;
continueToNextScreen();
}
}

// Exit screen for under-
function showExitScreen() {
// Display message
title: "This service is for adults only"
body: "Please speak to the pharmacist for advice."
action: "CLOSE" button returns to idle screen


}

**State Variables:** - state.age_bracket = ‘under-18’ | ‘18-64’ | ‘65-plus’ - state.is_elderly
= boolean (TRUE if 65-plus, FALSE otherwise)

**Handover Display Changes:**

**OLD:**

Age: 55
Elderly Patient (≥65): No

#### NEW:

Age Group: 18-
Elderly Patient (≥65): No

OR

Age Group: 65 or older
Elderly Patient (≥65): Yes

### 2.2 Sleepiness Assessment Modifications

**REMOVE:** - Entire Screen 4B-2 (8-item pESS with sliders) - All pESS image placeholders (pess-1.png
through pess-8.png) - pESS total calculation logic (sum of 8 sliders) - pESS score display on handover

**REPLACE WITH:**

**Screen 4B-2: Sleepiness Screening (New Implementation)**

**Part 1: Initial Question**

**Question Text:**
“Do you struggle to stay alert during the day?”

**UI Element:** Two large Yes/No buttons (horizontal layout)

**Buttons:** - “NO” (left) - neutral gray when not selected - “YES” (right) - pharmacy green when selected -
Minimum touch target: 140px × 60px each

**Logic:**

function answerAlertness(response) {
state.has_eds = response; // boolean

if (response === false) {
// Skip frequency question, proceed to OSA logic
state.eds_frequency = null;
submitOSA();
} else {
// Show frequency question
showFrequencyQuestion();
}
}


**Part 2: Frequency Question (Conditional - Only if YES to Part 1)**

**Question Text:**
“How often does this happen?”

**UI Element:** Three large tap buttons (vertical stack)

**Options:** 1. “Rarely” 2. “Several days a week” 3. “Nearly every day”

**Button Styling:** - Full width - Minimum 60px height each - 16px spacing between buttons - Pharmacy
green border when selected - Clear visual feedback on tap

**Logic:**

function selectFrequency(frequency) {
state.eds_frequency = frequency; // 'rarely' | 'several-days' | 'nearly-daily'

// Proceed to OSA risk calculation
submitOSA();
}

**State Variables:** - state.has_eds = boolean (true/false) - state.eds_frequency = ‘rarely’ |
‘several-days’ | ‘nearly-daily’ | null

**Screen Layout:**

┌─────────────────────────────────────┐
│ Part 2: Sleepiness Screening │
│ │
│ Do you struggle to stay alert │
│ during the day? │
│ │
│ ┌─────────┐ ┌─────────┐ │
│ │ NO │ │ YES │ │
│ └─────────┘ └─────────┘ │
│ │
│ [Conditional: If YES selected] │
│ │
│ How often does this happen? │
│ │
│ ┌──────────────────────────────┐ │
│ │ Rarely │ │
│ └──────────────────────────────┘ │
│ ┌──────────────────────────────┐ │
│ │ Several days a week │ │
│ └──────────────────────────────┘ │
│ ┌──────────────────────────────┐ │
│ │ Nearly every day │ │
│ └──────────────────────────────┘ │
│ │
│ [COMPLETE ASSESSMENT button] │
└─────────────────────────────────────┘

### 2.3 Updated OSA Risk Logic

**Clinical Decision Algorithm (v2.0)**

Replace existing pESS-based logic with frequency-based assessment:

function submitOSA() {


let osaRisk = 'LOW';

// Priority 1: Witnessed apnea (immediate high risk)
if (state.flags.apnea) {
osaRisk = 'HIGH';
}

// Priority 2: Severe sleepiness (standalone red flag)
else if (state.has_eds && state.eds_frequency === 'nearly-daily') {
osaRisk = 'HIGH';
}

// Priority 3: Anatomical risk factors
else if (state.flags.snore && state.flags.bp) {
osaRisk = 'HIGH';
}
else if (state.flags.snore && state.bmi_score > 30) {
osaRisk = 'HIGH';
}

// Priority 4: Combined moderate sleepiness + anatomical risk
else if (state.has_eds && state.eds_frequency === 'several-days') {
if (state.flags.snore || state.flags.neck) {
osaRisk = 'HIGH';
}
}

// Priority 5: Combined mild sleepiness + multiple anatomical risks
else if (state.has_eds && state.eds_frequency === 'rarely') {
if (state.flags.snore && state.flags.neck) {
osaRisk = 'MODERATE'; // AMBER-
}
}

// Priority 6: Snoring only (no significant sleepiness)
else if (state.flags.snore) {
osaRisk = 'MODERATE'; // AMBER-
}

// Code assignment
if (osaRisk === 'HIGH') {
state.final_code = 'RED-2';
} else if (osaRisk === 'MODERATE') {
state.final_code = 'AMBER-2';
} else {
state.final_code = 'GREEN-2';
}

showHandover();
}

**Logic Summary Table:**

```
Condition Code Action
```
```
Witnessed apnea = YES RED-2 HSAT referral
```
```
EDS “Nearly every day” RED-2 HSAT referral
```
```
Snore + High BP RED-2 HSAT referral
```
```
Snore + BMI >30 RED-2 HSAT referral
```
```
EDS “Several days a week” + (Snore OR Collar >17”)RED-2 HSAT referral
```

```
Condition Code Action
```
```
EDS “Rarely” + Snore + Collar >17” AMBER-2Dentist referral
```
```
Snore only (no EDS or EDS=NO) AMBER-2Dentist referral
```
```
None of above GREEN-2 Reassurance
```
### 2.4 Handover Screen Modifications

**REMOVE:** - “Sleepiness Score: XX/24” display line

**ADD (Conditional):** - Display sleepiness information only if patient answered YES to alertness question

**NEW Handover Display Format:**

Clinical Summary
────────────────
Age Group 18-
BMI 28.
Elderly Patient (≥65) No
[CONDITIONAL]
Daytime Alertness Struggles to stay alert
Frequency Several days a week
[END CONDITIONAL]

Recommended Action Recommend HSAT

**Conditional Display Logic:**

// Only show sleepiness data if patient has EDS
if (state.has_eds && state.complain_type === 'OSA') {
document.getElementById('summary-alertness-row').style.display = 'flex';
document.getElementById('summary-alertness').textContent = 'Struggles to stay
alert';
document.getElementById('summary-frequency').textContent =
formatFrequency(state.eds_frequency);
} else {
document.getElementById('summary-alertness-row').style.display = 'none';
}

function formatFrequency(freq) {
const map = {
'rarely': 'Rarely',
'several-days': 'Several days a week',
'nearly-daily': 'Nearly every day'
};
return map[freq] || '';
}


## Section 3: Screen Flow Changes

### 3.1 Updated Navigation Map

**OLD Flow (v1.0):**

Screen 1: Idle
↓
Screen 2: Demographics (age numeric, gender, height, weight)
↓
Screen 3: Primary Triage
↓ OSA pathway
Screen 4B-1: Clinical Questions (4 toggles)
↓
Screen 4B-2: pESS Assessment (8 sliders)
↓
Screen 5: Handover

**NEW Flow (v2.0):**

Screen 1: Idle
↓
Screen 2: Demographics (age BRACKET, gender, height, weight)
↓ [IF age = under-18 → EXIT SCREEN]
↓
Screen 3: Primary Triage
↓ OSA pathway
Screen 4B-1: Clinical Questions (4 toggles) [UNCHANGED]
↓
Screen 4B-2: Sleepiness Screening (2 questions) [NEW]
↓
Screen 5: Handover

### 3.2 Modified Screens Detail

**Screen 2: Demographics**

**Changes:** - Replace age numeric input with 3-button age bracket selector - Add conditional exit path for
under-18 selection - All other elements unchanged (gender, height, weight, unit toggle)

**Exit Screen (NEW - triggered by under-18 selection):** - Icon: - Title: “This service is for adults ℹ️
only” - Body: “Please speak to the pharmacist for advice.” - Single button: “CLOSE” (returns to idle
screen, resets all state)

**Screen 4B-2: Sleepiness Screening**

**Changes:** - Complete replacement of 8-slider pESS interface - New 2-question sequential flow -
Conditional display (frequency question only if EDS=YES) - Preserve “COMPLETE ASSESSMENT”
button at bottom

## Section 4: Testing Requirements

### 4.1 Unit Testing

**Age Bracket Selection:** - [ ] Under-18 triggers immediate exit - [ ] Exit screen displays correctly - [ ]
Exit screen CLOSE button resets to idle - [ ] 18-64 sets is_elderly = FALSE - [ ] 65-plus sets is_elderly =
TRUE - [ ] Age bracket displays correctly on handover


**Sleepiness Assessment:** - [ ] NO to alertness question skips frequency, proceeds to OSA logic - [ ] YES
to alertness question reveals frequency options - [ ] Each frequency option sets state correctly - [ ]
Conditional handover display shows/hides alertness data appropriately

### 4.2 Pathway Testing

**Complete flow testing required for all code outcomes:**

```
Test Case Inputs Expected Code
```
```
Witnessed apnea Apnea=YES, any other values RED-2-OSA
```
```
Severe sleepiness EDS=“Nearly every day”, no other flagsRED-2-OSA
```
```
Anatomical high risk Snore + BP, EDS=NO RED-2-OSA
```
```
Obesity risk Snore + BMI=32, EDS=NO RED-2-OSA
```
```
Moderate EDS + snoring EDS=“Several days”, Snore=YES RED-2-OSA
```
```
Mild EDS + multiple flags EDS=“Rarely”, Snore+Collar AMBER-2-OSA
```
```
Snoring only Snore=YES, EDS=NO, no other flags AMBER-2-OSA
```
```
Low risk All flags=NO, EDS=NO GREEN-2-OSA
```
**Edge cases:** - [ ] Patient selects YES to alertness but abandons before frequency → how to handle? - [ ]
Rapid tap on age brackets → ensure single selection enforced - [ ] Back navigation from sleepiness screen
→ state preservation

### 4.3 Integration Testing

- Insomnia pathway completely unchanged and functional
- Other pathway (RLS) completely unchanged and functional
- BMI calculation still working with age brackets
- Unit toggle (metric/imperial) still functional
- Beers Criteria flag (is_elderly) correctly applied in insomnia pathway
- All handover codes display correctly
- Auto-reset timer still functional (5 minutes)
- Manual close button still resets state

### 4.4 UI/UX Testing

- Age bracket buttons clearly tappable (>44px targets)
- Under-18 exit message clear and non-alarming
- Sleepiness questions use clear, accessible language
- Frequency buttons easily distinguishable
- Conditional frequency question appears smoothly (no UI jump)
- Handover screen conditional display works without layout shift
- All text readable on iPad screen in landscape mode


## Section 5: Regulatory Considerations

### 5.1 MHRA Classification Impact

**Device Classification:** Class I Clinical Decision Support Software (unchanged)

**Regulatory Status:** - Simplified screening approach does NOT change device class - Tool remains
“information provision for clinical decision support” - Pharmacist clinical override explicitly retained -
No automated diagnostic claims

**Documentation Updates Required:**

1. **Technical File Amendment:**
    - Update clinical algorithm description
    - Document rationale for pESS removal (burden reduction, IP protection)
    - Reference PHQ-9 validation literature for frequency language
    - Maintain evidence trail for age bracket approach
2. **Instructions for Use (IFU):**
    - Update screen flow documentation
    - Clarify screening vs assessment positioning
    - Update pharmacist training materials
3. **Risk Management File:**
    - Document reduced IP litigation risk
    - Note simplified tool = fewer administration errors
    - Maintain OSA risk sensitivity analysis
    - Update user error analysis (fewer input points)

### 5.2 UKCA Registration

**IF SleepNav Zone 1 is NEW registration (not update to Snorer CDSS):**

**Question for decision:** Does this share same Level 2 GMDN® Category as Snorer Pharmacy CDSS?

**IF YES (same GMDN Category):** - Consider delaying registration until April 2026 - Only one annual
fee (£300) will cover both devices - Pilot testing can proceed with unregistered version (internal use only)

**IF NO (different GMDN Categories):** - Register before 31 March 2026 (£261 one-time fee) - Avoids
dual annual fees (£300 × 2 = £600/year from April 2026)

**IF this is UPDATE to existing Snorer CDSS registration:** - Changes to clinical logic = chargeable
modification (£261) - Update conformity documentation - Maintain version control (v1.0 → v2.0)

### 5.3 Clinical Governance

**Professional Standards Compliance:**

- Tool remains compliant with GPhC standards for pharmacy triage
- Evidence-based approach (PHQ-9 validated language)
- Appropriate scope for pharmacy practice
- Does not exceed community pharmacist competence boundaries

**Liability Protection:**

- Generic screening language = no IP infringement risk
- Simplified tool = lower operational error risk
- Clear positioning as initial screening, not diagnostic assessment


- Pharmacist retains full clinical decision authority
- All handover codes trigger appropriate professional intervention

## Section 6: Implementation Checklist

### 6.1 Pre-Development

- Clinical rationale documented
- Regulatory impact assessed
- Stakeholder approval obtained (BSPSS, pharmacy partners)
- GMDN Category confirmed for registration strategy

### 6.2 Development Phase

- Age bracket selection implemented
- Under-18 exit screen created
- pESS screens removed
- 2-question sleepiness screening implemented
- OSA risk logic updated and tested
- Handover display modifications complete
- State management verified
- All pathways tested (see Section 4)

### 6.3 Pre-Deployment

- Full regression testing complete
- iPad Safari compatibility verified
- Offline PWA functionality confirmed
- Clinical validation review (pharmacist walkthrough)
- Documentation updated (spec, IFU, training materials)
- MHRA registration decision finalized

### 6.4 Deployment

- Upload to pharmacy.snorer.com
- Pharmacy pilot sites identified
- Training materials distributed
- Feedback mechanism established
- Version control maintained (v1.0 archived, v2.0 deployed)

## Section 7: Success Criteria

### 7.1 Performance Metrics

**Primary Outcomes:** - Assessment completion time: <2 minutes (target: 90 seconds) - Patient
abandonment rate: <10% - Code distribution matches clinical expectations (majority AMBER/RED)

**Secondary Outcomes:** - Pharmacist satisfaction with clinical information provided - Zero operational
errors in first 100 patient uses - No patient complaints regarding invasiveness


### 7.2 Clinical Validation

**Pharmacist Feedback Points:** - Age bracket provides sufficient clinical context (Yes/No) - Sleepiness
frequency information useful for triage (1-5 scale) - Handover codes align with clinical judgment
(concordance %)

**Safety Monitoring:** - Zero missed high-risk OSA cases (RED-2 sensitivity maintained) - Beers Criteria
flag correctly applied in all ≥65 cases - Appropriate escalation for witnessed apnea cases (100%)

## Section 8: Version Control

**Document Version:** 2.
**SleepNav Zone 1 Software Version:** v2.0 (proposed)
**Previous Version:** v1.0 (deployed December 2025)

**Change Log:**

```
Date Version Changes Author
```
```
Dec 2025 1.0 Initial deployment with 8-item pESS AZ
```
```
Dec 2025 2.
Age brackets + simplified sleepiness (this
brief)
```
#### AZ

## Appendices

### Appendix A: Clinical References

1. **American Geriatrics Society (2023).** American Geriatrics Society 2023 Updated AGS Beers
    Criteria® for Potentially Inappropriate Medication Use in Older Adults. _Journal of the American_
    _Geriatrics Society_ , 71(7), 2052-2081.
2. **Kroenke, K., Spitzer, R. L., & Williams, J. B. (2001).** The PHQ-9: validity of a brief depression
    severity measure. _Journal of General Internal Medicine_ , 16(9), 606-613.
       - _Rationale: Establishes validity of “rarely/several days/nearly every day” frequency_
          _language_
3. **National Institute for Health and Care Excellence (2021).** Obstructive sleep apnoea/hypopnoea
    syndrome and obesity hypoventilation syndrome in over 16s. _NICE Guideline [NG202]_.
       - _Supports symptom-based OSA screening in non-specialist settings_
4. **Selsick, H., et al. (2024).** Primary Care Algorithm for Insomnia Management.
    - _Unchanged - supports existing insomnia pathway_
5. **Ghiassi, R., et al. (2011).** The pictorial Epworth Sleepiness Scale.
    - _Referenced for comparison - NOT used in v2._

### Appendix B: PHQ-9 Frequency Language Validation

The frequency descriptors used in this modification (“Rarely”, “Several days a week”, “Nearly every


day”) are derived from the PHQ-9 (Patient Health Questionnaire-9), which has been:

- Administered to >50 million patients worldwide
- Validated across 50+ languages
- Proven to reduce defensive responses in screening contexts
- Established as primary care standard for symptom frequency assessment

This language choice is deliberate to: 1. Leverage validated, non-defensive phrasing 2. Align with
familiar primary care screening standards 3. Reduce litigation risk through use of established clinical
language 4. Maintain professional credibility with pharmacists who recognize this scale

### Appendix C: Age Bracket Rationale

**Clinical Decision Points:**

```
Age Bracket Clinical Significance System Action
```
```
Under 18 Legal/ethical exclusion from service Immediate exit, refer to pharmacist
```
```
18-64 Standard adult population Beers flag = FALSE, normal pathway
```
#### 65+

```
Beers Criteria threshold for anticholinergic
risk
```
```
Beers flag = TRUE, AMBER-1 if acute
insomnia
```
**Additional Considerations:** - OSA risk does increase with age, but not linearly enough to warrant
granular brackets - Pharmacist clinical judgment accounts for age-related risk within brackets - Exact age
precision does not materially change triage outcome for pharmacy scope - Privacy benefit outweighs
marginal clinical utility of exact age

## Contact & Approvals

**Technical Lead:** Adrian Zacher
**Email:** [contact details]
**Organization:** British Society of Pharmacy Sleep Services (BSPSS)

**Approvals Required:** - [ ] Clinical validation review (BSPSS clinical advisory board) - [ ] Regulatory
compliance review (MHRA strategy) - [ ] Pharmacy partner consultation (pilot site feedback) - [ ]
Technical feasibility confirmation (developer)

**Estimated Development Timeline:** - Specification review: 1-2 days - Development: 4-6 hours - Testing:
2-3 hours - Deployment: 1 hour

**Target Completion:** Q1 2026 (pre-pilot deployment)

#### END OF BRIEF


