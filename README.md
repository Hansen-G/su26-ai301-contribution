# su26-ai301-contribution
# Contribution 1: Implementations should report when a gateway name is too big

**Contribution Number:** 1
**Student:** Hansen Guo
**Issue:** https://github.com/kubernetes-sigs/gateway-api/issues/4464
**Status:** Phase I — In Progress

---

## Why I Chose This Issue

I chose this issue because it is closely aligned with my career goal of moving toward MLOps, AI platform infrastructure, model monitoring, and cloud-native reliability. The issue is in `kubernetes-sigs/gateway-api`, which is part of the Kubernetes ecosystem and directly related to production infrastructure behavior, conformance testing, and user-facing diagnostics. Contributing to this project will help me build practical experience with cloud-native APIs, Kubernetes resource constraints, and reliability-focused open-source development.

My professional background is in production pricing systems, automated data workflows, and model review processes in a regulated energy exchange environment. In my current work, I maintain and improve production pricing models, support settlement price calculations, and work with Python-based infrastructure. This issue interests me because it involves a similar mindset: identifying a production edge case, improving how the system communicates failure or risk to users, and adding test coverage so implementations behave more consistently. Through this contribution, I hope to learn more about Kubernetes Gateway API design, Gateway conditions, conformance testing, and how large open-source infrastructure projects handle compatibility-sensitive changes.

---

## Understanding the Issue

### Problem Description

The Gateway API In-Cluster GEP describes a naming convention for generated child resources based on the Gateway name and related GatewayClass or controller information. However, Kubernetes resources and labels often have a 63-character name or label value limit. If the combined Gateway name and GatewayClass/controller name becomes too long, implementations that generate child resources may fail to provision those resources correctly.

The current issue is not simply that long names exist. The deeper problem is that users may not receive a clear, user-facing signal explaining why provisioning failed or why generated resources could not be created. Since not every implementation generates child resources in the same way, the project cannot safely add a hard validation rule that blocks long Gateway names without creating a breaking change. Instead, the project needs a better way to report this potential problem to users, likely through a Gateway condition and/or a conformance test.

### Expected Behavior

When the combined Gateway name and GatewayClass/controller-related name is too long and may violate Kubernetes resource naming constraints, the implementation should clearly signal the problem to the user.

A good expected behavior would be one or both of the following:

1. A Gateway condition is added to indicate that the generated child resource name may be too long.
2. A conformance test verifies that implementations report this condition or otherwise surface the problem consistently.

The goal is not necessarily to reject the Gateway object at creation time. The goal is to improve user experience and diagnosability when an implementation cannot provision required child resources because generated names exceed Kubernetes length limits.

### Current Behavior

Currently, a user may create a Gateway whose name, combined with the GatewayClass or controller name, results in generated child resource names that exceed Kubernetes length limits. Depending on the implementation, child resources such as Services, labels, or other generated resources may fail to be created.

The user may then see that the Gateway is not provisioned, but the reason may not be obvious. The failure may appear in lower-level child resource errors instead of being reported clearly on the Gateway itself.

### Affected Components

The affected components may include:

* Gateway API specification and condition definitions
* Gateway status conditions
* In-Cluster GEP behavior around generated resource naming
* Conformance tests for Gateway implementations
* Implementation behavior around generated child resources
* Kubernetes resource names, labels, and child object generation

Specific files and packages will be confirmed during codebase exploration. Likely areas to investigate include Gateway API condition definitions, conformance test utilities, Gateway status handling, and documentation around generated resources.

---

## Reproduction Process

### Environment Setup

I am setting up a local development environment for the `kubernetes-sigs/gateway-api` repository. My initial setup plan includes:

1. Forking the upstream repository.
2. Cloning my fork locally.
3. Installing Go and any required project tooling.
4. Running the existing test suite or the relevant conformance test subset.
5. Exploring existing Gateway condition definitions and conformance test patterns.
6. Creating a local branch for this issue.

Potential setup challenges I expect include understanding the repository structure, identifying the correct conformance test location, and learning the project’s preferred test workflow. I will document any environment setup problems and solutions as I work through them.

### Steps to Reproduce

Initial reproduction plan:

1. Create or identify a Gateway with a long metadata name.
2. Pair it with a GatewayClass or controller-related name such that the combined generated child resource name would exceed the 63-character Kubernetes naming limit.
3. Observe whether the implementation reports a clear Gateway condition or whether the failure only appears indirectly through child resource provisioning errors.

Because this is a specification and conformance issue, reproduction may involve reading the current conformance tests and adding a new test case rather than reproducing against one specific implementation.

### Reproduction Evidence

* **Commit showing reproduction:** TBD — link to commit in my fork after I add a failing or illustrative test case.
* **Screenshots/logs:** TBD — logs or test output will be added if applicable.
* **My findings:** TBD — initial investigation will focus on whether existing conditions or tests already cover resource-name length failures.

---

## Solution Approach

### Analysis

The root cause appears to be a mismatch between the naming convention for generated resources and Kubernetes resource name or label constraints. The Gateway API In-Cluster GEP recommends names derived from Gateway and GatewayClass/controller information. However, generated child resources may have strict name length limits. When the combined names exceed those limits, some implementations may fail to create child resources.

A hard validation rule, such as a Validating Admission Policy that blocks Gateway creation when name length exceeds a threshold, may not be appropriate because not all implementations generate child resources in the same way. Blocking object creation could become a breaking change for implementations that are not affected.

Therefore, the safer solution is likely to improve reporting behavior through a Gateway condition and/or add a conformance test that verifies implementations provide a clear signal when this condition occurs.

### Proposed Solution

My initial proposed solution is to investigate and implement the smallest compatible change that improves user-facing diagnostics without introducing a breaking validation rule.

The likely approach is:

1. Identify the existing Gateway condition model and determine whether an existing condition reason can represent this case.
2. If no suitable condition exists, propose a new condition reason or condition behavior to report name-length-related provisioning risk.
3. Add or update conformance test coverage to ensure implementations surface this issue clearly.
4. Update documentation if needed to explain the expected behavior.

I will first confirm the preferred approach with maintainers before implementing a larger change.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:**
The problem is that Gateway implementations may generate child resource names by combining a Gateway name with a GatewayClass or controller name. If the resulting generated name exceeds Kubernetes length limits, child resources may fail to be created. Users need a clear Gateway-level signal explaining this risk or failure.

**Match:**
I will search the codebase for similar patterns involving:

* Gateway status conditions
* Condition reasons used for invalid or unsupported configuration
* Existing conformance tests that check Gateway conditions
* Existing tests around resource provisioning failures
* Existing patterns for compatibility-sensitive specification changes

**Plan:**

1. Review the issue discussion and the In-Cluster GEP section on automated deployments.
2. Explore the Gateway API condition definitions and conformance test structure.
3. Identify whether an existing condition can be reused or whether a new condition reason is needed.
4. Draft a minimal design note in the issue thread if the implementation path is ambiguous.
5. Add a test case that creates a Gateway and GatewayClass/controller-name combination likely to exceed the 63-character limit.
6. Verify that the expected condition or diagnostic behavior is asserted.
7. Update documentation or specification text if required.
8. Run the relevant tests locally.
9. Submit a PR with a focused explanation and ask for maintainer feedback.

**Implement:**
Branch and commits:

* Branch: TBD
* Initial investigation commit: TBD
* Test implementation commit: TBD
* Final implementation commit: TBD

**Review:**
Self-review checklist:

* [ ] The change follows Gateway API contribution guidelines.
* [ ] The change avoids introducing a breaking validation rule.
* [ ] The condition or test behavior matches the issue’s intent.
* [ ] The test is minimal and focused.
* [ ] The implementation is compatible with implementations that do not generate child resources.
* [ ] Documentation or comments explain the 63-character resource-name constraint clearly.
* [ ] Relevant tests pass locally.

**Evaluate:**
I will verify the solution by running the relevant unit and/or conformance tests. If I add a new conformance test, I will first confirm that it fails or is absent under the current behavior, then confirm it passes after the implementation change. I will also manually review the generated condition or test output to ensure that the user-facing message is clear.

---

## Testing Strategy

### Unit Tests

