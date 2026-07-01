---
layout: post
title: "Problem Solving — S. Ian Robertson"
date: 2025-11-16
description: Notes on Robertson's cognitive framework for problem solving — representation, heuristics, insight, schemas, and metacognition.
tags: [cognitive-science, problem-solving, learning, books]
---

## Summary

A problem is a gap between where you are and where you want to be — with no obvious way to close it. The "where you want to be" is always an imagined state, something you'd like to be true that isn't yet. Or more bluntly: problem solving is "what you do, when you don't know what to do" (Wheatley, 1984). The moment you _do_ know what to do, the problem has dissolved into a task — a locked door stops being a problem the instant you notice the key is already in your pocket.

Problem solving has been studied from several distinct research traditions, each illuminating a different piece. **Behaviourists** (Thorndike, Watson, Skinner) explained problem solving as trial-and-error learning shaped by reinforcement — useful for describing how organisms adapt, but silent on the internal structures that make complex reasoning possible. **Gestalt psychologists** (Wertheimer, Duncker, Koffka) shifted focus to perception and restructuring: insight — the sudden "Aha!" — wasn't random but arose from a reorganization of how the problem was seen. **Cognitive psychologists** (Newell, Simon, Anderson) formalized this into the problem space framework: a problem is defined by an initial state, a goal state, and operators that move between them; solving it is search through that space. **Educationalists** (Polya, Schoenfeld) brought the question down to practice — what heuristics and metacognitive habits actually help people solve problems they've never seen before?

Robertson's book draws on all four traditions. The cognitive framework is central, but the Gestalt insight into representation and the educationalist emphasis on deliberate strategy both run through the treatment.

**Problem taxonomy** — what kind of problem you have determines which approaches apply:

