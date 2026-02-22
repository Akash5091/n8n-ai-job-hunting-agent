# ATS Scoring Logic

This document explains the Applicant Tracking System (ATS) scoring algorithm used by the n8n AI Job Hunting Agent to match candidates with job postings.

## Overview

The ATS scoring system evaluates job postings against candidate profiles using multiple scoring components, each weighted to create a comprehensive match score. The final score ranges from 0-100.

## Scoring Components

### 1. Skills Match (Weight: 30%)

**Description:** Evaluates how well the candidate's skills align with the job requirements.

**Calculation:**
- Extract required skills from job posting (using Claude AI)
- Extract candidate skills from resume/profile
- Calculate intersection: `(matching_skills / total_required_skills) * 100`
- Apply skill level bonus: +10 points if candidate has advanced proficiency
- Cap at 100 points

**Example:**
```
Required Skills: [Java, Spring Boot, Docker, Kubernetes, AWS]
Candidate Skills: [Java (Expert), Spring Boot (Advanced), Docker, Python]

Matching Skills: 3/5 = 60%
Advanced Level Bonus: +10 (candidate has expert Java)
Skills Score: 70/100
```

### 2. Experience Match (Weight: 25%)

**Description:** Evaluates years of relevant experience.

**Calculation:**
```
if candidate_years >= required_years:
    score = 100
elif candidate_years >= (required_years * 0.75):
    score = 85
elif candidate_years >= (required_years * 0.5):
    score = 70
elif candidate_years >= (required_years * 0.25):
    score = 50
else:
    score = 25
```

**Bonuses:**
- +5 points for each additional year beyond required
- +10 points if candidate has managed teams (for leadership roles)
- +10 points if candidate has startup experience (for startup companies)

### 3. Education Match (Weight: 15%)

**Description:** Evaluates educational qualifications.

**Scoring Table:**
| Requirement | BS/BA | MS | PhD | Bootcamp | Relevant Certs |
|-------------|-------|----|----|----------|----------------|
| High School | 90 | 100 | 100 | 85 | 80 |
| BS Required | 100 | 100 | 100 | 60 | 70 |
| MS Required | 70 | 100 | 100 | 40 | 60 |
| PhD Required | 40 | 80 | 100 | 20 | 40 |

**Bonuses:**
- +5 points if from top 50 university (MIT, Stanford, Berkeley, CMU, etc.)
- +5 points if relevant degree (CS, Engineering, Mathematics, etc.)

### 4. Location Match (Weight: 10%)

**Description:** Evaluates location fit.

**Scoring:**
- 100 points: Exact location match or remote friendly
- 85 points: Same state/country
- 70 points: Same time zone
- 50 points: Different time zone but willing to relocate
- 0 points: Location mismatch and not open to relocation

### 5. Seniority Level Match (Weight: 10%)

**Description:** Evaluates career level alignment.

**Levels:**
1. Junior (0-2 years)
2. Mid-Level (2-5 years)
3. Senior (5-10 years)
4. Staff/Principal (10+ years)

**Scoring:**
- 100 points: Exact level match
- 85 points: One level above required
- 70 points: One level below required
- 50 points: Two levels difference
- 25 points: More than two levels difference

### 6. Industry Experience (Weight: 10%)

**Description:** Evaluates experience in relevant industries.

**Scoring:**
- 100 points: Direct industry experience
- 80 points: Adjacent industry experience
- 60 points: Transferable skills from different industry
- 40 points: No direct experience but trainable

## Minimum Thresholds

Jobs are filtered before scoring based on minimum requirements:

```
min_skills_match = 50%          # Candidate must have 50% of required skills
min_experience = 0.5 * required # Candidate years >= 50% of required
min_education = bachelor        # Minimum education level
min_seniority_level = 1         # At least 1 level below required
```

## Final Score Calculation

```
final_score = (
    (skills_score * 0.30) +
    (experience_score * 0.25) +
    (education_score * 0.15) +
    (location_score * 0.10) +
    (seniority_score * 0.10) +
    (industry_score * 0.10)
)
```

## Score Interpretation

