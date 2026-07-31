---
name: Onboard an employer and its population
description: Authenticate to Nayya Integrate, create an employer, then load its employees and dependents ready for benefits decision support.
api: openapi/nayya-accounts-openapi.json
operations: [get-token, create-employer, create-employee, create-dependent, list-employees]
---

# Onboard an employer and its population (Nayya Accounts API)

Use this skill to stand up an employer in Nayya and map its workforce so the Choose
decision-support experience has the entities it needs.

## Auth
1. Exchange your `clientId` and `clientSecret` (issued by Nayya via a 1Password vault) for a
   bearer token: `POST /token` (`get-token`) on `https://integrate.nayya.com/accounts`.
   The response returns `accessToken` and `expiresAt`.
2. Send `Authorization: Bearer <accessToken>` on every subsequent call. Refresh before `expiresAt`.

## Steps
1. **Create the employer** — `POST /employers` (`create-employer`). Supply your external ID if you
   use one; the response returns the Nayya `employerId`.
2. **Create employees** — `POST /employers/{employerId}/employees` (`create-employee`) for each
   worker. Responses include the created `employeeId` (and your external IDs when supplied).
3. **Create dependents** — `POST /employers/{employerId}/employees/{employeeId}/dependents`
   (`create-dependent`) for each dependent.
4. **Verify** — `GET /employers/{employerId}/employees` (`list-employees`) to confirm the
   population loaded. Results are paginated (`page`, `perPage`, `sortBy`, `sortOrder`, `search`).

## Rules
- Authorization is scoped to the organization that owns the employer; a `403` means your token
  is not authorized for that employer.
- There is no idempotency-key contract — guard against duplicate creates yourself; a duplicate
  external ID returns `409 Conflict`.
- `422 Unprocessable Entity` means the request body failed validation; fix and resubmit.
- See `conventions/nayya-conventions.yml` for pagination/versioning and
  `errors/nayya-problem-types.yml` for the full status-code registry.