| Axis                  | Poles                                                                                                             |
| --------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Knowledge requirement | **Knowledge-lean** — domain-general strategies suffice (a river-crossing puzzle) ↔ **Knowledge-rich** — needs domain-specific expertise (diagnosing a failing Kubernetes pod)  |
| Goal specification    | **Well-defined** — goal and constraints fully specified (chess: checkmate is the exact target; finding the moves is the work) ↔ **Ill-defined** — goal or constraints vague ("make the onboarding flow feel better")        |
| Solver experience     | **Semantically lean** — little relevant experience (your first-ever segfault) ↔ **Semantically rich** — deep experience of this type (an SRE who's debugged this outage many times) |
| Solution character    | **Standard** — deliberate step-by-step search (long division) ↔ **Insight** — needs a sudden restructuring (the nine-dot puzzle)      |

What makes that problem classification fast — sorting a new problem into one of these types — is *schemas*: organized representations of problem types in long-term memory that let you recognize "this is a problem of type X" and pull up its approach instead of reasoning from scratch. Deliberately building better schemas is, in the end, what getting good at problem solving means — and most of what follows is about how.

## Key concepts

### Representation determines what's reachable

Newell and Simon's foundational work frames every problem as a **problem space**: an initial state, a goal state, a set of operators that move between states, and a search process that navigates the space. This pins down what "representation" means. It's the structure that fixes which states exist, which operators count as legal, and which solutions can be reached at all — not a loose matter of how you picture the problem. Omit a relevant variable, or encode a constraint incorrectly, and the correct solution drops out of the search space altogether; no amount of clever searching brings it back.

The **Hobbits and Orcs** problem makes this concrete. Three hobbits and three orcs must cross a river using a boat that holds at most two; orcs can never outnumber hobbits on either bank. Greeno (1974) and Thomas (1974) both studied how solvers approach this. Many people represent the state informally — a vague mental image of who is where — and quickly run into what feel like dead ends or illegal moves. The logic is trivial. The representation is where people get stuck. A precise tuple `(hobbits_left, orcs_left, boat_side)` makes every legal operator explicit: enumerate the moves that keep each component valid, and the solution path becomes a straightforward search. Solvers who represented the state space this way made far fewer errors and reached the solution faster. Same puzzle; all that changed was how they wrote it down.

The **nine-dot problem** is a starker example. Connect nine dots arranged in a 3×3 grid using four straight lines without lifting the pen. Most people fail because they self-impose a square boundary around the dots — a constraint the problem statement never stated. The search space they're exploring doesn't contain the solution. The boundary was never in the problem. They added it themselves, in the representation they built while reading.

This last point is what makes representation especially tricky. Bransford, Barclay, and Franks (1972) showed that people don't encode problem statements neutrally — they construct an interpretation, and that interpretation can include inferences and assumptions the statement never made. You read "nine dots in a grid" and your encoding includes an imaginary boundary. You read "the server is timing out" and your encoding includes an implicit assumption about which server. The construction happens automatically; catching it requires deliberate effort.

A practical version: if you represent a failing Kubernetes pod as "the image is broken," your search focuses on the registry and build pipeline. Represent it as "the pod can't reach a ready state" and you open the search to readiness probes, resource limits, scheduling constraints, and network policy. Which operators are even available depends on the representation, so it's worth settling the frame before doing anything else.

### Heuristics are useful but bounded

If representation fixes the search space, heuristics are how you move through it. They're mental shortcuts that make search tractable — without them, the space of possible moves in almost any real problem is too large to explore exhaustively. Robertson focuses on three: hill climbing (always take the step that moves closest to the goal), means-end analysis (identify the gap between current state and goal state, then select an operator that reduces it), and working backward (start from the goal and ask what would need to be true immediately before it).

Each is genuinely useful, and each has a characteristic failure mode. Hill climbing fails when you have to move _away_ from the goal to ultimately reach it — a maze with dead ends is the simple version, but software dependency resolution is a real one where installing the right package requires temporarily removing a conflicting one. Means-end analysis can get locked onto a subgoal that isn't actually necessary, spending effort closing a gap the solution doesn't require; a classic example is a student who rewrites a proof step to get closer to a known lemma, not realizing the proof doesn't need that lemma at all. Working backward is powerful for debugging (trace from the error back through the call stack to the source) but doesn't generalize well to open-ended design problems where the goal state is fuzzy.

The Gestalt tradition identified two mechanisms that explain why heuristics break down even when the solver has all the information they need.

**Functional fixedness** (Duncker 1945): solvers encode objects in their conventional role and fail to see them as candidates for other uses. The thumbtack box in the candle problem never enters the search as a shelf because the initial encoding locked it out as a container. The mistake lives in the encoding, not the reasoning. A real-world version: a developer who can't see a config file as a substitute for a database entry because "config files don't work that way" is imposing a constraint that exists only in the encoding, not in the problem.

**Set effects / Einstellung** (Luchins 1942): solving a class of problems with a particular method creates a persistent bias toward that method even when simpler alternatives exist. Luchins's water jug problems show this directly: participants who learned a complex filling procedure applied it to problems with a trivial direct solution, often without noticing the shortcut was available. The set operates below awareness, biasing which operators the solver considers at all.

Wertheimer's distinction between **reproductive thinking** (applying learned procedures regardless of fit) and **productive thinking** (reasoning from the actual structure of the current problem) frames both: functional fixedness and set effects are both forms of structural blindness. That's why strategy rotation helps: deliberately switching methods breaks the implicit exclusions that set and fixedness quietly impose.

### Insight requires leaving the current search space

Strategy rotation fights those exclusions from inside a representation. Insight is the move for when that isn't enough — when the representation itself is what's wrong. The sudden shift that makes a previously intractable problem feel obvious isn't a random gift. Kaplan and Simon (1990) give the reframe worth keeping: conventional problem solving is searching **in** a representation, navigating the problem space you already have. Insight is searching **for** a representation — finding a better problem space first. It's still search, just search over possible framings instead of over the moves within one framing. When you're stuck, you aren't following the wrong path — you're on the wrong map.

Ohlsson's **Representational Change Theory (RCT)** explains why impasse happens and how the brain escapes it. The initial representation is assembled from salient features and whatever long-term memory retrieves first. If the solution isn't reachable from that representation, you hit impasse — forward progress stops regardless of effort. Four escape mechanisms:

- **Re-encoding**: attention shifts to different features; activation is subtracted from elements that produced dead ends and redistributed across others
- **Constraint relaxation**: drop constraints inadvertently imposed — the nine-dot imaginary boundary, the tack box as container-only
- **Elaboration**: add information that wasn't in the initial encoding
- **Chunk decomposition**: break elements into sub-elements, exposing structure hidden at the chunk level (matchstick arithmetic is the canonical case)

These are largely unconscious — they happen through spreading activation and cued retrieval, not deliberate reasoning.

MacGregor et al.'s **Criterion for Satisfactory Progress Theory (CSPT)** adds the conscious side. Solvers use hill climbing while monitoring progress. When expected progress isn't achieved — criterion failure — they look for other states to expand. Working memory capacity matters: solvers who can look further ahead before hitting criterion failure are more likely to recognize that a solution requires an initially unpromising move. In the nine-dot problem, people who could anticipate further consequences of each line were more likely to discover that lines must extend outside the imaginary boundary.

RCT and CSPT cover different halves of the same process — the unconscious and the conscious. Gilhooly and Murphy (2005) found evidence for both unconscious (Type 1) and conscious (Type 2) processes in insight, with the balance depending on problem type. Perceptual and spatial insight problems draw more on unconscious chunk decomposition; verbal and logical ones involve more conscious monitoring and search. Incubation — stepping away from a stuck problem — works because it allows unconscious re-encoding to run without conscious hill climbing suppressing the process by reinforcing the failed frame.

Norman Maier's **two-string problem** demonstrates the unconscious mechanism directly: participants must tie together two strings that are too far apart to reach simultaneously. Most keep trying variations on walking between them. The solution requires reframing one string as a pendulum. Solvers who were casually nudged toward pendulum-like motion by the experimenter solved it faster without realizing why — the nudge triggered re-encoding below awareness. A software equivalent: the developer who's spent an hour convinced the bug is a logic error in a function, takes a five-minute walk, and immediately wonders whether it's an encoding issue upstream — and it is. No new information arrived; the pieces just settled into a different representation.

### Expertise is schema density, not raw speed

Insight is the fallback for when you don't already have the right representation; expertise is having it in advance. Expert performance shows three characteristic signatures. First, **fast categorization**: experts classify problems by deep structure, not surface features. Chi, Feltovich, and Glaser (1981) found that novice physics students grouped problems by surface appearance ("this is an inclined plane problem") while experts grouped them by underlying principle ("this is a conservation of energy problem") — a fundamentally different representation that immediately implies a solution strategy. Second, **perceptual chunking**: experts encode larger meaningful configurations as single units, letting them process more information per cognitive step. Third, **long-term working memory**: Ericsson and Kintsch (1995) documented that experts develop domain-specific encoding strategies that effectively extend working memory into long-term memory — they can retrieve and manipulate information that novices would need to hold in limited-capacity short-term memory.

The chess evidence established all three. De Groot (1965) first showed that grandmasters didn't examine more moves than masters — they examined better moves, because they recognized meaningful board configurations immediately. Chase and Simon (1973) made the mechanism explicit: expert players reconstructed realistic mid-game positions from five-second exposures; when pieces were placed randomly, the advantage disappeared entirely. The skill was in pattern recognition applied to meaningful chess structure, not memory capacity or processing speed. The same signatures appear in radiology (Lesgold et al. 1988: experts see clinically meaningful regions; novices see individual features), programming (McKeithen et al. 1981: expert programmers chunk code into functional units), and physics problem solving.

**How expertise develops.** Three models from the chapter each illuminate a different dimension. Fitts and Posner (1967) describe three stages: a _cognitive_ stage (resource-intensive, reliant on declarative knowledge, slow and error-prone), an _associative_ stage (rules are compiled, performance becomes more automatic), and an _autonomous_ stage (highly proficient, no longer reliant on conscious control). Dreyfus (1997) extends this to five stages — novice, advanced beginner, competent, proficient, expert — and makes a point often missed: rule-based systems cannot account for expert intuition. Experts don't follow rules; they perceive situations holistically and respond based on pattern recognition accumulated over thousands of exposures. This is why experts are often poor at explaining what they do. Glaser (1996) focuses on the changing locus of control: from _external support_ (dependent on teachers and coaches), through _transitional_ (support gradually withdrawn), to _self-regulatory_ (learning fully under the learner's own direction).

