---
title: How We Reclaimed 40% of Our Engineering Time by Fixing Flaky Tests
excerpt: When I joined HubSpot as an engineering leader, I discovered our team was hemorrhaging productivity to unreliable acceptance tests. False positives were blocking releases, engineers were losing trust in our test suite, and debugging flaky tests was consuming hours of valuable development time each week.
publishDate: 'Jul 5 2022'
tags:
  - Guide
  - Developer Productivity
seo:
  image:
    src: '/peace.jpg'
    alt: Peace Street Sign
---

![Peace Street Sign](/peace.jpg)
When I joined HubSpot as an engineering lead, I discovered our teams were hemorrhaging productivity to unreliable acceptance tests. False positives were blocking releases, engineers were losing trust in our test suite, and debugging flaky tests was consuming hours of valuable development time each week.

Sound familiar? If your engineering teams are battling similar issues, this systematic approach can help you regain control and build a test suite your organization can trust.

## The Hidden Cost of Flaky Tests

Before diving into solutions, it's worth quantifying the problem. Flaky acceptance tests don't just slow down individual developers—they create cascading delays across your entire delivery pipeline. When tests fail unpredictably, teams lose confidence in automation, resort to manual verification, and release cycles stretch from days to weeks.

## Step 1: Data-Driven Diagnosis

The first step is understanding what's actually breaking. I manually categorized every acceptance test failure from the previous five days into seven distinct buckets:

1. **Broken Application Code** - Issues in our application logic
2. **Broken Test Code** - Problems with the test implementation itself  
3. **Broken Internal API** - Failures in APIs owned by other internal teams
4. **Broken Third-Party API** - External service disruptions
5. **Test Infrastructure Issues** - Environment instability, timeouts, resource constraints
6. **Unannounced Content Changes** - Marketing copy, pricing, or legal terms updated without notice
7. **Unannounced API Changes** - Schema or behavior changes without proper communication

**The results were eye-opening.** Of 30 failed tests, only 4 were caused by changes our team had made. The majority stemmed from process misalignments and inadequate change communication across the organization.

This data became crucial for two reasons: it helped us prioritize our internal improvements, and it provided concrete evidence for discussions about improving organizational processes.

## Solution 1: Strategic Test Migration

**Problem:** Many acceptance tests were validating UI components that could be tested more reliably at the unit level.

**Our context:** We were working on a checkout team with a cart component that had numerous variations depending on product combinations. Many of our acceptance tests simply opened the checkout UI to verify that cart values displayed correctly.

**Solution:** We restructured our testing approach into three focused layers:
- **Unit tests** for the cart component using mock data to cover all product combination scenarios
- **Snapshot testing** for API responses to catch payload changes
- **Type contracts** between frontend and backend to prevent integration issues

**Impact:** This dramatically reduced our acceptance test count while maintaining comprehensive coverage. The cart component alone went from 15 acceptance tests to 3, with better coverage through targeted unit tests. Fewer tests meant fewer infrastructure-related failures and faster feedback loops.

## Solution 2: Type Safety Across Service Boundaries

**Problem:** API payload changes across teams sometimes occurred without adequate coordination, leading to integration failures.

**Solution:** We implemented a centralized type repository that both frontend and backend teams consumed. The build pipeline would fail if any consumer project couldn't compile against the latest types.

**Trade-off consideration:** This approach did slow down development velocity slightly, as teams needed better coordination around type changes. However, the reliability gains far outweighed the minor speed reduction. As one senior engineer put it: "It's better to be deliberately slow than accidentally broken."

## Solution 3: Intelligent Failure Attribution

**Problem:** When tests failed due to external dependencies, engineers spent time investigating issues outside their direct control.

**Solution:** We enhanced our test logging to automatically surface relevant Slack channels for any failing APIs. Each internal API repository contained a configuration file mapping to its team's status channel, which our test runner would display in failure reports.

**Impact:** This enhancement cut debugging time significantly and gave developers confidence that when external services experienced issues, the responsible teams had visibility and were addressing them.

## Solution 4: Proactive Content Change Management

**Problem:** Content updates from various teams occurred without coordinated notification, causing unexpected test failures.

**Solution:** We implemented a two-part system:
- **Slack notifications** whenever content teams updated relevant sections
- **Automated snapshot updates** that could refresh all affected test files with a single command

This eliminated the manual overhead of tracking down hardcoded values across multiple test files whenever content changed and improved coordination across teams.

## Measuring Success

Three months after implementing these changes, we saw:
- **67% reduction** in false positive test failures
- **40% faster** average debugging time for legitimate failures  
- **50% fewer** release delays due to test-related blockers

More importantly, developer confidence in our test suite was restored, and teams began viewing acceptance tests as valuable guardrails rather than obstacles.

## Implementation Recommendations for Engineering Leaders

**Start with data collection.** Spend one week categorizing every test failure. You can't optimize what you don't measure, and the patterns will likely surprise you.

**Focus on internal improvements first.** While it's tempting to immediately tackle cross-team coordination issues, establishing reliable practices within your own team creates a foundation for broader organizational changes.

**Treat test reliability as a product.** Assign ownership, track metrics, and invest in tooling improvements the same way you would for customer-facing features.

**Communicate impact constructively.** When discussing findings with other teams, focus on business impact (release delays, engineering hours) and frame conversations around process improvements rather than blame.

## The Long-Term Perspective

Improving acceptance test reliability isn't a sprint—it's a foundational investment that pays dividends across your entire engineering organization. By systematically addressing both technical and process issues, you can transform your test suite from a source of frustration into a genuine competitive advantage.

The key insight is that most "flaky test" problems aren't actually testing problems—they're coordination and process alignment challenges that manifest as technical issues. Address the underlying organizational patterns, and the technical symptoms often resolve themselves.

---

*Have you implemented similar solutions in your organization? I'd love to hear about your experiences and any additional strategies that have worked for your teams.*