# Verification checklist — lab data (HL7 v2 & FHIR)

Before trusting an AI-generated lab result, run these five checks. You don't need to
code — you need to spot when the output deviates from what's correct.

## 1. Is the code real — or invented?

Does the code claim an official dictionary (LOINC, SNOMED) but use an invented code?

- ✅ FHIR: own code under own namespace — `"system": "urn:omar-dahir:lab-codes"`
- ✅ HL7 v2: `PIPETTE_MEAN_VOL^...^L` (`L` = local)
- ❌ `"system": "http://loinc.org"` + invented code, or HL7 `...^LN` (falsely claims LOINC)
- **Why:** the receiver looks the code up, finds nothing → the result is lost silently.
  The most dangerous, most invisible error.

## 2. Does the value type match the value?

- ✅ a number → `valueQuantity` / HL7 `NM`
- ✅ text or coded → `valueCodeableConcept` / HL7 `ST` or `CE`
- ❌ `valueQuantity: "Gram-positive"` / HL7 `NM | Positive`
- **Why:** the receiver tries to parse text as a number → rejected or dropped.

## 3. Does the subject point to the specimen — not the answer?

- ✅ `subject: Specimen/MIC-SA-2026-01` or `Device/P12556`
- ❌ `subject: "Staphylococcus aureus"` — that's the *answer*; it belongs in the value
- **Why:** a result that can't be linked to the right record is unusable.

## 4. Is status kept separate from the verdict?

- ✅ `status: final` **and** `interpretation: N`
- ❌ `interpretation: "validated"` — "validated" is status, not the verdict
- **Why:** they answer different questions; merging them loses the actual verdict.

## 5. Do value, unit, range and flag agree?

- ✅ `998.5 uL · range 994–1006 · N`; a CFU count uses `{CFU}/mL`
- ❌ a value outside the range but flagged normal
- **Why:** an internal contradiction gives the receiver a conflicting signal.

---

**The core instinct:** you don't memorise the rules — you feel when something deviates
from the correct pattern you've seen. That verification eye is the skill.
