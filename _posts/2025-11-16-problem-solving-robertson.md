---
layout: post
title: "Problem Solving — S. Ian Robertson"
date: 2025-11-16
description: Notes on Robertson's cognitive framework for problem solving — representation, heuristics, insight, schemas, and metacognition.
tags: [cognitive-science, problem-solving, learning, books]
---

## Summary

At its core, problem solving is navigating toward a goal — but the goal itself isn't always clear in advance. Sometimes it's precisely defined; sometimes you'll only recognize it when you see it. That second case turns out to be surprisingly common in real work, and it means that goal ambiguity is often the actual difficulty, not the path-finding.

The field has been studied from several distinct traditions that each illuminated a different piece. **Behaviourists** (Thorndike, Watson, Skinner) explained problem solving as trial-and-error learning shaped by reinforcement — useful for describing how organisms adapt, but silent on the internal structures that make complex reasoning possible. **Gestalt psychologists** (Wertheimer, Duncker, Koffka) shifted focus to perception and restructuring: insight — the sudden "Aha!" — wasn't random but arose from a reorganization of how the problem was seen. **Cognitive psychologists** (Newell, Simon, Anderson) formalized this into the problem space framework: a problem is defined by an initial state, a goal state, and operators that move between them; solving it is search through that space. **Educationalists** (Polya, Schoenfeld) brought the question down to practice — what heuristics and metacognitive habits actually help people solve problems they've never seen before?

Robertson's book draws on all four traditions. The cognitive framework is central, but the Gestalt insight into representation and the educationalist emphasis on deliberate strategy both run through the treatment.

**Problem taxonomy** — what kind of problem you have determines which approaches apply:

| Axis | Poles |
|---|---|
| Knowledge requirement | **Knowledge-lean** (domain-general strategies suffice) ↔ **Knowledge-rich** (requires domain-specific expertise) |
| Goal specification | **Well-defined** (all needed info is given or inferable) ↔ **Ill-defined** (goal or constraints are vague) |
| Solver experience | **Semantically lean** (little relevant experience) ↔ **Semantically rich** (deep experience of this problem type) |
| Solution character | **Standard** (deliberate search through operators) ↔ **Insight** (solution preceded by sudden restructuring) |

A knowledge-lean, well-defined problem — a logic puzzle, a river-crossing problem — calls for very different approaches than an ill-defined, knowledge-rich one like debugging a production incident or designing a system under unclear requirements. Identifying which kind of problem you're facing before starting is itself a meta-skill.

Schemas are the memory structures that make this classification fast in practice: organized representations of problem types in long-term memory that let you recognize "this is a problem of type X" and retrieve the relevant approach without having to reason from scratch. Building them deliberately is most of what expertise development looks like.

## Key concepts

### Representation determines what's reachable

Newell and Simon's foundational work frames every problem as a **problem space**: an initial state, a goal state, a set of operators that move between states, and a search process that navigates the space. This formalism makes representation precise — it isn't just "how you think about the problem," it's the structure that determines which states exist, which operators are legal, and therefore which solutions are reachable at all. A representation that omits a relevant variable or encodes a constraint wrong doesn't make the problem harder; it makes the correct solution unreachable by definition, no matter how hard or cleverly you search.

The **Hobbits and Orcs** problem makes this concrete. Three hobbits and three orcs must cross a river using a boat that holds at most two; orcs can never outnumber hobbits on either bank. Greeno (1974) and Thomas (1974) both studied how solvers approach this. Many people represent the state informally — a vague mental image of who is where — and quickly run into what feel like dead ends or illegal moves. The difficulty isn't the logic; it's the representation. A precise tuple `(hobbits_left, orcs_left, boat_side)` makes every legal operator explicit: enumerate the moves that keep each component valid, and the solution path becomes a straightforward search. Solvers who represented the state space this way made far fewer errors and reached the solution faster. The problem didn't change; the representation did.

