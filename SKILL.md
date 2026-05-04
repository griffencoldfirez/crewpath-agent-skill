---
name: CrewPath Job Extractor
description: Extracts maritime job postings, training schedules, and recruitment details from URLs, text, or images, then converts them into clean CrewPath-ready JSON.
version: 1.5.0
author: GriffenColdFirez
---

# CrewPath Job Extractor Skill

## Purpose

Extract maritime job postings, training announcements, and crew recruitment notices from text, images, or URLs. Output clean JSON only. No explanations.

## Rules

- Output valid JSON only. Nothing else.
- No markdown, no code fences, no comments.
- Missing fields use null. Never guess or fabricate.
- If input is an image, read all visible text first before extracting.
- If URL is blocked, return the error JSON below.

## Steps

1. Read the input fully.
2. Detect: is it a job post, training announcement, or recruitment notice?
3. Detect: is it maritime-related? Philippines-sourced? Bilingual (Tagalog+English)?
4. Extract all available fields into the JSON below.
5. Check legitimacy. Flag if: no company name, no contact info, asks for payment, vague salary only.
6. Set confidence_score: 1.0 = all fields found, 0.5 = partial, 0.0 = almost nothing.
7. Recommend if CrewPath should post it.

## Philippine Maritime Notes

- Manning Agency = POEA-accredited recruiter. Extract separately from company/shipowner.
- POEA License format: XXX-LB-XXXXXX-XX
- Common rank codes: C/O, 2/O, 3/O, C/E, 2/E, 3/E, AB, OS, BSN, OILER, WIPER, CADET
- Common docs: STCW, GMDSS, PDOS, PEME, OEC, SIRB, Basic Safety
- Tagalog hints: "Naghahanap ng" = Hiring, "Mga Kinakailangan" = Requirements, "Makipag-ugnayan" = Contact
- Red flag: any post asking for placement fee violates POEA rules.

## Output JSON

{
  "schema_version": "1.5",
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
  "join_date": null,
  "contract_duration": null,
  "rotation": null,
  "port_of_deployment": null,
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
  "documents_required": [],
  "certifications_required": [],
  "experience_required": [],
  "other_requirements": [],
  "benefits": [],
  "salary_range": null,
  "deadline": null,
  "phone": null,
  "viber": null,
  "email": null,
  "facebook_page": null,
  "address": null,
  "website": null,
  "application_method": "",
  "walk_in_schedule": null,
  "location": null,
  "source_url": null,
  "source_language": "en",
  "is_bilingual": false,
  "is_maritime_related": false,
  "is_philippines_sourced": false,
  "confidence_score": 0.0,
  "legitimacy_status": "",
  "legitimacy_flags": [],
  "legitimacy_reason": "",
  "should_post": false,
  "post_category": "",
  "post_reason": ""
}

## Legitimacy Status Values

Use one of: verified, unverified, suspicious, red_flag

## Post Category Values

Use one of: crew_vacancy, training, cadetship, pooling

## Error JSON

If URL is blocked or unreadable return this:

{
  "schema_version": "1.5",
  "source_type": "url",
  "source_url": "",
  "error_code": "URL_INACCESSIBLE",
  "error_reason": "Cannot access URL. Please paste the post text or upload a screenshot.",
  "confidence_score": 0.0,
  "should_post": false,
  "post_reason": "No data extracted."
}