The **Power Law of Practice** describes the shape of skill acquisition: rapid improvement early, dramatically diminishing returns as practice accumulates (Newell & Rosenbloom 1981). This is consistent across domains — it's a structural feature of skill acquisition, not a motivation problem. It explains why early study of a domain has very high return and why pushing past intermediate competence requires disproportionate effort.

**Deliberate practice** (Ericsson, Krampe & Tesch-Römer 1993) is what determines how fast you move through those stages. Not all practice is equal. Deliberate practice is effortful, aimed at the edges of current ability, with immediate and accurate feedback, focused on specific components that need improvement. Comfortable practice within well-established skill produces much less improvement than shorter periods of targeted, difficult work at the boundary of competence. Ericsson and Charness (1994) extend this: expertise involves continuous effortful adaptation to domain demands, not consolidation of learned routines.

**The intermediate effect.** Boshuizen and Schmidt (1990, 1992) documented a striking pattern in medical expertise: people with intermediate knowledge can _outperform_ both novices and experts on some tasks, and be _outperformed by novices_ on others. Intermediates encode too much detail — they "process too much garbage" — and so recall more clinical propositions than experts, but make worse diagnostic decisions because the detail obscures the pattern. Experts have compiled their knowledge into higher-level scripts that suppress irrelevant detail. The implication: accumulating more knowledge at the same representational level isn't always progress; what matters is whether knowledge is compiled into higher-level schemas.

**Domain specificity.** Expertise doesn't transfer across domains. Content knowledge is domain-specific (deep physics knowledge doesn't help with medical diagnosis). Task knowledge — procedural and strategic knowledge — transfers only to closely related domains that share a genuine skill overlap. There is no general expert reasoner; there are domain experts, and their domains may or may not share transferable skills.

**The automaticity paradox.** Automatisation ought to make expert performance brittle — if procedures run without conscious access, how do experts handle novel problems? Hatano and Inagaki (1986) resolve this with the distinction between _routine expertise_ (fast, efficient, handles typical problems with compiled procedures) and _adaptive expertise_ (flexible, handles novel and exceptional cases through schemas that explicitly represent exceptions and edge cases). Expert schemas hold more than the common cases; they include strategies for when the common case doesn't apply. This is what distinguishes a senior engineer who can debug an unfamiliar system from one who can only optimize systems they built.

**Downsides of expertise.** Two documented failure modes. Ottati et al. (2015) demonstrated the _earned dogmatism effect_: people who perceive themselves as expert become more closed-minded — expertise feels like a license to dismiss alternatives without engaging with them. Fisher and Keil (2015) documented the _illusion of explanatory depth_: experts may have forgotten how much they've forgotten; they report more explanatory competence than they actually possess when pressed to generate detailed mechanistic explanations. In both cases the expertise is real; what's broken is the person's sense of where it runs out.

### Memory reconstructs, it doesn't replay