The **nine-dot problem** is a starker example. Connect nine dots arranged in a 3×3 grid using four straight lines without lifting the pen. Most people fail because they self-impose a square boundary around the dots — a constraint the problem statement never stated. The search space they're exploring doesn't contain the solution. The boundary isn't in the problem; it's in the representation they constructed when they read it.

This last point is what makes representation especially tricky. Bransford, Barclay, and Franks (1972) showed that people don't encode problem statements neutrally — they construct an interpretation, and that interpretation can include inferences and assumptions the statement never made. You read "nine dots in a grid" and your encoding includes an imaginary boundary. You read "the server is timing out" and your encoding includes an implicit assumption about which server. The construction happens automatically; catching it requires deliberate effort.

A practical version: if you represent a failing Kubernetes pod as "the image is broken," your search focuses on the registry and build pipeline. Represent it as "the pod can't reach a ready state" and you open the search to readiness probes, resource limits, scheduling constraints, and network policy. The operators available to you are determined by the representation. Choosing the frame first is the highest-leverage step before doing anything else.

### Heuristics are useful but bounded

Heuristics are mental shortcuts that make search tractable — without them, the space of possible moves in almost any real problem is too large to explore exhaustively. Robertson focuses on three: hill climbing (always take the step that moves closest to the goal), means-end analysis (identify the gap between current state and goal state, then select an operator that reduces it), and working backward (start from the goal and ask what would need to be true immediately before it).

Each is genuinely useful, and each has a characteristic failure mode. Hill climbing fails when you have to move *away* from the goal to ultimately reach it — a maze with dead ends is the simple version, but software dependency resolution is a real one where installing the right package requires temporarily removing a conflicting one. Means-end analysis can get locked onto a subgoal that isn't actually necessary, spending effort closing a gap the solution doesn't require; a classic example is a student who rewrites a proof step to get closer to a known lemma, not realizing the proof doesn't need that lemma at all. Working backward is powerful for debugging (trace from the error back through the call stack to the source) but doesn't generalize well to open-ended design problems where the goal state is fuzzy.

The Gestalt tradition identified two mechanisms that explain why heuristics break down even when the solver has all the information they need.

**Functional fixedness** (Duncker 1945): solvers encode objects in their conventional role and fail to see them as candidates for other uses. The thumbtack box in the candle problem never enters the search as a shelf because the initial encoding locked it out as a container. This isn't a reasoning error — it's a representational one. A real-world version: a developer who can't see a config file as a substitute for a database entry because "config files don't work that way" is imposing a constraint that exists only in the encoding, not in the problem.

**Set effects / Einstellung** (Luchins 1942): solving a class of problems with a particular method creates a persistent bias toward that method even when simpler alternatives exist. Luchins's water jug problems show this directly: participants who learned a complex filling procedure applied it to problems with a trivial direct solution, often without noticing the shortcut was available. The set operates below awareness, biasing which operators the solver considers at all.

Wertheimer's distinction between **reproductive thinking** (applying learned procedures regardless of fit) and **productive thinking** (reasoning from the actual structure of the current problem) frames both: functional fixedness and set effects are both forms of structural blindness. Knowing this is why strategy rotation works — it isn't just a general fallback, it's specifically a way of fighting the implicit exclusions that set and fixedness introduce.

### Insight requires leaving the current search space

Insight — the sudden shift that makes a previously intractable problem feel obvious — isn't a random gift. The key theoretical reframe comes from Kaplan and Simon (1990): conventional problem solving is searching **in** a representation, navigating the problem space you already have. Insight is searching **for** a representation — finding a better problem space first. This isn't a categorically different cognitive process; it's search operating over the space of possible framings rather than the space of moves within a given framing. When you're stuck, you aren't following the wrong path — you're on the wrong map.

Ohlsson's **Representational Change Theory (RCT)** explains why impasse happens and how the brain escapes it. The initial representation is assembled from salient features and whatever long-term memory retrieves first. If the solution isn't reachable from that representation, you hit impasse — forward progress stops regardless of effort. Four escape mechanisms:

