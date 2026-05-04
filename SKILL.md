---
name: CrewPath Job Extractor
description: >
  Extracts maritime job postings, training schedules, crew recruitment notices,
  and manning agency announcements from URLs, raw text, screenshots, or images.
  Converts all extracted data into clean, validated, CrewPath-ready JSON for
  review and publishing. Handles Philippine maritime context, Tagalog/English
  bilingual posts, and POEA-regulated agency detection.
version: 2.0.0
author: GriffenColdFirez
tags: [maritime, crewpath, extraction, philippines, seafarer, manning-agency]
---

# CrewPath Job Extractor Skill

## Purpose

This skill powers CrewPath's data ingestion pipeline. It enables the agent to
read any maritime hiring post — whether a messy Facebook screenshot, a
recruitment flyer image, a plain-text announcement, or a live URL — and
transform it into a clean, structured JSON object that can be directly reviewed
and published inside the CrewPath platform.

---

## Scope of Coverage

This skill handles the following content types:

| Content Type             | Examples                                              |
|--------------------------|-------------------------------------------------------|
| Job Postings             | Crew vacancies, shipboard positions, officer roles    |
| Manning Agency Notices   | Walk-in schedules, pooling announcements              |
| Training Announcements   | STCW refresher, Basic Safety Training, SOLAS courses  |
| Deployment Schedules     | Join date, deployment duration, rotation              |
| Recruitment Drives       | Cadetship programs, bulk hiring, career fairs         |

---

## Input Types Supported

- *Raw text* — pasted job post, announcement, or notice
- *Image / Screenshot* — recruitment flyer, Facebook post, Viber blast
- *URL* — publicly accessible job listing page (note: Facebook URLs may be blocked)
- *Structured data* — partially filled forms or CSV rows needing normalization

For images: use vision to read all visible text, logos, contact details, and
dates before extraction.

For URLs: attempt to fetch the page. If blocked (e.g., Facebook, robots.txt),
return a structured error block inside the JSON explaining the limitation.

---

## Agent Behavior

### Step 1 — Classify the Content

Before extracting, determine:
- Is this *maritime-related*? (vessel operations, seafaring, crewing)
- Is this *training-related*? (STCW, MARINA, maritime certification)
- Is this *Philippines-focused*? (POEA, OWWA, flag state PH, Manning Agency)
- Is this *bilingual*? (Tagalog + English mixed content)

Set the appropriate boolean flags and industry value in the output.

### Step 2 — Extract All Available Fields

Pull every available detail. Do not skip fields just because they seem minor.
For missing fields, use null — never guess or fabricate data.

Key fields to prioritize:
- *Company / Vessel / Manning Agency name*
- *All open positions* (with rank codes if detectable)
- *Deployment schedule* (join date, rotation, contract duration)
- *Requirements* (documents, licenses, certifications, experience)
- *Contact details* (phone, email, address, Facebook page, Viber)
- *Application method* (walk-in, email, online form, referral)
- *Deadline* (application cutoff if stated)
- *Salary / Allotment* (if disclosed)

### Step 3 — Legitimacy Check

Evaluate whether the listing appears trustworthy:

| Signal                        | Impact        |
|-------------------------------|---------------|
| Named company / agency        | Positive      |
| POEA license number present   | Strong positive |
| Verifiable contact info       | Positive      |
| Specific vessel/trade route   | Positive      |
| No contact info at all        | Negative      |
| Asks for upfront payment      | RED FLAG      |
| Vague "earn $5000/month" only | Suspicious    |
| No company name               | Negative      |

Set legitimacy_check.status to one of: verified, unverified, suspicious, red_flag.

### Step 4 — CrewPath Recommendation

Based on extraction quality and legitimacy, recommend whether this listing
should be published on CrewPath. Provide a clear, one-sentence reason.

---

## Philippine Maritime Context

When extracting Philippine-sourced listings, apply these domain rules:

- *Manning Agency* = a POEA-accredited recruiter; always extract separately from shipowner/company
- *Rank codes* follow STCW/MARINA standards (e.g., 2/O, BSN, AB, OILER, C/E, CADET)
- *POEA License* format: XXX-LB-XXXXXX-XX — extract if present
- *Common certifications*: STCW Basic Safety, GMDSS, PDOS, PEME, OEC, SIRB
- *Flag states*: Note if Philippine flag (MARINA) or foreign flag (MLC 2006)
- *Tagalog terms to translate*: "Naghahanap ng" = Hiring for, "Mga Kinakailangan" = Requirements, "Makipag-ugnayan" = Contact