Expertise rests on schemas held in long-term memory, which makes how memory actually works load-bearing — and it doesn't work the way it feels like it does. A common mental model of memory is a recording — you store an experience, and later you play it back. The cognitive science is very different: memory is reconstructive. When you remember something, the brain assembles the recollection from fragments, filling gaps with inference, prior expectations, and current context. This makes memory fast and flexible but also systematically unreliable in specific ways.

Elizabeth Loftus's research on eyewitness testimony is the most studied example. Her experiments showed that the phrasing of a post-event question — "How fast were the cars going when they _smashed_ into each other?" vs. "contacted" — significantly changed what participants later remembered, including whether they recalled seeing broken glass that wasn't there. What came back was rebuilt around the suggestion the question had planted. A more technical analogue: write a complex module, set it aside for six months, then need to modify it. You'll feel confident about how it works — and be wrong about specific details in ways that cause bugs. The confidence is real, but it tracks how fluent the reconstruction feels, not how accurate it is.

The consequence for schema building: a schema that gets reinforced through repeated reconstruction of a slightly wrong version becomes a well-practiced error. Schema quality — whether the pattern you've chunked is actually correct — matters as much as schema quantity.

Because recall is reconstruction, encoding strategies like elaboration and dual coding work by laying down richer cues to rebuild from; there's no cleaner recording to be had. And reconstruction has a second consequence that reaches past memory into problem solving itself: what you can later retrieve is governed by surface resemblance, not structure. That is where transfer succeeds or fails.

### Transfer requires structural similarity

The hope that becoming good at hard problems in one domain makes you better at hard problems in general is mostly false. Transfer of skill happens when two problems share underlying abstract structure — not surface features, not domain vocabulary, not difficulty level.

**Transfer types.** Transfer can be positive (prior experience helps), negative (prior experience interferes — what the Einstellung effect produces when a learned procedure is applied where it no longer fits), near (similar domain, slightly different problem), or far (different domain, same underlying structure). Far positive transfer is what people usually mean when they talk about generalizable problem-solving skill, and it's also the hardest to reliably produce.

Problems with identical underlying structure but different surface content are called **isomorphs**. Simon and Hayes (1976) demonstrated that isomorphic problems can vary dramatically in difficulty: two river-crossing problems with identical state-space structure differ in solution rate because the surface framing changes which operators feel natural to try. **Homomorphs** are structurally similar but not identical — the same template with slight variations in constraints. Both transfer when solvers notice the structural match; neither transfers reliably when they don't.

**Three kinds of similarity** determine whether transfer occurs. _Surface similarity_ means the objects or perceptual features of the two problems resemble each other. _Relational similarity_ (also called structural similarity) means the relations between objects are the same — the underlying structure maps. _Procedural similarity_ means the same solution method applies even if surface and structure differ. Surface similarity drives what gets _retrieved_ from memory; relational similarity determines whether the analogue is actually _useful_. These two properties — _retrievability_ and _inferential soundness_ — come apart, and the gap between them is where transfer usually fails: Gentner, Rattermann, and Forbus (1993) found people spontaneously retrieve surface-similar analogues even when a structurally similar one is what would actually solve the problem. You can retrieve a surface match that gives you nothing, or miss a structurally identical analogue because it looks nothing like the current problem.

**Successful analogical transfer requires three things in sequence**: retrieval of a relevant analogue from long-term memory, mapping of corresponding roles across source and target (which objects in the source play the same role as which objects in the target), and adaptation of the source solution to the target context. Failure at any step blocks transfer. Most naturally occurring transfer failure is a retrieval failure — the right analogue is in memory but wasn't retrieved because it doesn't look like the current problem on the surface.

Gick and Holyoak's (1983) radiation/fortress study is the canonical demonstration. A doctor must destroy a tumor using radiation, but a strong-enough ray damages surrounding tissue; the solution is multiple weak rays converging from different directions. Most participants fail. Participants who had previously read a story about a general who converged small army groups along separate roads to capture a fortress solved it at much higher rates — the underlying structure is identical. But participants who read the story without being prompted to use it as an analogy transferred poorly anyway. The structural mapping existed; retrieval didn't surface it without explicit prompting.

Retrieval isn't hopeless, though. Chen, Mo, and Honomichl (2004) showed that even analogues learned long ago surface reliably when their structural features were encoded strongly at learning time. The practical lever is the label you file a solution under: not "I solved a rate problem" but "I solved a convergence problem" — a structural tag that primes retrieval when the next convergence problem appears, whatever domain it's dressed in.

**Gentner's Structure Mapping Theory (SMT)** formalizes what makes an analogy work. The central claim: analogy is a mapping of relational structure from a source to a target, independent of the objects involved. What gets mapped isn't attributes of objects ("both involve radiation" / "both involve armies") but the _relations between objects_ — the structural skeleton. The **principle of systematicity** says that mappings which preserve higher-order relations (relations between relations) are preferred over mappings that only match first-order features. "The atom is like a solar system" works because the relational structure maps: electrons orbit the nucleus as planets orbit the sun, and the force-governs-orbit relationship holds in both — even though the objects share no surface features. A mapping based only on "both are round" would score low on systematicity and license few useful inferences.