- **Re-encoding**: attention shifts to different features; activation is subtracted from elements that produced dead ends and redistributed across others
- **Constraint relaxation**: drop constraints inadvertently imposed — the nine-dot imaginary boundary, the tack box as container-only
- **Elaboration**: add information that wasn't in the initial encoding
- **Chunk decomposition**: break elements into sub-elements, exposing structure hidden at the chunk level (matchstick arithmetic is the canonical case)

These are largely unconscious — they happen through spreading activation and cued retrieval, not deliberate reasoning.

MacGregor et al.'s **Criterion for Satisfactory Progress Theory (CSPT)** adds the conscious side. Solvers use hill climbing while monitoring progress. When expected progress isn't achieved — criterion failure — they look for other states to expand. Working memory capacity matters: solvers who can look further ahead before hitting criterion failure are more likely to recognize that a solution requires an initially unpromising move. In the nine-dot problem, people who could anticipate further consequences of each line were more likely to discover that lines must extend outside the imaginary boundary.

RCT and CSPT aren't competing — they're complementary. Gilhooly and Murphy (2005) found evidence for both unconscious (Type 1) and conscious (Type 2) processes in insight, with the balance depending on problem type. Perceptual and spatial insight problems draw more on unconscious chunk decomposition; verbal and logical ones involve more conscious monitoring and search. Incubation — stepping away from a stuck problem — works because it allows unconscious re-encoding to run without conscious hill climbing suppressing the process by reinforcing the failed frame.

Norman Maier's **two-string problem** demonstrates the unconscious mechanism directly: participants must tie together two strings that are too far apart to reach simultaneously. Most keep trying variations on walking between them. The solution requires reframing one string as a pendulum. Solvers who were casually nudged toward pendulum-like motion by the experimenter solved it faster without realizing why — the nudge triggered re-encoding below awareness. A software equivalent: the developer who's spent an hour convinced the bug is a logic error in a function, takes a five-minute walk, and immediately wonders whether it's an encoding issue upstream — and it is. The information didn't change; the representation it was organized into did.

### Expertise is schema density, not raw speed

Expert performance has three characteristic signatures. First, **fast categorization**: experts classify problems by deep structure, not surface features. Chi, Feltovich, and Glaser (1981) found that novice physics students grouped problems by surface appearance ("this is an inclined plane problem") while experts grouped them by underlying principle ("this is a conservation of energy problem") — a fundamentally different representation that immediately implies a solution strategy. Second, **perceptual chunking**: experts encode larger meaningful configurations as single units, letting them process more information per cognitive step. Third, **long-term working memory**: Ericsson and Kintsch (1995) documented that experts develop domain-specific encoding strategies that effectively extend working memory into long-term memory — they can retrieve and manipulate information that novices would need to hold in limited-capacity short-term memory.

The chess evidence established all three. De Groot (1965) first showed that grandmasters didn't examine more moves than masters — they examined better moves, because they recognized meaningful board configurations immediately. Chase and Simon (1973) made the mechanism explicit: expert players reconstructed realistic mid-game positions from five-second exposures; when pieces were placed randomly, the advantage disappeared entirely. The skill was in pattern recognition applied to meaningful chess structure, not memory capacity or processing speed. The same signatures appear in radiology (Lesgold et al. 1988: experts see clinically meaningful regions; novices see individual features), programming (McKeithen et al. 1981: expert programmers chunk code into functional units), and physics problem solving.

**How expertise develops.** Three models from the chapter each illuminate a different dimension. Fitts and Posner (1967) describe three stages: a *cognitive* stage (resource-intensive, reliant on declarative knowledge, slow and error-prone), an *associative* stage (rules are compiled, performance becomes more automatic), and an *autonomous* stage (highly proficient, no longer reliant on conscious control). Dreyfus (1997) extends this to five stages — novice, advanced beginner, competent, proficient, expert — and makes a point often missed: rule-based systems cannot account for expert intuition. Experts don't follow rules; they perceive situations holistically and respond based on pattern recognition accumulated over thousands of exposures. This is why experts are often poor at explaining what they do. Glaser (1996) focuses on the changing locus of control: from *external support* (dependent on teachers and coaches), through *transitional* (support gradually withdrawn), to *self-regulatory* (learning fully under the learner's own direction).

