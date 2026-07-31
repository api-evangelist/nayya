---
name: Configure benefits and get recommendations
description: Load an employer's benefits into Nayya, snapshot an employee's context, retrieve personalized recommendations, and record enrollments.
api: openapi/nayya-benefits-openapi.json
operations: [create-or-update-benefit, list-benefits-for-employer, create-snapshot, list-benefit-recommendations, create-enrollments]
---

# Configure benefits and get recommendations (Nayya Benefits + Choose APIs)

Use this skill after an employer and its employees exist (see the onboarding skill).

## Auth
Send `Authorization: Bearer <accessToken>` from `POST /token`. Benefits and Choose share the
same bearer; base URLs are `https://integrate.nayya.com/benefits` and `.../choose`.

## Steps
1. **Configure benefits** — `POST /employers/{employerId}/benefits` (`create-or-update-benefit`).
   Optionally use `GET /rule_templates` (`list-templates`) and `GET /carriers` (`list-carriers`)
   to reference rule templates and carriers when building benefits.
2. **Verify benefits** — `GET /employers/{employerId}/benefits` (`list-benefits-for-employer`).
3. **Snapshot the employee** — `POST /employers/{employerId}/employees/{employeeId}/snapshot`
   (`create-snapshot`) to capture the decision context.
4. **Get recommendations** — `GET /employers/{employerId}/employees/{employeeId}/recommendations`
   (`list-benefit-recommendations`, Choose API) for personalized benefit recommendations.
5. **Record enrollments** — `POST /employers/{employerId}/enrollments` (`create-enrollments`).
   Multiple enrollments are created atomically: if one fails, all fail and roll back.

## Rules
- Enrollment creation is atomic; treat a partial failure as a full failure and retry the batch.
- A `403` means the token is not authorized for the employer's organization.
- The Benefits API returns `429 Too Many Requests` under rate limiting — back off and retry.
- See `conventions/nayya-conventions.yml` and `errors/nayya-problem-types.yml`.