Gentner and Gentner (1983) showed that the choice of source analogy materially affects what inferences people draw. Participants given a "flowing water" analogy for electricity reasoned well about resistance; those given a "teeming crowds" analogy reasoned better about batteries. The metaphor did real work: it decided which structural inferences people drew.

**Metaphors are compressed analogies.** Novel metaphors require active structural comparison; as they become conventional, they become category assertions and lose their live analogical force. Bowdle and Gentner (2005) call this the "career of metaphor": "electricity flows" was once a live structural analogy, now it's nearly literal. The freshest analogies are often most useful, before the structural mapping has been compressed into a label that no longer prompts active reasoning.

In practice: the habit worth building is asking "what underlying pattern does this resemble?" — not "have I seen something like this before?" A DNS caching bug and a memory interference problem look nothing alike on the surface but share the same structure of stale state contaminating fresh queries. Recognizing that is transfer. Recognizing that "both involve the network" is not.

### Metacognition is the compounding lever

Metacognition is cognition about cognition — monitoring and regulating your own thinking processes as you work. It's distinct from domain knowledge and distinct from raw ability. Two people with equal skill can have very different outcomes if one actively tracks what strategy they're using, whether it's working, and when to switch, while the other just pushes forward.

Alan Schoenfeld's research on mathematical problem solving makes this concrete. He compared novice and expert mathematicians working through unfamiliar problems in think-aloud protocols. Novices almost never stopped to evaluate whether what they were doing was working — they'd invest ten or fifteen minutes in a fruitless approach without questioning the approach itself. Expert mathematicians regularly paused: "Is this going anywhere? What else could I try? Have I seen this structure before?" The monitoring paid for itself — cutting off long detours early made the whole thing faster. The software equivalent is the developer who rubber-ducks early (forcing articulation of the problem often reveals the bug immediately) versus the one who keeps reading the same function expecting a different insight.

Glaser's (1996) change-of-agency model frames this as a developmental arc. In the early stage of any domain, monitoring is externalized: teachers, coaches, and rubrics provide the feedback loop that the learner can't yet run internally. In the transitional stage, the learner begins developing self-monitoring — identifying criteria for high performance and tracking their own progress against them. The final, self-regulatory stage is the metacognitive ideal: the learner designs their own practice environment, identifies their own gaps, and decides when external input is worth seeking. The stages track more than growing skill; they track how far the monitoring has moved inside the learner. Chi, Glaser, and Rees (1983) found a concrete instance of this in physics: experts were better than novices not only at solving problems but at assessing a problem's difficulty in advance and knowing which schemas applied. Knowing what you don't yet know — and being right about it — is a metacognitive skill as much as a domain knowledge skill.

Metacognition is also what makes the rest of this post pay off. You can't rotate strategies on purpose without noticing which one you're on, step away at the right moment without tracking how long you've been stuck, or build schemas from experience without stopping to reflect. Externalizing the monitoring loop — logging representations, strategies, and outcomes — gives you data about your own thinking patterns that you can't access from inside a stuck problem.

## Practical takeaways

These are reference items — templates and habits that apply the mechanisms above.

### Before starting: build the representation

1. **Classify the problem type** — knowledge-lean or knowledge-rich? Well-defined or ill-defined? This determines which approaches apply and whether you have a schema to retrieve.
2. **Name the structural pattern** — "Is this a convergence problem? A search problem? A constraint satisfaction problem?" The structural label primes retrieval of relevant analogues across domains, regardless of surface domain.
3. **Goal state** — what exactly must be achieved?
4. **Current state** — what is known and observed?
5. **Operators** — what actions, techniques, or tools are available?
6. **Constraints** — which are real? which are assumed?
7. **Unknowns** — what data is missing?
8. **Alternative forms** — can this be restated visually, mathematically, or stepwise?

For each constraint, explicitly label it: real (externally imposed), assumed (inferred but unverified), negotiable, or unnecessary. Most breakthroughs come from discovering a constraint was assumed, not real.

### When stuck: rotate strategies

If progress has stopped, the strategy is probably the problem. Before rotating, ask: is the approach I'm reaching for there because the surface of this problem matches a prior one — not because the structure does? Surface similarity drives retrieval, and the Einstellung effect means the most available strategy may be the wrong one. Then rotate:

- **Means-end analysis** — identify gaps, reduce them systematically
- **Working backward** — start from goal state, reverse-engineer; good for debugging
- **Generate-and-test** — brainstorm multiple paths, test small steps quickly
- **Constraint relaxation** — what if X isn't actually required?
- **Structural analogy** — what shares the same relational structure as this problem? (not "have I seen something like this before?")
- **Divide-and-conquer** — split into smaller subproblems
- **Diagram/visualization** — turn the abstract into shapes or flow
- **Counterfactual** — what would I do if I had to solve this in 10 minutes?

### After 8–12 minutes stuck: trigger diffuse mode

- Walk, switch context, sketch abstractly, restate with different constraints
- "How would a complete beginner see this?"
- "What problem would have this as a solution?" (reverse the problem)
- Re-label the structural pattern — "Is this actually a decomposition problem? A convergence problem?" Shifting the label can trigger re-encoding without leaving the desk.
- Change modality: text → diagram → equation → pseudocode