| Score Range | Rating | Action |
|-------------|--------|--------|
| 90-100 | Perfect Match | Immediate outreach |
| 80-89 | Excellent Match | Priority outreach |
| 70-79 | Good Match | Schedule for outreach |
| 60-69 | Fair Match | Optional outreach |
| 50-59 | Below Average | Not recommended |
| <50 | Poor Match | Skip |

## Configuration

These thresholds and weights can be customized in the workflow:

```json
{
  "ats_config": {
    "weights": {
      "skills": 0.30,
      "experience": 0.25,
      "education": 0.15,
      "location": 0.10,
      "seniority": 0.10,
      "industry": 0.10
    },
    "thresholds": {
      "min_skills_percentage": 0.50,
      "min_experience_ratio": 0.50,
      "action_score_threshold": 70,
      "priority_threshold": 80
    },
    "skill_bonuses": {
      "expert_level": 10,
      "per_additional_year": 5,
      "leadership_experience": 10,
      "startup_experience": 10
    }
  }
}
```

## Special Adjustments

### Industry-Specific Adjustments

**Tech Startups:**
- Increase startup experience weight from +10 to +20
- Reduce education weight from 15% to 10%
- Increase seniority flexibility: allow 2 levels difference

**Large Corporations:**
- Increase education weight from 15% to 20%
- Increase experience weight from 25% to 30%
- Reduce seniority flexibility: allow only 1 level difference

**Government/Regulated Industries:**
- Increase education weight from 15% to 25%
- Add security clearance match component (20%)
- Require minimum experience = required experience

### Role-Specific Adjustments

**Management Roles:**
- Add leadership experience component (weighted 15%)
- Add team size managed metric (weighted 10%)
- Reduce pure technical skills weight to 20%

**Specialized Roles (ML, Data Science):**
- Increase education weight for advanced degrees (MS/PhD bonus)
- Add research/publication bonus (+10 points if published)
- Add certification weight (+5 points per relevant certification)

## Debugging ATS Scores

To see the detailed breakdown of an ATS score, enable debug mode:

```
DEBUG_MODE=true
DEBUG_LEVEL=verbose
```

This will output:
```
{
  "job_id": "job_123",
  "candidate_id": "cand_456",
  "scores": {
    "skills": { "score": 70, "weight": 0.30, "contribution": 21 },
    "experience": { "score": 85, "weight": 0.25, "contribution": 21.25 },
    "education": { "score": 100, "weight": 0.15, "contribution": 15 },
    "location": { "score": 100, "weight": 0.10, "contribution": 10 },
    "seniority": { "score": 90, "weight": 0.10, "contribution": 9 },
    "industry": { "score": 80, "weight": 0.10, "contribution": 8 }
  },
  "final_score": 84.25,
  "recommendation": "Excellent Match - Priority Outreach",
  "reasons": [
    "Strong skills match (70%)",
    "Exceeds experience requirements",
    "Relevant education",
    "Perfect location match",
    "Appropriate seniority level"
  ],
  "concerns": [
    "Skills gap: missing experience with Kubernetes"
  ]
}
```

## Continuous Improvement

The ATS scoring algorithm is continuously improved based on:

1. **User Feedback**: If a candidate with low score gets hired, adjust weights
2. **Outcome Tracking**: Monitor which scores correlate with successful placements
3. **ML Model**: Future versions may use machine learning to optimize weights

## Integration with Claude AI

Claude is used in the ATS workflow for:

1. **Skill Extraction**: Natural language extraction of skills from job descriptions
2. **Experience Evaluation**: Semantic analysis of candidate experience
3. **Education Validation**: Recognition of various degree types and institutions
4. **Role Analysis**: Understanding role requirements and culture fit

## API Usage

ATS scoring is called via n8n webhook:

```bash
POST /webhook/ats-score
Content-Type: application/json

{
  "job_posting": {
    "title": "Senior Backend Engineer",
    "description": "...",
    "requirements": [...],
    "location": "San Francisco, CA"
  },
  "candidate": {
    "resume": "...",
    "skills": [...],
    "experience": 6,
    "location": "Remote, USA"
  }
}

Response:
{
  "score": 84.25,
  "rating": "Excellent Match",
  "components": {...},
  "recommendations": [...]
}
```
