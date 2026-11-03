# Implementing a feature from a GitHub Issue

Give a GitHub issue to GitHub Copilot CLI, then steer the conversation to implement the feature.

GitHub Copilot CLI can read a GitHub issue, propose an implementation plan, and draft code changes that satisfy the requirements. Because the workflow is conversational, you can correct assumptions, point to relevant files, and refine the approach as GitHub Copilot CLI works.

## Example scenario

Imagine you work on a platform team that maintains an internal Node.js billing API that powers customer subscriptions. A product manager opens an issue requesting a new `GET /subscriptions/{id}/plan` endpoint that returns the plan attached to a given subscription, along with unit tests and an update to the OpenAPI specification. The issue includes the desired response shape, error cases for missing or cancelled subscriptions, and a link to the existing `/subscriptions/{id}` endpoint to follow as a pattern. You can hand the issue to GitHub Copilot CLI and have it scaffold the route, controller, and tests, then open a pull request.

## Example workflow

GitHub Copilot CLI can read the issue directly from GitHub, plan the work, write the code, run your tests, and open a pull request.

### Pull the issue into context

Start in the root of the repository and reference the issue by URL or `owner/repo#number`. GitHub Copilot CLI uses the GitHub MCP server to fetch the issue body, comments, and linked items, then keeps that context available for the rest of the session.

```text copy
Read octo-org/billing-api#482 and any issues or PRs it links to. 
Summarize what needs to be implemented before we plan the work.
```

### Draft an implementation plan with `/plan`

Switch to plan mode so GitHub Copilot CLI thinks through the change before touching any files.

```text copy
/plan Implement the subscription lookup endpoint described in the issue. 
Follow the same conventions as the existing GET /subscriptions/{id} endpoint.
```

> [!NOTE] The following response is an example. Copilot Chat responses are non-deterministic, so you may get a different response from the one shown here.

GitHub Copilot CLI returns a plan similar to:

```text
1. Add a `getPlanForSubscription` method in `src/services/subscriptionService.ts` that reuses the existing billing client, throws `NotFoundError` when the subscription or plan does not exist, and throws `CancelledSubscriptionError` when the subscription has been cancelled.
1. Add a `GET /subscriptions/:id/plan` route in `src/routes/subscriptions.ts`, mirroring the error handling in the sibling `GET /subscriptions/:id` route and mapping `CancelledSubscriptionError` to a `410` response.
1. Update `openapi/subscriptions.yaml` with the new path, response schema, and `404` and `410` responses.
1. Add unit tests in `test/services/subscriptionService.test.ts` and `test/routes/subscriptions.test.ts` covering the success case, missing subscription, cancelled subscription, and missing plan.
1. Run the test suite and update the changelog entry.
```

Review the plan, push back on anything that looks off, and ask GitHub Copilot CLI to revise it before you approve. For example, you might reply with `Use the existing error classes in src/errors—NotFoundError for missing subscriptions or plans and CancelledSubscriptionError for cancelled subscriptions rather than introducing new ones` and have it update the affected steps.

### Kick off the implementation and steer as needed

Approve the plan to start coding. GitHub Copilot CLI works through the steps one at a time, showing each edit before applying it. If it heads in the wrong direction, interrupt and redirect.

For example, suppose GitHub Copilot CLI adds a new database query in your `getPlanForSubscription` method instead of reusing the billing client your team standardized on. You can stop and steer it:

```text copy
Don't add a new query here. The billing client in 
src/clients/billingClient.ts already exposes a getPlan method. 
Use that and update the service to handle its NotFound response.
```

GitHub Copilot CLI revises the code and continues with the remaining plan steps.

### Generate and run unit tests

When GitHub Copilot CLI reaches the testing step, it scaffolds tests that match the patterns in your existing test files. For the plan endpoint, it might add cases like:

* Returns the plan record for an active subscription.
* Returns `404` when the subscription does not exist.
* Returns `410` when the subscription has been cancelled.
* Returns `404` when the subscription has no plan attached.

After writing the tests, GitHub Copilot CLI runs them in the terminal so you can see the results immediately.

```text copy
Run the test suite for the new endpoint and fix any failures.
```

If a test fails, GitHub Copilot CLI reads the failure output, updates the implementation, and re-runs until the test suite is green. Review each fix to make sure it's addressing the root cause rather than masking it.

### Review the changes with `/diff`

Use `/diff` to see a consolidated view of changes made across the session.

```text copy
/diff
```

If something looks wrong, ask GitHub Copilot CLI to revise it before you commit. For example, you might reply with `Revert the formatting changes in src/routes/subscriptions.ts to only keep the new route handler` to scope the diff back to the intended changes.

### Open a pull request

Once the feature is implemented, tested, and reviewed, ask GitHub Copilot CLI to open a pull request. It uses the GitHub MCP server to push the branch and create the pull request

```text copy
Commit the changes on a new branch, push it, and open a pull request 
against main. Link it to octo-org/billing-api#482 and summarize the 
implementation, the tests added, and any follow-up work.
```

GitHub Copilot CLI reports back with the pull request's URL so you can move it forward from there.

## Further reading

* [GitHub Copilot CLI](/en/copilot/how-tos/copilot-cli)# GitHub Docs <!-- omit in toc -->

Welcome to GitHub Docs! GitHub’s documentation is open source, meaning anyone from inside or outside the company can contribute. For full contributing guidelines, visit our [contributing guide](https://docs.github.com/en/contributing).


## Quick links by contributor type

* **Hubbers (GitHub employees):** See [CONTRIBUTING.md](https://github.com/github/docs-content/blob/main/CONTRIBUTING.md) in the `docs-content` repository for GitHub-specific processes.

* **Open source contributors:** See [CONTRIBUTING.md](https://github.com/github/docs/blob/main/.github/CONTRIBUTING.md) in the `docs` repository for a quick-start summary.

## How we sync changes across Docs repositories

There are two GitHub Docs repositories: 

- **`github/docs`** (public): Open to external contributions

- **`github/docs-internal`** (private): For GitHub employee contributions. 

The two repositories sync frequently. Content changes in one are reflected in the other.  Hubbers might prefer to post in `docs` when working with a customer, but `docs` has limitations on the types of contributions it accepts to safeguard the site and our workflows. Internal contributions should usually go to `docs-internal`.

**Important:** The `docs` repository accepts contributions to content files (`.md` files in `/content` and select `/data` sections like reusables only). Infrastructure files, workflows, and site-building code are not open for external modification.

## New to contributing

Here are some resources to help you get started with open source contributions:

* [Finding ways to contribute to open source on GitHub](https://docs.github.com/en/get-started/exploring-projects-on-github/finding-ways-to-contribute-to-open-source-on-github)
* [Set up Git](https://docs.github.com/en/get-started/git-basics/set-up-git)
* [GitHub flow](https://docs.github.com/en/get-started/using-github/github-flow)
* [Collaborating with pull requests](https://docs.github.com/en/github/collaborating-with-pull-requests)

## License

This project is dual-licensed under:

* **Creative Commons Attribution 4.0** - for documentation and content in the assets, content, and data folders (see [LICENSE](LICENSE))
* **MIT License** - for code (see [LICENSE-CODE](LICENSE-CODE))