### Building schemas

A schema entry needs: the **structural label** (the relational pattern, not the surface domain), conditions of applicability, failure modes, and a worked example. The structural label is what makes the schema retrievable across domains — not "Kubernetes networking problem" but "multi-hop dependency with invisible intermediary."

Intermediate effect warning: accumulating more detail at the same representational level isn't progress. The goal is compilation into higher-level patterns that suppress irrelevant detail. If you're recalling more propositions but solving problems no faster, you're at the intermediate plateau, not advancing toward expert.

Deliberate practice means drilling at the boundary of current competence — problems that feel effortful, not fluent. Comfortable repetition within established skill produces little schema improvement.

Domain examples:

- Kubernetes: bootstrap steps, PV/PVC patterns, CNI configs — structural label: ordered dependency chains with external state
- Networking: DHCP/DNS/mDNS resolution chains — structural label: hierarchical lookup with caching and staleness
- Math: factoring, substitution, completing the square — structural label: reformulation to expose exploitable structure
- Programming: container setup patterns, FP combinators, OOP encapsulation — structural label: abstraction of shared behavior behind stable interfaces

### Encoding new material

- **Structural labels first** — encode not just what you did, but what structural pattern it was. "I fixed a DNS caching bug" is a surface label. "I solved a stale-state contamination problem" is a structural label that will retrieve when a memory interference problem or cache invalidation problem appears in a different domain.
- **Relational skeleton** — encode what roles the elements play relative to each other, not just what they are. The relations are what transfers; the objects don't.
- **Elaborative encoding** — link to an existing mental model, personal experience, or analogy; don't just restate the definition
- **Dual coding** — pair verbal with visual (diagram, boxes-and-arrows, timeline)
- **Spaced repetition** — review sequence: day 1 → 3 → 7 → 14 → 30

### Finding structural transfers

The central question is "What relational structure does this problem have?" — before asking what else shares it. Not "have I seen something like this before?" (surface-driven) but "what structural pattern is this?" (structure-driven).

Three-step transfer checklist:

1. **Retrieve** — find a structural analogue, not a surface match. If nothing comes to mind, label the structure more abstractly until something in memory responds.
2. **Map** — identify which objects in the source play the same role as which objects in the target.
3. **Adapt** — modify the source solution to fit the target's constraints.

If step 1 fails, it's a retrieval failure, not a knowledge failure — the analogue may be there but stored under the wrong label. Build a personal analogy library: for each solved problem, record the structural label. These become retrieval hooks for future problems across all domains.

Structural mappings worth building:

- State machine ↔ Kubernetes node join flow (ordered transitions with guard conditions)
- Stale cache ↔ memory interference (prior state contaminating current lookup)
- Convergence ↔ radiation/fortress problem (distributed resources applied simultaneously to a single point)
- Graph search ↔ network troubleshooting (traversal with cost or constraint per edge)
- Abstraction barrier ↔ Terraform module / OOP encapsulation (internal structure hidden behind stable interface)
- Substitution ↔ code refactoring (replace one form with a structure-preserving equivalent)

### Problem-solving journal

For each non-trivial problem, log:

- The original representation and how it changed
- Strategies tried and where each stalled
- What finally worked
- **Structural label** — what pattern was this really? (convergence, search, decomposition, etc.)
- **Negative transfer check** — did a prior method bias the search based on surface similarity?
- **Transfer source** — if an analogue helped, what made it findable — surface or structural recognition?
- What the experience revealed about your thinking patterns

Over time this becomes a personal structural pattern library — more useful than a log of domain-specific events.

### Daily loop

1. Represent the problem (5 min)
2. Classify the problem type and name the structural pattern (1–2 min)
3. Pick a strategy appropriate to that type (1–2 min)
4. Try for 8–12 min
5. If stuck → diffuse mode or rotate strategy
6. Solve or decompose further
7. Reflect: log the structural label, any negative transfer, and what finally worked

### Anchoring learning to real projects

Abstract practice rarely transfers. Attach new concepts to a real system, and when you do, explicitly name the structural mapping — not just "I applied X to Y" but "X and Y share structure Z, so the solution transferred."

- New networking concept → test it in the homelab; label the structural pattern
- New math technique → apply to a simulation or procedural graphic; identify the relational skeleton
- New OOP pattern → build a real CLI or module; name what abstraction it implements
- New Linux internals → reproduce the behavior in a Kubernetes context; map the structural equivalence

The anchor is transferable only if the relational structure is explicit — otherwise it stays a domain-specific fact.

### Deliberate creativity practice