---

## Output Rules

### STRICT: Return valid JSON only.

Never return:
- Markdown prose or explanation outside the JSON
- Code fences (` 
json `) wrapping the output
- Comments inside JSON (`// comment`)
- Trailing commas
- Broken or unescaped quote characters
- Placeholder strings like `"TBD"` or `"N/A"` — use `null`

If input is insufficient to extract meaningful data, return the JSON with all
fields null and `confidence_score: 0` with a descriptive `legitimacy_check.reason`.

---

## JSON Output Schema

```json
{
  "schema_version": "2.0",
  "extracted_at": "",
  "source_type": "",

  "type": "job_listing",
  "title": "",
  "company_name": "",
  "agency_name": "",
  "poea_license": null,

  "industry": "",
  "vessel_type": null,
  "flag_state": null,
  "trade_route": null,

  "deployment_schedule": {
    "join_date": null,
    "contract_duration": null,
    "rotation": null,
    "port_of_deployment": null
  },

  "job_positions": [
    {
      "position": "",
      "rank_code": null,
      "vacancies": null,
      "salary": null
    }
  ],

  "is_training_related": false,
  "course_name": null,
  "training_provider": null,
  "training_schedule": null,
  "training_location": null,
  "training_fee": null,
  "certification_issued": null,

  "description": "",

  "qualifications": [],
  "requirements": {
    "documents": [],
    "certifications": [],
    "experience": [],
    "other": []
  },

  "benefits": [],
  "salary_range": null,
  "deadline": null,

  "contact": {
    "phone": null,
    "viber": null,
    "email": null,
    "facebook_page": null,
    "address": null,
    "website": null
  },

  "application_method": "",
  "walk_in_schedule": null,

  "location": null,
  "source_url": null,
  "source_language": "en",
  "is_bilingual": false,

  "is_maritime_related": false,
  "is_philippines_sourced": false,

  "confidence_score": 0,

  "legitimacy_check": {
    "status": "",
    "flags": [],
    "reason": ""
  },

  "crewpath_recommendation": {
    "should_post": false,
    "category": "",
    "reason": ""
  }
}

---

## Field Reference

| Field | Type | Notes |
|---|---|---|
| schema_version | string | Always "2.0" |
| extracted_at | string | ISO 8601 datetime of extraction |
| source_type | string | "text", "image", "url", "structured" |
| type | string | "job_listing" or "training_announcement" |
| poea_license | string\|null | Extract if present; format XXX-LB-XXXXXX-XX |
| flag_state | string\|null | e.g., "Panama", "Philippines", "Marshall Islands" |
| trade_route | string\|null | e.g., "International", "Domestic", "Asia-Pacific" |
| rank_code | string\|null | Standard STCW rank abbreviation |
| vacancies | integer\|null | Number of open slots per position |
| source_language | string | "en", "tl", "mixed" |
| confidence_score | float | 0.0–1.0; reflects extraction completeness |
| legitimacy_check.status | string | "verified", "unverified", "suspicious", "red_flag" |
| legitimacy_check.flags | array | List of detected warning signals |
| crewpath_recommendation.category | string | "crew_vacancy", "training", "cadetship", "pooling" |

---

## Error Handling

If the URL is inaccessible (blocked, login-required, 404):

{
  "schema_version": "2.0",
  "source_type": "url",
  "source_url": "https://...",
  "error": {
    "code": "URL_INACCESSIBLE",
    "reason": "Facebook URLs are blocked by robots.txt. Please paste the post text or share a screenshot.",
    "suggested_action": "Paste raw text or upload image"
  },
  "confidence_score": 0,
  "crewpath_recommendation": {
    "should_post": false,
    "reason": "Cannot extract data from inaccessible source."
  }
}

---

## Example Legitimacy Flags

"legitimacy_check": {
  "status": "suspicious",
  "flags": [
    "no_poea_license_mentioned",
    "requests_upfront_payment",
    "no_verifiable_contact"
  ],
  "reason": "Post requests placement fee before boarding, which violates POEA regulations."
}

---

## Skill Notes

- Always prioritize *seafarer safety* — flag any post that violates POEA
  no-placement-fee rules or requests money from applicants.
- When vessel type is unclear, infer from rank codes (e.g., BOSUN suggests
  deck dept; OILER suggests engine dept).
- For training posts, is_training_related must be true and
  crewpath_recommendation.category must be "training".
- Confidence score guide: 0.9–1.0 = all key fields extracted;
  0.6–0.89 = most fields extracted; 0.3–0.59 = partial; <0.3 = poor.
