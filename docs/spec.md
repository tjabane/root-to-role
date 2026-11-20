# Tech career skill tree — draft specification

**Status:** Draft  
**Date:** 2026-11-20  
**Project:** Web application for learning, building, and finding relevant tech roles

## Purpose

Help a person answer: **What should I learn or build next to qualify for more of
the tech roles I want?** The application uses job requirements from the Gold
layer, evidence from GitHub repositories selected by the user, and the user's
own goals to create a skill tree. As the user learns and builds, the tree and
recommended job listings update.

The product reports evidence of fit and gaps. It does not predict the user's
probability of being hired.

The proposed job-context MCP server is a **separate project**. This application
may consume the same job data or a future shared read API, but its first release
does not depend on that MCP server.

## Product loop

1. The user signs in with GitHub and chooses repositories to analyse.
2. The user selects one or more target role families and enters skills or
   experience that repository analysis cannot establish.
3. The application shows a skill tree: shared foundations, role-specific
   branches, evidence of current progress, and a useful next step.
4. The user learns a skill and builds or improves a project.
5. A rescan updates the evidence and the tree.
6. The application shows relevant current listings, explains the match and
   remaining gaps, and links to each job's application page.
7. The user applies through the employer's page and continues the loop.

Initial role families are software development, QA and test automation, and
data engineering. A user may select several to find skills that transfer
across them. The role taxonomy and prerequisites are curated and versioned;
job titles and extracted terms provide market evidence, not the entire graph.

## Weekly experiment mode

The app can support a time-boxed, repeatable career experiment. Example
question: **Can a biology graduate with no professional tech experience earn
an IT assessment or interview by learning and building with this app?** The
experiment tests a concrete path and outcome for one participant; it does not
establish that the same result is likely for everyone with that background.

At the start, record a dated baseline: education, prior tech experience,
existing skills, location, target role families, available hours per week,
GitHub repositories, and any constraints the participant wants considered.
Define the observation period and primary outcome before work begins. For the
example above, the primary outcome is an employer assessment invitation or
interview invitation for a tech role; an offer is a later outcome.

Each weekly check-in records:

- Learning completed and time spent, with links to notes or exercises when
  available.
- Projects built or improved, linked GitHub revisions, and the skill evidence
  the app observed or the participant supplied.
- Skill-tree changes and the recommendation shown at the start of the week.
- Listings shown, applications the participant chose to submit, and the date
  and status of each application.
- Assessments, interviews, feedback, and offers, with participant-controlled
  visibility for personal information.

Show a timeline and funnel from **learned → built → demonstrated → applied →
assessment → interview → offer**. Report elapsed weeks, effort, applications,
and outcomes alongside the project work. Keep nonresponses and rejections in
the record so a weekly write-up does not selectively report successes. The
participant can export a dated summary for a public weekly experiment while
choosing which private details to omit.

The participant can log a product pain point during any step: what they tried,
where they got stuck, the workaround, and a proposed improvement. Link the note
to the relevant screen or recommendation and the week it occurred. A weekly
review groups repeated problems and records which product change was made, so
using the app becomes a source of concrete design feedback.

Provide a shareable weekly summary with the goal, work completed, project
evidence, application funnel, result, and next step. Support a short visual
summary suitable for a video update and a longer written account. Publication
is always initiated by the participant; drafts omit personal application
details, private repository content, and employer correspondence unless the
participant explicitly includes them.

Experiment mode is an optional workflow. Ordinary users can use the skill tree
and listing recommendations without publishing a case study.

## First-release experience

### Onboarding and GitHub connection

- Sign in with GitHub. Request read-only access needed for the repositories
  the user selects. Public repositories are sufficient for the first release;
  private repositories require a separate, explicit opt-in later.
- Let the user select repositories, exclude a repository, trigger a rescan,
  disconnect GitHub, and delete derived profile data.
- Let the user enter target roles, location preferences, and skills or
  experience that code cannot demonstrate. Keep self-reported claims separate
  from repository evidence.

### Skill tree

- A node represents a skill, a prerequisite, or a project milestone. A skill
  may appear in several role paths without being duplicated in the graph.
- Show the market signal for each skill: distinct relevant vacancies requiring
  it, the denominator, coverage by role family, and example source evidence.
  Distinguish explicit requirements from technologies merely mentioned in a
  description.
- Show progress as **not started**, **learning** (user reported), **observed in
  code**, or **demonstrated by project**. These states describe evidence, not a
  certification of mastery. Display confidence and the supporting repository
  or user claim.
- Recommend a next skill or project using the selected role families, current
  gaps, prerequisites, and the number of relevant roles it could help cover.
  Explain the recommendation in plain language and allow the user to choose a
  different branch.

### Repository analysis

- Inspect selected repositories for languages, dependencies, tests, CI,
  documentation, and implemented features. Prefer evidence from working
  project structure and tests over language or line counts alone.
- Record evidence with repository, revision, file or feature reference,
  observed timestamp, and confidence. Let the user correct an incorrect
  inference. A dependency alone is weak evidence of proficiency.
- Never treat code as proof of formal qualifications, employment history, or
  production experience. Do not run arbitrary repository code during analysis.
- Store only the derived evidence needed for progress and matching by default;
  repository access and any retained content must be visible to the user.

### Projects and listings

- Project cards state what to build, which skills the work would demonstrate,
  and what evidence the app will look for (for example, tests, a documented API,
  or a working data pipeline). Project templates are curated; recommendations
  are tailored to the user's gaps.