The **Power Law of Practice** describes the shape of skill acquisition: rapid improvement early, dramatically diminishing returns as practice accumulates (Newell & Rosenbloom 1981). This is consistent across domains — it's a structural feature of skill acquisition, not a motivation problem. It explains why early study of a domain has very high return and why pushing past intermediate competence requires disproportionate effort.

**Deliberate practice** (Ericsson, Krampe & Tesch-Römer 1993) is what determines how fast you move through those stages. Not all practice is equal. Deliberate practice is effortful, aimed at the edges of current ability, with immediate and accurate feedback, focused on specific components that need improvement. Comfortable practice within well-established skill produces much less improvement than shorter periods of targeted, difficult work at the boundary of competence. Ericsson and Charness (1994) extend this: expertise involves continuous effortful adaptation to domain demands, not consolidation of learned routines.

**The intermediate effect.** Boshuizen and Schmidt (1990, 1992) documented a striking pattern in medical expertise: people with intermediate knowledge can *outperform* both novices and experts on some tasks, and be *outperformed by novices* on others. Intermediates encode too much detail — they "process too much garbage" — and so recall more clinical propositions than experts, but make worse diagnostic decisions because the detail obscures the pattern. Experts have compiled their knowledge into higher-level scripts that suppress irrelevant detail. The implication: accumulating more knowledge at the same representational level isn't always progress; what matters is whether knowledge is compiled into higher-level schemas.

**Domain specificity.** Expertise doesn't transfer across domains. Content knowledge is domain-specific (deep physics knowledge doesn't help with medical diagnosis). Task knowledge — procedural and strategic knowledge — transfers only to closely related domains that share a genuine skill overlap. There is no such thing as a general expert reasoner in the meaningful sense; there are domain experts and the domains may or may not share transferable skills.

**The automaticity paradox.** Automatisation ought to make expert performance brittle — if procedures run without conscious access, how do experts handle novel problems? Hatano and Inagaki (1986) resolve this with the distinction between *routine expertise* (fast, efficient, handles typical problems with compiled procedures) and *adaptive expertise* (flexible, handles novel and exceptional cases through schemas that explicitly represent exceptions and edge cases). Expert schemas aren't just libraries of common cases — they include strategies and knowledge structures for when the common case doesn't apply. This is what distinguishes a senior engineer who can debug an unfamiliar system from one who can only optimize systems they built.

**Downsides of expertise.** Two documented failure modes. Ottati et al. (2015) demonstrated the *earned dogmatism effect*: people who perceive themselves as expert become more closed-minded — expertise feels like a license to dismiss alternatives without engaging with them. Fisher and Keil (2015) documented the *illusion of explanatory depth*: experts may have forgotten how much they've forgotten; they report more explanatory competence than they actually possess when pressed to generate detailed mechanistic explanations. Both are calibration failures rather than competence failures — the expertise is real, but the metacognitive monitoring of its limits is broken.

### Memory is reconstructive, not retrieval

A common mental model of memory is a recording — you store an experience, and later you play it back. The cognitive science is very different: memory is reconstructive. When you remember something, the brain assembles the recollection from fragments, filling gaps with inference, prior expectations, and current context. This makes memory fast and flexible but also systematically unreliable in specific ways.

Elizabeth Loftus's research on eyewitness testimony is the most studied example. Her experiments showed that the phrasing of a post-event question — "How fast were the cars going when they *smashed* into each other?" vs. "contacted" — significantly changed what participants later remembered, including whether they recalled seeing broken glass that wasn't there. The memory wasn't retrieved; it was reconstructed around the implicit suggestion. A more technical analogue: write a complex module, set it aside for six months, then need to modify it. You'll feel confident about how it works — and be wrong about specific details in ways that cause bugs. The confidence is real; it comes from the reconstruction feeling fluent, not from the reconstruction being accurate.