This is the practice of building adaptive expertise — the schemas that cover exceptions, not just typical cases. Routine expertise handles familiar problems efficiently; adaptive expertise handles novel ones. The gap is trained by working on problems that force re-encoding, constraint relaxation, and chunk decomposition (Ohlsson's RCT escape mechanisms).

Insight-heavy problems serve this function: math puzzles requiring non-obvious transformations, lateral thinking puzzles, spatial problems, short creative coding exercises (fractals, simulations, animations). Treat it as targeted training for the parts of problem-solving that deliberate domain practice doesn't reach, rather than as time off from real work.

## Gotchas

- **Assuming constraints are real.** The nine-dot and candle problems hinge on constraints the solver invented. Labeling them explicitly is the forcing function.
- **Hill climbing into a local minimum.** Feels productive right up until it doesn't. If progress has stopped, the strategy is probably the problem.
- **Mistaking familiarity for understanding.** Memory reconstructs — you can feel confident in a recalled solution that's actually wrong. Schema quality matters as much as quantity.
- **Transfer doesn't happen automatically.** Solving lots of coding problems doesn't make you better at math. Structure must be made explicit and encoded by structural label, not surface domain.
- **Pushing harder when stuck blocks insight.** Diffuse mode is the mechanism, not a break.
- **Negative transfer is invisible.** The Einstellung effect operates below awareness — you apply a method because the surface matched a prior problem, without realizing the structure doesn't. It doesn't feel like a mistake; it feels like experience.
- **Retrieval feels right even when it's wrong.** Surface similarity drives what gets retrieved from memory. You'll feel like you remembered the relevant analogue when you actually retrieved a surface match that doesn't structurally help. Confidence in retrieval is not evidence of structural fitness.
- **More knowledge isn't always progress.** The intermediate effect: accumulating propositions at the same representational level can worsen performance on some tasks relative to novices. Progress requires compilation into higher-level schemas, not just accumulation of facts.
- **Expertise breeds dogmatism.** The earned dogmatism effect: perceiving yourself as expert creates a license to dismiss alternatives without engaging. Expertise can narrow search as much as deepen it.

## Open questions

- How much of metacognitive skill is domain-general vs. domain-specific?
- What's the minimum effective chunk — how explicit does a schema need to be documented before it becomes overhead rather than leverage?
- Does deliberate structural labeling at encoding actually improve far-transfer retrieval in naturalistic (non-lab) settings, or only in prompted conditions like Gick & Holyoak?
- When does negative transfer override positive, and is there a reliable real-time signal that surface has overridden structure in your own search?
- Does the intermediate-effect plateau have to be endured, or can deliberate practice accelerate compilation into higher-level schemas?
- Is adaptive expertise trainable directly, or is it a byproduct of sufficient breadth of deliberate practice across varied problem types?

## References

- Boshuizen, H.P.A., & Schmidt, H. G. (1992). [On the role of biomedical knowledge in clinical reasoning by experts, intermediates and novices](https://doi.org/10.1207/s15516709cog1602_1). _Cognitive Science_, 16(2), 153–184
- Bowdle, B. F., & Gentner, D. (2005). [The career of metaphor](https://doi.org/10.1037/0033-295X.112.1.193). _Psychological Review_, 112(1), 193–216
- Bransford, J. D., Barclay, J. R., & Franks, J. J. (1972). Sentence memory: A constructive versus interpretative approach. _Cognitive Psychology_, 3, 193–209
- Chase, W. G., & Simon, H. A. (1973). _The Mind's Eye in Chess_. Academic Press
- Chen, Z., Mo, L., & Honomichl, R. (2004). [Having the memory of an elephant: Long-term retrieval and the use of analogues in problem solving](https://doi.org/10.1037/0096-3445.133.3.415). _Journal of Experimental Psychology: General_, 133(3), 415–433
- Chi, M.T.H., Feltovich, P. J., & Glaser, R. (1981). Categorization and representation of physics problems by experts and novices. _Cognitive Science_, 5(2), 121–152
- Chi, M.T.H., Glaser, R., & Rees, E. (1983). Expertise in problem solving. In R. J. Sternberg (Ed.), _Advances in the Psychology of Human Intelligence_ (Vol. 2, pp. 7–75). Erlbaum
- De Groot, A. D. (1965). _Thought and Choice in Chess_. Mouton
- Dreyfus, H. L. (1997). Intuitive, deliberative, and calculative models of expert performance. In C. E. Zsambok & G. Klein (Eds.), _Naturalistic Decision Making_ (pp. 17–28). Lawrence Erlbaum
- Duncker, K. (1945). On problem solving. _Psychological Monographs_, 58
- Ericsson, K. A., & Charness, N. (1994). [Expert performance: Its structure and acquisition](https://doi.org/10.1037/0003-066X.49.8.725). _American Psychologist_, 49(8), 725–747
- Ericsson, K. A., & Kintsch, W. (1995). Long-term working memory. _Psychological Review_, 102, 211–245
- Ericsson, K. A., Krampe, R. T., & Tesch-Römer, C. (1993). [The role of deliberate practice in the acquisition of expert performance](https://doi.org/10.1037/0033-295X.100.3.363). _Psychological Review_, 100(3), 363–406
- Fisher, M., & Keil, F. C. (2015). [The curse of expertise](https://doi.org/10.1111/cogs.12280). _Cognitive Science_, 40(5), 1251–1269
- Fitts, P. M., & Posner, M. I. (1967). _Human Performance_. Brooks/Cole
- Flavell, J. H. (1979). [Metacognition and cognitive monitoring](https://doi.org/10.1037/0003-066X.34.10.906). _American Psychologist_, 34(10), 906–911
- Gentner, D. (1983). Structure-mapping: A theoretical framework for analogy. _Cognitive Science_, 7, 155–170
- Gentner, D., & Gentner, D. R. (1983). Flowing waters or teeming crowds: Mental models of electricity. In _Mental Models_. Lawrence Erlbaum
- Gentner, D., Rattermann, M. J., & Forbus, K. D. (1993). The roles of similarity in transfer: Separating retrievability from inferential soundness. _Cognitive Psychology_, 25, 524–575
- Gick, M. L., & Holyoak, K. J. (1983). [Schema induction and analogical transfer](<https://doi.org/10.1016/0010-0285(83)90002-6>). _Cognitive Psychology_, 15(1), 1–38
- Gilhooly, K. J., & Murphy, P. (2005). Differentiating insight from non-insight problems. _Thinking & Reasoning_, 11(3), 279–302
- Glaser, R. (1996). Changing the agency for learning: Acquiring expert performance. In K. A. Ericsson (Ed.), _The Road to Excellence_ (pp. 303–311). Lawrence Erlbaum
- Greeno, J. G. (1974). Hobbits and Orcs: Acquisition of a sequential concept. _Cognitive Psychology_, 6, 270–292
- Hatano, G., & Inagaki, K. (1986). Two courses of expertise. In H. Stevenson, H. Azuma, & K. Hakuta (Eds.), _Child Development in Japan_ (pp. 262–272). Freeman
- Holyoak, K. J., & Koh, K. (1987). Surface and structural similarity in analogical transfer. _Memory and Cognition_, 15(4), 332–340
- Jonassen, D. H. (1997). Instructional design models for well-structured and ill-structured problem-solving learning outcomes. _Educational Technology Research and Development_, 45(1), 65–94
- Kaplan, C. A., & Simon, H. A. (1990). [In search of insight](<https://doi.org/10.1016/0010-0285(90)90008-R>). _Cognitive Psychology_, 22(3), 374–419
- Knoblich, G., Ohlsson, S., Haider, H., & Rhenius, D. (1999). Constraint relaxation and chunk decomposition in insight problem solving. _Journal of Experimental Psychology: Learning, Memory, and Cognition_, 25(6), 1534–1556
- Loftus, E. F. (1975). [Leading questions and the eyewitness report](<https://doi.org/10.1016/0010-0285(75)90023-7>). _Cognitive Psychology_, 7(4), 560–572
- Luchins, A. S. (1942). Mechanization in problem solving: The effect of Einstellung. _Psychological Monographs_, 54(248)
- MacGregor, J. N., Ormerod, T. C., & Chronicle, E. P. (2001). [Information processing and insight](https://doi.org/10.1037/0278-7393.27.1.176). _Journal of Experimental Psychology: Learning, Memory, and Cognition_, 27(1), 176–201
- Newell, A., & Rosenbloom, P. S. (1981). Mechanisms of skill acquisition and the law of practice. In J. R. Anderson (Ed.), _Cognitive Skills and their Acquisition_ (pp. 1–56). Erlbaum
- Newell, A., & Simon, H. A. (1972). _Human Problem Solving_. Prentice-Hall
- Ohlsson, S. (1992). Information processing explanations of insight and related phenomena. In M. T. Keane & K. J. Gilhooly (Eds.), _Advances in the Psychology of Thinking_ (pp. 1–43). Harvester-Wheatsheaf
- Ottati, V., Price, E. D., Wilson, C., & Sumaktoyo, N. (2015). When self-perceptions of expertise increase closed-minded cognition: The earned dogmatism effect. _Journal of Experimental Social Psychology_, 61(1), 131–138
- Polya, G. (1957). _How to Solve It_. Princeton University Press
- Robertson, S. I. [_Problem Solving_, 2nd ed.](https://learning.oreilly.com/library/view/problem-solving-2nd/9781317496007/) (Psychology Press)
- Simon, H. A. (1973). The structure of ill-structured problems. _Artificial Intelligence_, 4, 181–201
- Simon, H. A., & Hayes, J. R. (1976). [The understanding process: Problem isomorphs](<https://doi.org/10.1016/0010-0285(76)90022-0>). _Cognitive Psychology_, 8, 165–190
- Simon, H. A., & Newell, A. (1971). [Human problem solving: The state of the theory in 1970](https://doi.org/10.1037/h0030806). _American Psychologist_, 26(2), 145–159
- Sio, U. N., & Ormerod, T. C. (2009). [Does incubation enhance problem solving? A meta-analytic review](https://doi.org/10.1037/a0014212). _Psychological Bulletin_, 135(1), 94–120
- Wertheimer, M. (1945). _Productive Thinking_. Harper & Row
- Wheatley, G. H. (1984). _Problem Solving in School Mathematics_ (MEPS Technical Report 84.01). Purdue University, School Mathematics and Science Center
- [Metacognition](https://en.wikipedia.org/wiki/Metacognition) — Wikipedia
- [Reconstructive memory](https://en.wikipedia.org/wiki/Reconstructive_memory) — Wikipedia
- [Transfer of learning](https://en.wikipedia.org/wiki/Transfer_of_learning) — Wikipedia
