Test harnesses, automated or otherwise, are a valuable tool for enforcing correctness constraints on a codebase.

TDD-as-practiced seems to lose sight of the fundamental goal of correctness-constraint.  If a programmer is capable of writing error-free code and knowing the full impact of their changes on an established code-base equally well/quickly without tests, any tests they write are *purely* for the other mere mortals who may happen to be interested in the code or a waste of time.  I've never met such a programmer, or anyone who has, so I'm willing to believe that they effectively don't exist and that most programmers are benefited by having the landing-lights of a well-written test harness.  

Tests shouldn't be written to keep a build light green (or inexplicably blue, in the case of Jenkins).  Tests shouldn't be written to make code-coverage metrics go up.  Tests should only be written to minimize the chances that human error results in avoidable computer error. 

"But what about the...": avoidable computer errors are lost value.  Hopefully it's monetary value (if your servers aren't net profitable, you have a problem), but that's another post.  If it's possible to invest X in preventing > X lost value, that's a valuable correctness constraint.

Unit tests are not correctness constraints.  Mocked tests are (probably) not correctness constraints.

Let's break that down a bit.  

### Unit tests are not correctness constraints

If I have a test that asserts that `foo(7)` = 14, I've made my test harness slower.  If I'm lucky, I've also avoided the dreaded "`foo(7)` causes error", but odds are that I've merely exchanged some CPU and human time for absolute certainty that `foo(7) == 14` (assuming it's a pure function).

Unfortunately for hypothetical me, `foo` never gets invoked with 7.  It's dead code, or all the inputs are floats (or strings, or...).  As a result, my test is just marginally accelerating the heat death of the universe and giving some number of people (false) confidence about quality and correctness.

In a more realistic bad case, `foo` is part of a pipeline and another entity doesn't handle some output correctly, but all tests pass because the pipeline is only ever tested in segments too short to include the interaction: eg, `[foo] -> [bar] -> [baz]`, with the pairwise interactions `[foo] -> [bar]` and `[bar] -> [baz]` under test.

Testing at the level of the end-user interface (eg, HTTP API) avoids most possible unit-level fragilities, possibly at the expense of test harness run-time or verbosity.

Aside: refactoring can be made more difficult by extensive unit tests.

### Mocked tests are (probably) not correctness constraints

First, a definition of terms: a mock is a stand-in for a real system.  That real system could be entirely under your own control (eg, `AbstractNightmareFactory`), partially under your control (eg, `postgresql`), or entirely out of your control (eg, `maps.googleapis.com`).

The danger with mocks in general is that they may not be realistic simulations of the real system. 

Mocking systems entirely under your own control makes sense in some circumstances.  I've used the technique as a hack to simulate concurrent requests in Rails apps, eg.  It is, unfortunately, deceptively easy to have divergent behavior between the mock and the real system: "implements interface X" is inadequate in the general case.  

For systems that are to some extent out of direct control, mocks are a gamble that the relevant behaviors are captured.  Weigh the costs and probability of being wrong carefully.

## Synthesis

Mock- and unit- based testing both make bold assumptions: "these are the only behaviors possible / worth testing".

I strongly prefer testing at the seams of a product (eg, inbound / outbound HTTP requests for a server) for the knowns and monitoring relevant environments to discover unknown unknowns. 

Fuzz-testing is an interesting option to explore for faster (or more thorough) unknown-unknown discovery, but it's hard to do well. 

Well-executed hybrid strategies likely have a better ROI than strict adherence to any one style of testing, with lower-level tests as executable documentation and high-level tests as correctness specs.