The consequence for schema building: a schema that gets reinforced through repeated reconstruction of a slightly wrong version becomes a well-practiced error. Schema quality — whether the pattern you've chunked is actually correct — matters as much as schema quantity.

**Retrieval is surface-driven, not structure-driven.** When you encounter a problem and search memory for a relevant analogue, retrieval is largely governed by surface similarity — whether the objects, vocabulary, and perceptual features of the new situation match something stored. Structural similarity, which is what actually determines whether the analogue will help, doesn't drive retrieval as reliably. Gentner, Rattermann, and Forbus (1993) demonstrated this directly: people were more likely to spontaneously retrieve surface-similar analogues even when the structurally similar analogue was the one that would actually solve the problem. The two properties — *retrievability* and *inferential soundness* — come apart in ways that matter enormously.

This is the mechanism behind the Gick & Holyoak fortress/tumor finding. Participants had the relevant structural analogue in memory — they'd just read the fortress convergence story — but didn't spontaneously apply it to the radiation problem because the surface domains are completely different. The analogue was there; the retrieval process didn't surface it. Only when participants were explicitly prompted to use the story as an analogy did transfer occur at meaningful rates.

Chen, Mo, and Honomichl (2004) showed that even analogues learned long ago can be retrieved effectively when structural features were encoded strongly at learning time. The practical implication: encoding structural labels matters. Not just "I solved a rate problem" but "I solved a convergence problem" — a label that primes retrieval when the next convergence problem appears regardless of what domain it's in. Encoding strategies like elaboration and dual coding work partly because they build richer reconstructive cues, not because they improve storage of raw content.

### Transfer requires structural similarity

The hope that becoming good at hard problems in one domain makes you better at hard problems in general is mostly false. Transfer of skill happens when two problems share underlying abstract structure — not surface features, not domain vocabulary, not difficulty level.

**Transfer types.** Transfer can be positive (prior experience helps), negative (prior experience interferes — what the Einstellung effect produces when a learned procedure is applied where it no longer fits), near (similar domain, slightly different problem), or far (different domain, same underlying structure). Far positive transfer is what people usually mean when they talk about generalizable problem-solving skill, and it's also the hardest to reliably produce.

Problems with identical underlying structure but different surface content are called **isomorphs**. Simon and Hayes (1976) demonstrated that isomorphic problems can vary dramatically in difficulty: two river-crossing problems with identical state-space structure differ in solution rate because the surface framing changes which operators feel natural to try. **Homomorphs** are structurally similar but not identical — the same template with slight variations in constraints. Both transfer when solvers notice the structural match; neither transfers reliably when they don't.

**Three kinds of similarity** determine whether transfer occurs. *Surface similarity* means the objects or perceptual features of the two problems resemble each other. *Relational similarity* (also called structural similarity) means the relations between objects are the same — the underlying structure maps. *Procedural similarity* means the same solution method applies even if surface and structure differ. Surface similarity drives what gets *retrieved* from memory; relational similarity determines whether the analogue is actually *useful*. These two come apart: you can retrieve a surface-similar analogue that gives you nothing, or fail to retrieve a structurally identical one because it looks nothing like the current problem.

**Successful analogical transfer requires three things in sequence**: retrieval of a relevant analogue from long-term memory, mapping of corresponding roles across source and target (which objects in the source play the same role as which objects in the target), and adaptation of the source solution to the target context. Failure at any step blocks transfer. Most naturally occurring transfer failure is a retrieval failure — the right analogue is in memory but wasn't retrieved because it doesn't look like the current problem on the surface.

Gick and Holyoak's (1983) radiation/fortress study is the canonical demonstration. A doctor must destroy a tumor using radiation, but a strong-enough ray damages surrounding tissue; the solution is multiple weak rays converging from different directions. Most participants fail. Participants who had previously read a story about a general who converged small army groups along separate roads to capture a fortress solved it at much higher rates — the underlying structure is identical. But participants who read the story without being prompted to use it as an analogy transferred poorly anyway. The structural mapping existed; retrieval didn't surface it without explicit prompting.

