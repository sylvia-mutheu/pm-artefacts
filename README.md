# PM artefacts

This repository holds worked examples of the documents I produce: specifications, acceptance criteria, API contracts, decision logs, test plans and delivery updates.

**These are not client deliverables.** Almost all of my real work sits under NDA and none of it is reproduced here. Each document below was written specifically for this repository, using an invented product, so that the format and the level of detail can be judged without exposing anyone's work. The invented product is a wallet top-up feature, chosen because payments is the domain I know best and because it has enough failure states to make the documents worth reading.

If you want to know whether I would be useful to your team, this is the honest way to find out. Read one of these and decide whether it is the kind of document your engineers would want to be handed.

| Artefact | What it demonstrates |
|---|---|
| [Functional specification](./functional-specification.md) | How I scope a feature, including the states most specs leave undefined |
| [User stories and acceptance criteria](./user-stories-and-acceptance-criteria.md) | Stories written so that done is not a matter of opinion |
| [API contract](./api-contract.md) | Request and response contracts specified rather than described |
| [Decision log](./decision-log.md) | How decisions get recorded so they are not relitigated every month |
| [UAT test plan](./uat-test-plan.md) | How I gate a release, from my QA background |
| [Weekly delivery update](./weekly-delivery-update.md) | The written cadence that keeps distributed delivery legible |

## How I work, in one paragraph

I write the specification before the roadmap, because a roadmap built on undefined states is a list of guesses with dates attached. I specify the failure cases explicitly, because those are where the product actually gets judged. I came up through QA, so I test the build myself before it reaches the client rather than discovering problems in their UAT. And I write the weekly update every week, not most weeks, because that is the only thing that makes remote delivery legible to the people paying for it.

---

Sylvia Mutheu · [sylviamutheu.com](https://sylviamutheu.com) · [LinkedIn](https://www.linkedin.com/in/sylviamutheu)