- Listing cards show title, company, location when known, application URL,
  last-seen time, current status, matching evidence, and important gaps.
- Rank listings by role relevance and supported skill evidence. Show a clear
  explanation and avoid a numeric “chance of getting hired.”
- Present a listing as **ready to apply** only when the current-listing
  projection supports an open state and the URL is a job-specific application
  or detail URL. Show unknown-status listings separately with a prompt to
  verify the employer page. Clicking a link leaves the application.

## Job data contract and quality rules

The existing Gold technology-jobs table supplies title, company, location,
source text, qualifications, experience, skills, technologies, evidence,
review status, and extraction lineage. Gold rows are immutable extraction
versions. The current-listing projection stores lifecycle status separately.

- Resolve each Gold row through its `sourcePk` and `sourceSk` to the Silver
  record, then use Silver's stable `identityKey` to connect observations from
  different scrape runs to the current-listing projection.
- Deduplicate repeated observations of the same vacancy before computing skill
  frequency or match counts. Select the appropriate Gold extraction for each
  source observation, preferring a repaired URL version where one exists.
- Keep `required` and `preferred` requirements separate. Preserve the exact
  source evidence and the extraction or review status. Exclude or clearly flag
  uncertain extractions in aggregate recommendations.
- Report both vacancy coverage and employer coverage so one employer's many
  postings do not masquerade as broad market demand. Show sample sizes and
  dates; avoid claims about the whole tech market from this source set.
- Use `CurrentListingsTable` for current status. Gold `listingStatus` and
  `statusAsOf` describe extraction time. Preserve unknown and missing values
  rather than treating them as open, closed, or zero.
- Use the job-specific `sourceUrl` for application links after URL repair
  rules. `collectionUrl` is a source or collection page; `canonicalUrl` is the
  vacancy detail page when available. Historical URL errors are documented in
  the [Gold URL audit](https://github.com/tjabane/melo-kgodumodumo/blob/master/docs/job-url-audit-2026-11-05.md).
- Treat location and salary as optional. Source descriptions can contain
  scraped form text and encoding errors; show concise, evidence-backed fields
  in the UI rather than raw descriptions by default.

The [Gold schema](https://github.com/tjabane/melo-kgodumodumo/blob/master/infra/README.md) and [listing lifecycle contract](https://github.com/tjabane/melo-kgodumodumo/blob/master/docs/job-lifecycle.md)
are the source contracts for implementation.

## Recommendation logic

For a selected set of role families, calculate skill coverage from distinct
relevant vacancies, with required and preferred counts shown separately.
Consider a skill a gap when the user has no evidence for it or only weak
evidence. Rank possible next steps using:

1. Coverage across the selected role families and employers.
2. Prerequisites already met and the effort to reach a demonstrable milestone.
3. The user's expressed interest and preferred location.
4. Evidence quality, extraction review state, and listing recency.

Keep the underlying counts visible. Do not infer that a common skill alone
qualifies a person for a senior role. For example, a shared Python and SQL
foundation might support development, QA automation, and data engineering,
while each branch retains its own project and experience requirements. That
relationship must be established from the job data before it appears as a
market claim.

## System boundaries

| Component | Responsibility |
| --- | --- |
| Web client | Onboarding, skill tree, project cards, evidence review, listing cards. |
| GitHub connection | Authentication and read-only access to user-selected repositories. |
| Evidence service | Inspect repositories, retain derived evidence, process rescans and corrections. |
| Career model | Curated role taxonomy, prerequisites, project templates, and ranking rules. |
| Job read model | Deduplicated, searchable projection from Gold, Silver identity, and current listings. |
| User data store | Goals, selected repositories, self-reported skills, progress, and preferences. |

The Gold table's existing indexes support source-identity and review-status
lookups. Company, skill, and role search for this application needs a separate
read model or search index; a table-wide scan is not the interactive query path.

## Acceptance criteria for the first release

- A GitHub user can select public repositories and see which skills were
  inferred, why, and with what confidence; they can remove or correct evidence.
- A user can select multiple role families and see shared versus branch skills
  with vacancy counts and example job evidence.
- A user can see at least one next project recommendation tied to specific gaps
  and can rescan a repository to update progress.
- A user can view matched listings with match reasons, gaps, last-seen time,
  current status, and a job-specific application URL when available.
- Duplicate Gold versions or repeated scrapes of one vacancy do not inflate
  requirement counts. Unknown listing status is never labelled open.
- Disconnecting GitHub stops future repository access and offers deletion of
  stored derived evidence.
- A participant can create a dated baseline, record weekly learning, projects,
  applications, and interview outcomes, then export an experiment summary with
  private fields excluded.
- A participant can log friction at the point it occurred and include resolved
  pain points and product changes in a weekly summary or video outline.

## Later phases

- Private repository analysis with explicit user selection and retention
  controls.
- More role families, learning resources, richer project evaluation, and
  optional CV or experience evidence supplied by the user.
- Application tracking and, later, auto apply. Any future submission flow must
  show the exact job and application content to the user and obtain approval
  before submitting each application.

## Open decisions

- Define the initial role-family taxonomy and the minimum vacancy sample
  required to show a demand signal.
- Choose the search projection and refresh cadence for Gold and lifecycle
  updates.
- Define repository evidence rules for each first-release skill and the
  minimum evidence needed for “demonstrated by project.”
- Decide how to handle employers whose current-listing status remains unknown
  because their scraper cannot prove a complete snapshot.