**Gentner's Structure Mapping Theory (SMT)** formalizes what makes an analogy work. The central claim: analogy is a mapping of relational structure from a source to a target, independent of the objects involved. What gets mapped isn't attributes of objects ("both involve radiation" / "both involve armies") but the *relations between objects* — the structural skeleton. The **principle of systematicity** says that mappings which preserve higher-order relations (relations between relations) are preferred over mappings that only match first-order features. "The atom is like a solar system" works because the relational structure maps: electrons orbit the nucleus as planets orbit the sun, and the force-governs-orbit relationship holds in both — even though the objects share no surface features. A mapping based only on "both are round" would score low on systematicity and license few useful inferences.

Gentner and Gentner (1983) showed that the choice of source analogy materially affects what inferences people draw. Participants given a "flowing water" analogy for electricity reasoned well about resistance; those given a "teeming crowds" analogy reasoned better about batteries. The metaphor wasn't decorative — it determined which structural inferences were made.

**Metaphors are compressed analogies.** Novel metaphors require active structural comparison; as they become conventional, they become category assertions and lose their live analogical force. Bowdle and Gentner (2005) call this the "career of metaphor": "electricity flows" was once a live structural analogy, now it's nearly literal. The freshest analogies are often most useful, before the structural mapping has been compressed into a label that no longer prompts active reasoning.

In practice: the habit worth building is asking "what underlying pattern does this resemble?" — not "have I seen something like this before?" A DNS caching bug and a memory interference problem look nothing alike on the surface but share the same structure of stale state contaminating fresh queries. Recognizing that is transfer. Recognizing that "both involve the network" is not.

### Metacognition is the compounding lever

Metacognition is cognition about cognition — monitoring and regulating your own thinking processes as you work. It's distinct from domain knowledge and distinct from raw ability. Two people with equal skill can have very different outcomes if one actively tracks what strategy they're using, whether it's working, and when to switch, while the other just pushes forward.

Alan Schoenfeld's research on mathematical problem solving makes this concrete. He compared novice and expert mathematicians working through unfamiliar problems in think-aloud protocols. Novices almost never stopped to evaluate whether what they were doing was working — they'd invest ten or fifteen minutes in a fruitless approach without questioning the approach itself. Expert mathematicians regularly paused: "Is this going anywhere? What else could I try? Have I seen this structure before?" This meta-monitoring wasn't slower; it was faster overall, because it cut off long detours early. The software equivalent is the developer who rubber-ducks early (forcing articulation of the problem often reveals the bug immediately) versus the one who keeps reading the same function expecting a different insight.

Glaser's (1996) change-of-agency model frames this as a developmental arc. In the early stage of any domain, monitoring is externalized: teachers, coaches, and rubrics provide the feedback loop that the learner can't yet run internally. In the transitional stage, the learner begins developing self-monitoring — identifying criteria for high performance and tracking their own progress against them. The final, self-regulatory stage is the metacognitive ideal: the learner designs their own practice environment, identifies their own gaps, and decides when external input is worth seeking. The stages are not just a progression of skill — they're a progression of metacognitive internalization. Chi, Glaser, and Rees (1983) found a concrete instance of this in physics: experts were better than novices not only at solving problems but at assessing a problem's difficulty in advance and knowing which schemas applied. Knowing what you don't yet know — and being right about it — is a metacognitive skill as much as a domain knowledge skill.

Metacognition is also the mechanism that makes everything else in this post actually work. You can only rotate strategies deliberately if you're monitoring which strategy you're using. You can only trigger diffuse mode at the right moment if you're tracking how long you've been stuck. You can only build schemas from experience if you reflect on what happened rather than just moving on. Externalizing the monitoring loop — logging representations, strategies, and outcomes — gives you data about your own thinking patterns that you can't access from inside a stuck problem.

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

Insight-heavy problems serve this function: math puzzles requiring non-obvious transformations, lateral thinking puzzles, spatial problems, short creative coding exercises (fractals, simulations, animations). Not a break from analytical work — targeted training for the parts of problem-solving that deliberate domain practice doesn't reach.

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

