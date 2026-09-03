# HW06 — AI Critique

**Student ID:** 23127531  
**Length:** approximately 250 words

AI was useful for accelerating API test design, especially when generating broad input partitions, authentication variants, response checks, and state-transition ideas. It also reduced repetitive work by producing Postman collections, Newman-oriented assertions, CI configuration, bug-report templates, and spreadsheet summaries. However, the strongest lesson from this homework is that AI-generated tests are only starting points, not trustworthy test oracles.

For the registration API, AI initially assumed common production rules such as valid email syntax, password constraints, and duplicate-email rejection. Those expectations were not supported by the supplied specification or implementation, so human review had to mark many cases INVALID or INCOMPLETE. A similar issue appeared in the cart API, where AI assumed nonexistent product IDs should return 404 and repeated additions should merge quantities. Source inspection showed that the implementation simply stores the submitted body in memory, making those expectations unjustified.

AI performed better when the requirement was explicit. For the Admin users API, the specification clearly required an Admin account, so the generated non-admin denial tests had a reliable oracle. Keeping those assertions strict exposed a genuine broken-access-control defect. Human review was still necessary to separate that defect from weaker assumptions about JWT revocation, lockout, stale roles, or exact response schemas.

Overall, AI improved speed, breadth, documentation consistency, and automation, but it also tended to import “best practice” expectations that were not actual requirements. The most valuable human contribution was validating every oracle against the specification and source, then distinguishing confirmed defects from robustness observations and characterization results.
