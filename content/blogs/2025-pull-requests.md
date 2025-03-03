---
title: "Crafting Good Pull Requests"
date: 2025-03-03T00:00:00+00:00
author: "fryorcraken"
tags:
  - software engineering
summary: "What criteria make a pull request *good*."
toc: false
---

# Crafting Good Pull Requests

Pull Requests are at the core of team software development.
They are the gateway from local development to upstream merge; moving a code change from one brain to many.

It can be difficult to find the right balance when creating pull requests, in terms of size, scope, and readiness.

In this article, I provide a principle first approach to make pull requests an effective collaboration tool and increment to software delivery

## Branching Strategy

This article assumes the usage of GitHub Flow strategy:

- One long-running branch: `master` from which releases happen and is always ready (do not break `master`)
- Developers create a branch from `master` to isolate their work, and the branch name must be descriptive (e.g., feat/name-of-feature)
- A Pull Request is opened to merge the developer's branch to `master`

This is the simplest branching strategy when working, as a team, on medium codebases.

## Objectives

The primary objective of a Pull Request (in its literal sense) is to request a series of patches to be pulled into `master`.
The reasons to pull patches, i.e., introduce a code change, can be:

- **For users**: Fix a bug, introduce or modify a feature that will later be released and made available to users
- **For maintainers**: Refactor to make a specific piece of code easier to maintain or extend, stabilize CI, add tests to improve quality

Naturally, a PR should aim to avoid doing the opposite: introducing bugs, feature regression, destabilizing CI;
and avoid doing neither of those: aimless refactor, introduced a feature worthless to users.

## PRs that are Not to be Merged Against `master`

The goal definition above states that all PRs are to be merged against `master`.
However, there are a few reasons why a maintainer may decide to open a PR that breaks this statement:

1. The PR is not to be merged: This PR can be opened to demonstrate a code change or a PoC.
   The PR itself is documentation for other or future developers and will be closed without being merged.
2. The PR is not for `master`: The PR is opened to be merged against a different branch, against which a PR may already exist.
3. The PR is not ready to be merged: The PR either depends on other PRs to be merged first or has been published to get some direction/help from other team members.

For each of those scenario, I recommend:

1. If a PR is never to be merged, then it MUST remain as a draft until closed.
2. This is poor practice, that can lead to confusion and unnecessary overheard for both author and reviewers.
3. It is best to keep the PR as a draft until it is considered ready to merge by the author.

## Pull Request Optimization

The goal of a Pull Request is to ensure that the objectives defined above are reached:

- There is a consensus from code owners that this PR is either useful to users or maintainers
- And, no regression or bug has been introduced, thanks to the tests run during the CI and review from other maintainers

As a PR is meant to pull changes into `master`, then to optimize a PR is to shorten the time between publication and merging.

There are two steps to get a PR merged:

1. CI needs to pass
2. Reviewers need to approve

To ensure the CI passes, one needs to not break any tests.
More generally, maintainers need to ensure the CI can run fast, and doesn't have failing or flacky tests.

To get approval, one can optimize for review.
Here are practices I encourage adopting to get PRs approved in a timely fashion:

### PR Not Against `master`

As mentioned in the previous sections, a PR is meant to introduce changes into `master`, useful to the users or maintainers.

A PR that is not opened against `master` but against another branch does neither of the goals above; it just brings changes into another branch.

Asking other maintainers to review a non-`master` PR is a distraction:

- It does not reflect the code that will ultimately be merged against `master`
- It may serve as a double review, as someone will have to review the code (again) before it gets merged against `master`

Hence, it is recommended to only ask for reviews of PRs against `master`.

If a maintainer prefers to open a non-`master` PR as part of their workflow, it is their choice.
However, they should be considerate of the time of other reviewers and avoid asking for reviews.

### PR Author Mandate

To optimize for fast approval, the author should ensure the PR has the following properties:

#### Code Changes are Quick to Read

- Total code changes under 400 lines
- Or, commits can be reviewed individually
- Even better: both

#### Benefit is Obvious

There is a clear benefit for the user or maintainer.
PR description and/or commit descriptions should be used to highlight the benefit.

#### Changes are Coherent as a Set

It is easier to review if the reviewer only has to think about one specific issue/concern/feature.

Refactoring, formatting, and other minor issues being fixed at the same time create distractions for the reviewers.
It increases the mental load of the reviewer.

Such changes should be segregated from the main change the PR addresses.
There are two strategies to do so:

- **Separate minor changes or refactoring in the commits**: this means keeping a clean commit tree and using a rebase strategy so the segregation remains as the PR is updated.
- **OR, create a new separate PR that addresses said minor changes**. If the original PR cannot be merged until other PRs are, then it should be marked as a draft until ready to merge.

#### Controversial Changes are Isolated

Similar to the property above, if a PR introduces several changes, with some trivial while others are controversial, then the merge of all changes will be delayed.

To optimize a PR, it is necessary to separate changes that will be merged with little controversy from changes that need extensive discussion.

### PR Reviewers Mandate

In return, the reviewers should ensure:

#### PRs are Reviewed Daily

PRs should be reviewed daily to ensure that changes are promptly merged.

#### PRs are Fully Reviewed

By fully reviewing a PR, the reviewers give the opportunity to the author to tackle all comments at once, maximizing the chance of the PR being approved at the next review.

However, if there is a fundamental issue with the code change that leads to a rewrite of the full PR, then it is acceptable for the reviewer to not spend time reviewing code that will be modified and only highlight the fundamental issue.

#### Expectations are Clear

Last but **not least**, the reviewers should clearly state their expectations from the author.
I believe a comment usually falls within five categories:

- Blocking: The current code introduces a bug or does not match what the PR should deliver; a change is expected.
- Question: Clarification requested to ensure that reviewer and author are on the same page; a reply is preferred.
- Nitpick: A change that does not impact the code behavior, that may be purely style or preferred practice; change or reply are optional.
- Context: Some information about the piece of code or feature, to help the author acquire knowledge about either; reply is optional.
- Future Improvement: Some tidying up, clean or logic improvements that seem to be out of the scope of the PR; may be tackled now, with a follow-up PR, tracked in an issue or discussed further for planning.

One should be able to deduce what type of comment can lead to a "Request Change" review result:

- Critical: This should lead to "request change"
- Question: While it should not lead to "request change", an "approval" may be held off until clarifications are made.
- Nitpick/Context/Future Improvement: Should not impact the review result either way.

Which means that a reviewer marking a PR as "Request Change" must clearly state why with the presence of blocking comments.
Similarly, a reviewer commenting on a PR without "approving" must clearly state why with the presence of questions or desired clarifications.

# Conclusion

Opening or reviewing a PR is collaborating with other maintainers towards the best outcome.
The same way code needs to be readable for later maintenance, a PR, or set of changes, needs to also be readable for review.
Small incremental PRs will lead to faster merge and feature delivery.

On the other hand, reviewing a PR is providing feedback on said code, and it is essential to state expectations and make expectations ambiguous.