- Boshuizen, H.P.A., & Schmidt, H. G. (1992). [On the role of biomedical knowledge in clinical reasoning by experts, intermediates and novices](https://doi.org/10.1207/s15516709cog1602_1). *Cognitive Science*, 16(2), 153–184
- Bowdle, B. F., & Gentner, D. (2005). [The career of metaphor](https://doi.org/10.1037/0033-295X.112.1.193). *Psychological Review*, 112(1), 193–216
- Bransford, J. D., Barclay, J. R., & Franks, J. J. (1972). Sentence memory: A constructive versus interpretative approach. *Cognitive Psychology*, 3, 193–209
- Chase, W. G., & Simon, H. A. (1973). *The Mind's Eye in Chess*. Academic Press
- Chen, Z., Mo, L., & Honomichl, R. (2004). [Having the memory of an elephant: Long-term retrieval and the use of analogues in problem solving](https://doi.org/10.1037/0096-3445.133.3.415). *Journal of Experimental Psychology: General*, 133(3), 415–433
- Chi, M.T.H., Feltovich, P. J., & Glaser, R. (1981). Categorization and representation of physics problems by experts and novices. *Cognitive Science*, 5(2), 121–152
- Chi, M.T.H., Glaser, R., & Rees, E. (1983). Expertise in problem solving. In R. J. Sternberg (Ed.), *Advances in the Psychology of Human Intelligence* (Vol. 2, pp. 7–75). Erlbaum
- De Groot, A. D. (1965). *Thought and Choice in Chess*. Mouton
- Dreyfus, H. L. (1997). Intuitive, deliberative, and calculative models of expert performance. In C. E. Zsambok & G. Klein (Eds.), *Naturalistic Decision Making* (pp. 17–28). Lawrence Erlbaum
- Duncker, K. (1945). On problem solving. *Psychological Monographs*, 58
- Ericsson, K. A., & Charness, N. (1994). [Expert performance: Its structure and acquisition](https://doi.org/10.1037/0003-066X.49.8.725). *American Psychologist*, 49(8), 725–747
- Ericsson, K. A., & Kintsch, W. (1995). Long-term working memory. *Psychological Review*, 102, 211–245
- Ericsson, K. A., Krampe, R. T., & Tesch-Römer, C. (1993). [The role of deliberate practice in the acquisition of expert performance](https://doi.org/10.1037/0033-295X.100.3.363). *Psychological Review*, 100(3), 363–406
- Fisher, M., & Keil, F. C. (2015). [The curse of expertise](https://doi.org/10.1111/cogs.12280). *Cognitive Science*, 40(5), 1251–1269
- Fitts, P. M., & Posner, M. I. (1967). *Human Performance*. Brooks/Cole
- Flavell, J. H. (1979). [Metacognition and cognitive monitoring](https://doi.org/10.1037/0003-066X.34.10.906). *American Psychologist*, 34(10), 906–911
- Gentner, D. (1983). Structure-mapping: A theoretical framework for analogy. *Cognitive Science*, 7, 155–170
- Gentner, D., & Gentner, D. R. (1983). Flowing waters or teeming crowds: Mental models of electricity. In *Mental Models*. Lawrence Erlbaum
- Gentner, D., Rattermann, M. J., & Forbus, K. D. (1993). The roles of similarity in transfer: Separating retrievability from inferential soundness. *Cognitive Psychology*, 25, 524–575
- Gick, M. L., & Holyoak, K. J. (1983). [Schema induction and analogical transfer](https://doi.org/10.1016/0010-0285(83)90002-6). *Cognitive Psychology*, 15(1), 1–38
- Gilhooly, K. J., & Murphy, P. (2005). Differentiating insight from non-insight problems. *Thinking & Reasoning*, 11(3), 279–302
- Glaser, R. (1996). Changing the agency for learning: Acquiring expert performance. In K. A. Ericsson (Ed.), *The Road to Excellence* (pp. 303–311). Lawrence Erlbaum
- Greeno, J. G. (1974). Hobbits and Orcs: Acquisition of a sequential concept. *Cognitive Psychology*, 6, 270–292
- Hatano, G., & Inagaki, K. (1986). Two courses of expertise. In H. Stevenson, H. Azuma, & K. Hakuta (Eds.), *Child Development in Japan* (pp. 262–272). Freeman
- Holyoak, K. J., & Koh, K. (1987). Surface and structural similarity in analogical transfer. *Memory and Cognition*, 15(4), 332–340
- Jonassen, D. H. (1997). Instructional design models for well-structured and ill-structured problem-solving learning outcomes. *Educational Technology Research and Development*, 45(1), 65–94
- Kaplan, C. A., & Simon, H. A. (1990). [In search of insight](https://doi.org/10.1016/0010-0285(90)90008-R). *Cognitive Psychology*, 22(3), 374–419
- Knoblich, G., Ohlsson, S., Haider, H., & Rhenius, D. (1999). Constraint relaxation and chunk decomposition in insight problem solving. *Journal of Experimental Psychology: Learning, Memory, and Cognition*, 25(6), 1534–1556
- Loftus, E. F. (1975). [Leading questions and the eyewitness report](https://doi.org/10.1016/0010-0285(75)90023-7). *Cognitive Psychology*, 7(4), 560–572
- Luchins, A. S. (1942). Mechanization in problem solving: The effect of Einstellung. *Psychological Monographs*, 54(248)
- MacGregor, J. N., Ormerod, T. C., & Chronicle, E. P. (2001). [Information processing and insight](https://doi.org/10.1037/0278-7393.27.1.176). *Journal of Experimental Psychology: Learning, Memory, and Cognition*, 27(1), 176–201
- Newell, A., & Rosenbloom, P. S. (1981). Mechanisms of skill acquisition and the law of practice. In J. R. Anderson (Ed.), *Cognitive Skills and their Acquisition* (pp. 1–56). Erlbaum
- Newell, A., & Simon, H. A. (1972). *Human Problem Solving*. Prentice-Hall
- Ohlsson, S. (1992). Information processing explanations of insight and related phenomena. In M. T. Keane & K. J. Gilhooly (Eds.), *Advances in the Psychology of Thinking* (pp. 1–43). Harvester-Wheatsheaf
- Ottati, V., Price, E. D., Wilson, C., & Sumaktoyo, N. (2015). When self-perceptions of expertise increase closed-minded cognition: The earned dogmatism effect. *Journal of Experimental Social Psychology*, 61(1), 131–138
- Polya, G. (1957). *How to Solve It*. Princeton University Press
- Robertson, S. I. [*Problem Solving*, 2nd ed.](https://learning.oreilly.com/library/view/problem-solving-2nd/9781317496007/) (Psychology Press)
- Simon, H. A. (1973). The structure of ill-structured problems. *Artificial Intelligence*, 4, 181–201
- Simon, H. A., & Hayes, J. R. (1976). [The understanding process: Problem isomorphs](https://doi.org/10.1016/0010-0285(76)90022-0). *Cognitive Psychology*, 8, 165–190
- Simon, H. A., & Newell, A. (1971). [Human problem solving: The state of the theory in 1970](https://doi.org/10.1037/h0030806). *American Psychologist*, 26(2), 145–159
- Sio, U. N., & Ormerod, T. C. (2009). [Does incubation enhance problem solving? A meta-analytic review](https://doi.org/10.1037/a0014212). *Psychological Bulletin*, 135(1), 94–120
- Wertheimer, M. (1945). *Productive Thinking*. Harper & Row
- [Metacognition](https://en.wikipedia.org/wiki/Metacognition) — Wikipedia
- [Reconstructive memory](https://en.wikipedia.org/wiki/Reconstructive_memory) — Wikipedia
- [Transfer of learning](https://en.wikipedia.org/wiki/Transfer_of_learning) — Wikipedia