* [ ] Test case 1: Gateway name and GatewayClass/controller name combination is within the safe length limit.
* [ ] Test case 2: Gateway name and GatewayClass/controller name combination exceeds the 63-character generated-resource limit.
* [ ] Test case 3: Existing Gateway condition behavior remains unchanged for unrelated provisioning failures.

### Integration Tests

* [ ] Integration scenario 1: A Gateway with a long generated child-resource name receives a clear condition or diagnostic signal.
* [ ] Integration scenario 2: A Gateway with valid name length continues to behave normally and does not receive the new warning/error condition.

### Manual Testing

Manual testing will include creating test Gateway and GatewayClass objects with intentionally long names and observing whether the implementation reports the expected condition. If the repository’s conformance framework is the primary way to validate this behavior, manual testing may be limited to running the relevant test suite and inspecting test output.

---

## Implementation Notes

### Week 1 Progress

Planned tasks:

* Fork and clone `kubernetes-sigs/gateway-api`.
* Set up local development environment.
* Read the contribution guide.
* Read the In-Cluster GEP section about generated resource naming.
* Locate Gateway condition definitions.
* Locate conformance tests involving Gateway status or provisioning behavior.
* Comment on the issue to confirm the intended implementation direction.

Challenges expected:

* Understanding the repository structure.
* Determining whether the solution belongs in specification text, condition definitions, conformance tests, or a combination of these.
* Avoiding a breaking validation rule while still improving user experience.

### Week 2 Progress

Planned tasks:

* Implement a minimal failing or illustrative test case.
* Confirm whether an existing condition reason can be reused.
* Draft the first implementation branch.
* Run the relevant tests locally.
* Document findings and update this README.

### Code Changes

* **Files modified:** TBD
* **Key commits:** TBD
* **Approach decisions:** TBD

Expected approach decisions:

* Whether to add a new Gateway condition or reuse an existing one.
* Whether the initial PR should focus on conformance test coverage, specification language, or both.
* How to phrase the user-facing condition message so it is clear without over-prescribing implementation behavior.

---

## Pull Request

**PR Link:** TBD

**PR Description:** Draft:

This PR improves diagnostics for Gateway implementations when generated child resource names may exceed Kubernetes resource name or label length constraints. The issue can occur when the Gateway name and GatewayClass/controller-related name combine into a generated name longer than the 63-character Kubernetes limit.

The PR adds or updates test coverage and/or Gateway condition behavior so users receive a clearer signal when name length may prevent child resources from being provisioned. The change avoids adding a hard admission-time restriction, since that could be breaking for implementations that do not generate affected child resources.

**Maintainer Feedback:**

* TBD: Awaiting maintainer guidance.
* TBD: Feedback will be summarized here after issue or PR discussion.

**Status:** Awaiting implementation and review.

---

## Learnings & Reflections

### Technical Skills Gained

Expected technical skills:

* Kubernetes Gateway API concepts
* Gateway status conditions
* Kubernetes resource naming constraints
* Conformance test design
* Cloud-native API compatibility considerations
* Open-source contribution workflow
* Reading and modifying a large infrastructure codebase

### Challenges Overcome

Expected challenges:

* Understanding where the issue belongs: specification, condition behavior, conformance test, or documentation.
* Designing a non-breaking solution.
* Learning the Gateway API test structure.
* Communicating clearly with maintainers before making assumptions.

### What I'd Do Differently Next Time

TBD after implementation. I expect to reflect on whether I should have asked maintainers for guidance earlier, whether I scoped the first PR small enough, and whether my test design was sufficiently aligned with existing project patterns.

---

## Resources Used

* GitHub issue: https://github.com/kubernetes-sigs/gateway-api/issues/4464
* Gateway API repository: https://github.com/kubernetes-sigs/gateway-api
* In-Cluster GEP 1762: https://gateway-api.sigs.k8s.io/geps/gep-1762/
* Gateway API documentation: https://gateway-api.sigs.k8s.io/
* Kubernetes object names and IDs documentation: https://kubernetes.io/docs/concepts/overview/working-with-objects/names/
* Kubernetes labels and selectors documentation: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
* Gateway API contribution guide: TBD after reviewing repository documentation
* Relevant conformance test documentation: TBD after codebase exploration
