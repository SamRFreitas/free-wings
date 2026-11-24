# Evaluating the quality of an LLM agent — methods, rigor, metrics, and where a one-person setting fits

> Researched 2026-11-23 by `researcher`. Question: before any experiment
> that changes an agent's `model` or `effort`, what does the academic
> literature say about how to test the quality of an agent, quantitatively
> and qualitatively, and how does a small setting (one person, a few
> projects, tasks that never repeat identically, subscription plan) compare
> with the settings the literature describes? Peer-reviewed or
> methodologically sound papers first; each source is marked
> **peer-reviewed** (venue stated on the page read), **preprint** (arXiv
> page lists no venue), or **secondary** (developer documentation, blog,
> search-result summary). Verdicts: **documented** (the source says it),
> **inference** (our reasoning), **not found** (looked for, not located).
>
> Builds on, does not repeat:
> [2026-11-17 — model choice per agent](2026-11-17-model-choice-per-agent.md)
> (section 5: the current informal method and quality signals) and
> [2026-11-23 — cost tracking per agent](2026-11-23-cost-tracking-per-agent.md)
> (how cost is measured; cited here only where cost is a metric).
>
> This report proposes no experiment design as decided. Section 7 lists
> options for the person to choose from.

## 0. Short version

- **Documented, consistent across sources:** one run of an agent is a noisy
  sample. Papers on LLM evals, deep RL and coding agents all say a
  difference between two configurations must be reported with an
  uncertainty interval, and that small test sets give wide intervals.
- **Documented:** the usual interval method (central limit theorem, CLT)
  is unreliable on small samples; Wilson or Bayesian intervals are
  recommended instead (Bowyer et al., ICML 2025).
- **Documented:** people's own impression of an AI tool's effect can be
  wrong in sign (METR: developers expected and believed a ~20% speedup;
  measured: a 19% slowdown). A person's judgment alone is not a
  measurement.
- **Documented:** LLM judges have named biases (position, verbosity,
  self-preference, leniency) and low run-to-run self-consistency; every
  source that recommends them also recommends calibrating them against
  human labels.
- **Our setting does not appear in the literature as such.** The closest
  neighbours are single-case / small-n experimental designs (other
  fields) and the METR practitioner trial (16 people, 246 tasks). The
  large-benchmark statistics transfer only as cautions about what we
  cannot detect, not as procedures. Section 5 gives the comparison.
- **Not found:** a study that evaluates *model/effort choice per role* for
  a multi-agent coding harness on a subscription plan; any source applying
  non-inferiority or equivalence testing to LLM agent configurations;
  any quality comparison of Claude models for teaching or bilingual prose
  (already "not found" on 2026-11-17).

## 1. Methods for evaluating agents: what each measures, documented limits

| Method | What it measures | Documented limits |
| :- | :- | :- |
| **Outcome benchmarks with automated checks** (SWE-bench, AgentBench, GAIA, tau-bench) | Whether the agent reaches a verifiable end state. SWE-bench: 2,294 real GitHub issues over 12 Python repositories, resolved by editing a codebase (Jimenez et al., ICLR 2024). AgentBench: 8 interactive environments (Liu et al., ICLR 2024). GAIA: 466 questions, answers to 300 withheld (Mialon et al., preprint). tau-bench: database state at the end of a simulated conversation compared with an annotated goal state (Yao et al., preprint) | Task setup and reward design can distort scores "by up to 100% in relative terms"; SWE-bench Verified's tests "not adequate", tau-bench counted empty responses as success (Zhu et al., preprint). Weak or missing holdout sets lead to overfitting; non-standard practices hurt reproducibility; accuracy-only reporting hides cost (Kapoor et al., preprint). Possible training-data leakage and closed, changing models threaten validity (Sallou et al., ICSE 2024 NIER, peer-reviewed). One agent "searched for the benchmark on HuggingFace instead of solving the task" (Holistic Agent Leaderboard, preprint) |
| **Functional-correctness tests** (HumanEval, unit tests) | Whether generated code passes tests; reported as pass@k (Chen et al., preprint) | Tests only check what they encode; see "insufficient test cases" above |
| **Rubric scoring by an LLM judge** | Quality of open-ended output against written criteria | Section 4 |
| **Pairwise preference by an LLM judge or by people** (MT-Bench, Chatbot Arena) | Which of two outputs is preferred. GPT-4 judges reached "over 80% agreement" with human preferences, "roughly the same level" as humans with each other (Zheng et al., NeurIPS 2023 D&B) | Position, verbosity and self-enhancement bias; "limited reasoning ability" (same paper) |
| **Agent-as-a-judge** | An agentic system inspects the whole task-solving process, not only the final output; DevAI: 55 tasks, 365 hierarchical requirements (Zhuge et al., preprint) | Authors report it "as reliable as our human evaluation baseline" on their benchmark; one benchmark, preprint, not independently replicated in what I read. **Not found:** independent validation |
| **Trajectory review** (LLM-written reviews of the full run) | Instruction following, tool use, self-verification, recovery, communication (AgentLens, preprint) | Preprint; the benchmark is the authors' own |
| **Human evaluation** | Quality as judged by raters | Section 4; "notoriously unstable" and hard to reproduce (Chiang & Lee, ACL 2023, peer-reviewed); untrained raters distinguished GPT-3 text from human text "at random chance level" (Clark et al., preprint) |
| **Randomized trial on real work** | Effect of a tool on real tasks of real practitioners | METR: 16 developers, 246 tasks, each task randomly assigned AI allowed or not. Not blinded; self-reported times; "cannot be entirely ruled out" experimental artifacts; subset analyses underpowered (Becker et al., preprint) |
| **Transcript / log reading** | Whether a failure is a real agent mistake or a broken grader or task | Anthropic: "You won't know if your graders are working well unless you read the transcripts" (secondary). Example given: a CORE-Bench score of 42% rose to 95% after grading bugs were fixed (secondary, from the same post) |
| **Survey / taxonomy** | Map of the field: capabilities, application benchmarks, generalist agents, frameworks (Yehudai et al., preprint, arXiv comments say ACL Findings) | Names gaps: cost-efficiency, safety, robustness, fine-grained scalable evaluation |

**Verified that:** Anthropic's own guidance (secondary, Jan 9 2026)
combines three grader types: code-based ("fast, cheap, objective,
reproducible" but "brittle"), model-based ("captures nuance" but
"non-deterministic" and "require calibration with human graders") and
human ("gold standard quality" but expensive and slow), and advises
grading the outcome, not the exact path, because rigid step checks
"punish creativity."

**Inference:** the methods that fit our agents differ by role. `engenir`
and `tester` produce outcomes that can be checked by tests (code graders).
`researcher` produces claims that can be checked against sources.
`programmer` (teaching), `writer` and `deneir` produce prose, where only
rubric, pairwise or human judgment exists.

## 2. Rigor: repeated runs, variance, significance

### 2.1 Run-to-run variance is real, and large relative to small effects

- **Documented** (Atil et al., preprint): five LLMs configured to be
  deterministic, eight tasks, 10 runs each. Accuracy varied by up to 15%
  across natural runs; none gave consistently repeatable accuracy on every
  task; identical output strings were rarer still. These are not agent
  tasks, but the point (a "deterministic" setting is not repeatable)
  carries over as a caution. **Inference:** agents with tools add more
  variance.
- **Documented** (Ariño de la Rubia & Pafka, preprint, 2026-09-27): six
  agent/model pairings on one task (improving an XGBoost classifier),
  52 runs each, 584 runs. "Variation between identical runs of one pairing
  was larger than the differences between pairings"; comparing a few runs
  "ranked agents unreliably"; resolving the differences "would take tens
  to over a hundred runs per agent." One task type, one author team, not
  peer-reviewed.
- **Documented** (tau-bench): "even state-of-the-art" agents were "quite
  inconsistent"; pass^8 below 25% in retail for GPT-4o, with at least 3
  trials per task in the main results. In the simulator the user was an
  LLM at temperature 1.0 and the agent at 0.0, so conversations differed
  even with the same task.
- **Documented** (Sclar et al., ICLR 2024): few-shot accuracy differed by
  up to 76 points from small prompt-format changes (LLaMA-2-13B).
  **Inference:** an agent's instruction file is a prompt; editing wording
  while changing model or effort would confound the two.
- **Documented** (Chen, Zaharia & Zou, preprint): GPT-3.5 and GPT-4
  versions from March and June 2023 differed substantially on several
  tasks (e.g. GPT-4 on prime identification fell from 84% to 51%); they
  call for "continuous monitoring." **Inference:** the aliases `sonnet`,
  `opus` point to whatever model Anthropic currently maps them to
  (2026-11-17, section 1); a model update in the middle of an experiment
  would act the same way. The paper studied OpenAI models, not Claude.
- **Documented** (HAL, Kapoor et al., preprint, abstract): "higher
  reasoning effort reduced accuracy in most runs." Read from the abstract
  only; this is the one agent-level finding I found that touches the
  `effort` lever, and it says more effort is not automatically better.

### 2.2 What the statistics literature recommends

- **Report intervals, not point estimates, and prefer robust aggregates.**
  Agarwal et al. (NeurIPS 2021, outstanding paper; deep RL): papers
  "compare point estimates ... and ignore the uncertainty" from few runs;
  recommend interval estimates, performance profiles and the interquartile
  mean. Henderson et al. (AAAI 2018, peer-reviewed): without significance
  metrics and standardized reporting it is hard to tell whether gains over
  prior work are real. **Inference:** the field with the closest
  structure to ours (expensive runs, few seeds) reached the same advice.
- **Treat an eval as an experiment.** Miller (preprint, Anthropic
  author-affiliation not verified on the page) gives five recommendations:
  (1) standard errors of the mean; (2) clustered standard errors when
  questions come in related groups; (3) reduce variance by resampling
  answers and using next-token probabilities; (4) inference on
  question-level **paired differences**; (5) **power analysis** to decide
  whether an eval can test the hypothesis at all. The paired standard
  error is lower when the two configurations' scores are positively
  correlated, and the sample size needed for a minimum detectable effect δ
  is n = (z_{α/2}+z_β)² (ω² + σ_A²/K_A + σ_B²/K_B) / δ², where ω² is the
  variance between questions' true means, σ² the within-question variance
  and K the samples per question (formulas quoted from the paper's
  sections 4.2 and 5).
- **Do not use the CLT on small samples.** Bowyer, Aitchison & Ivanova
  (ICML 2025 spotlight; position paper): CLT intervals "dramatically
  underestimate uncertainty" on small benchmarks (their simulation: a
  nominal 95% interval covered 92.5% at N=100; they tested N=3, 10, 30,
  100 and avoid naming a hard cut-off). Recommended: Wilson or Bayesian
  (Beta-Bernoulli) intervals; for comparing two models, the paired Bayes
  method, or Bayesian intervals on the difference. They found bootstrap
  intervals also performed poorly, and Clopper-Pearson too wide.
- **Underpowered experiments are the norm and distort findings.** Card et
  al. (EMNLP 2020): power "largely ignored" in NLP; typical human-rating
  designs likely underpowered to detect small differences; underpowered
  experiments "raise the risk of exaggerated findings." Dror et al. (ACL
  2018): a survey of 2017 ACL/TACL papers found significance testing often
  ignored or misused; offers a protocol for choosing a test (steps not
  read; see "not found").
- **Many benchmark rankings are unresolved.** Kotawala (preprint, ICML 2026
  workshop on hypothesis testing): 11 of 40 pairwise comparisons on the
  Open LLM Leaderboard v1 did not meet a paired-test power target at
  alpha 0.05, power 0.8.
- **Non-benchmark variance sources are quantified for pretraining, not for
  agents.** Madaan et al. (preprint): practitioners "should account for
  variance before concluding that one model outperforms another."
- **Absence of a difference is not equivalence.** The search surfaced this
  caution in a preprint (AMLworld-Compact) that I did not read; treat as
  unverified. **Not found:** a paper applying non-inferiority or
  equivalence tests (TOST) to choosing a cheaper LLM or agent
  configuration. **Inference:** our real question ("is the cheaper
  configuration *not worse by more than we accept*?") is a
  non-inferiority question, and the superiority-test literature above
  answers a different one.

### 2.3 How many repeated runs

- **There is no general number.** Sources give ranges tied to a setting:
  tau-bench used at least 3 trials per task (and at least 40 for one
  difficulty analysis); Ariño de la Rubia & Pafka say tens to over a
  hundred runs per pairing on one task type when the effect is small;
  Anthropic (secondary) says "20-50 simple tasks drawn from real
  failures is a great start" because early changes have large effects and
  larger suites are needed "to detect smaller effects."
- **The structure of the formula says why.** Miller's n depends on the
  variance **between tasks** (ω²) and the variance **within a task** (σ²).
  More repeated runs per task (K) shrink only the σ²/K terms; they cannot
  shrink ω². **Inference** from the formula: if tasks differ a lot in
  difficulty, repeating one task many times does not rescue the
  comparison; more *different* tasks, or the *same tasks under both
  configurations* (pairing), is what helps.
- **Illustration, computed here (inference, textbook two-proportion
  normal approximation, alpha 0.05, power 0.8, so optimistic given
  Bowyer et al.):** to tell 70% from 85% success you need about 120
  independent tasks per configuration; with 20 tasks per configuration
  only a gap of roughly 44 points is detectable. This is a rough order of
  magnitude, not a design.

## 3. Quantitative metrics in use

| Metric | Definition (as documented) | What it answers | Notes |
| :- | :- | :- | :- |
| **Success / resolve rate (pass@1)** | Share of tasks where the outcome check passes | Does it work, on average | Needs a trustworthy check (Zhu et al.) |
| **pass@k** | `1 - C(n-c,k)/C(n,k)`, estimated from n >= k samples with c correct, averaged over problems; computing it from exactly k samples "can have high variance" (Chen et al., preprint) | Is *at least one* of k attempts correct | Anthropic (secondary): suits cases where one success matters; rises with k |
| **pass^k** | `E_task[C(c,k)/C(n,k)]`, "the chance that all k i.i.d. task trials are successful" (Yao et al., preprint) | Does it succeed *every* time | Falls with k; Anthropic (secondary): example 0.75 per trial gives about 42% at k=3 |
| **Cost per solved task, tokens, price** | Cost as a first-class metric next to accuracy (Kapoor et al., preprint); Anthropic's own docs report "$ per solved task" (2026-11-17, section 3) | Is the cheaper configuration worth it | Our measurement mechanics are in [2026-11-23 cost tracking](2026-11-23-cost-tracking-per-agent.md); its section 0 warns that a worker's reported "tokens" is its last request, not its total |
| **Time / latency** | Wall-clock per task; METR used self-reported time to completion | Speed | METR's self-reports are a stated limitation |
| **Consistency / agreement across runs** | TARr@N and TARa@N, total agreement rate over raw outputs and parsed answers at N runs (Atil et al., preprint) | How repeatable | Defined for fixed-answer tasks |
| **Grader agreement** | Cohen's kappa, Krippendorff alpha, correlation (section 4) | How reliable the judgment is | |
| **Process / trajectory measures** | Instruction following, tool use, verification, recovery (AgentLens, preprint); partial credit "a continuum" (Anthropic, secondary) | Why it succeeded or failed | Mostly qualitative or LLM-reviewed |

**Inference for our setting:** pass@k and pass^k need several attempts at
the *same* task. In our work tasks do not repeat, so per-task pass^k is
not computable from ordinary use; it becomes computable only if a spec is
deliberately re-run (a replay). That is a design choice, not something
the literature settles for us.

**Inference, to the person's existing signals** (2026-11-17, section 5):
"failures `tester` catches in `engenir`'s work" is a code-graded outcome
metric; "re-delegations" and "corrections the person has to make" are
outcome proxies, graded by a human who knows the configuration used;
"`researcher` claims later found wrong" is a delayed outcome check. All
four are consistent with the grader taxonomy above; none has a documented
reliability in the literature I read.

## 4. Qualitative metrics: making judgment reliable

### 4.1 People as judges

- **Documented:** human judgment is not automatically reliable. Human
  evaluation is "notoriously unstable" (Chiang & Lee, ACL 2023). Humans
  and LLM judges both showed misinformation-oversight, gender, authority
  and beauty biases (Chen et al., EMNLP 2024 per arXiv comments). Untrained
  raters were at chance distinguishing GPT-3 from human text and short
  training gave no statistically significant gain (Clark et al., preprint).
- **Documented, directly relevant to a single person:** "criteria drift":
  users need criteria to grade outputs, but grading outputs also changes
  their criteria; some criteria depend on which outputs the user has seen
  (Shankar et al., arXiv; the page read lists no venue). **Inference:**
  one person grading over weeks will drift unless the rubric is written,
  frozen and re-checked.
- **Documented, practice guidance (secondary):** OpenAI's guide: pairwise
  comparison or pass/fail "for more reliability"; examples of each score
  level; consensus votes across reviewers; run several review rounds to
  refine the scorecard. Anthropic's docs: criteria should be specific,
  measurable, achievable, relevant; define the scale endpoints;
  "More questions with slightly lower signal automated grading is better
  than fewer questions with high-quality human hand-graded evals."
- Best-practice papers for human evaluation of generated text exist (van
  der Lee et al., *Computer Speech & Language* 2021, peer-reviewed;
  confirmed through bibliographic records only). **Not read; recommendations
  not extracted.**

### 4.2 Agreement statistics

- **Documented (secondary, search summaries):** Cohen's kappa and
  Krippendorff's alpha are the usual agreement coefficients; alpha handles
  several raters, missing values and non-nominal scales; the commonly
  quoted kappa bands (0.61-0.80 "substantial", Landis & Koch 1977) are a
  convention without empirical support, and different coefficients can
  give different values on the same data. **The primary papers (Landis &
  Koch 1977; McHugh 2012) could not be fetched (the host blocked
  automated access); these statements are second-hand.**
- **Documented** (Thakur et al., GEM workshop at ACL 2025,
  peer-reviewed workshop): "alignment metrics beyond simple percent
  alignment" are needed, because judges with high percent agreement can
  still assign very different scores.
- **Single-rater case, documented in a search summary only:** intra-rater
  reliability (the same rater, or judge, agreeing with itself over
  repeated gradings) is the usable statistic when there is one human. The
  paper "Rating Roulette" (Haldar & Hockenmaier, EMNLP 2025) reports LLM
  judges show "low intra-rater reliability in their assigned scores across
  different runs" (abstract read; statistics and numbers not read). A
  claim in a search summary about humans varying within the same rater over
  two sessions was not traced to a source I verified: **not verified.**

### 4.3 LLM-as-judge: validity and known biases

| Finding | Source |
| :- | :- |
| Position, verbosity, self-enhancement bias; limited reasoning; >80% agreement with human preferences | Zheng et al., NeurIPS 2023 D&B (peer-reviewed) |
| Rankings "can be easily hacked" by response order; Vicuna-13B beat ChatGPT in 66 of 80 queries when ChatGPT judged; mitigations: several pieces of evidence before the rating, balanced position (aggregate over orders), human-in-the-loop on hard cases | Wang et al., preprint |
| Position bias is not random chance; varies by judge and task; strongly affected by the quality gap between the answers; 15 judges, over 150,000 instances | Shi et al., AACL-IJCNLP 2025 (peer-reviewed) |
| Judges recognize and favor their own generations; self-preference grows linearly with self-recognition after fine-tuning (GPT-4 and Llama 2 tested) | Panickssery et al., preprint |
| 12 biases quantified with the CALM framework; "significant biases persist in certain specific tasks" | Ye et al., preprint |
| Best and largest judges reach only "reasonable alignment" and fall short of inter-human agreement; scores can differ by up to 5 points; judges lenient; sensitive to prompt complexity and length; for *ranking* models even small judges and a lexical metric gave a reasonable signal | Thakur et al., GEM 2025 |
| Low self-consistency across runs of the same judge | Haldar & Hockenmaier, EMNLP 2025 (abstract only) |
| LLM-based evaluators may favor LLM-generated text (preliminary); Spearman 0.514 with humans on summarization | Liu et al. (G-Eval), preprint |
| Rubric-conditioned evaluator: Pearson 0.897 with humans on 45 custom rubrics (GPT-4: 0.882) | Kim et al. (Prometheus), ICLR 2024 |
| Calibrating rubric answers to each human judge's labels gave RMS error below 0.5 on a 1-4 scale, about 2x better than uncalibrated; "humans disagree with each other" | Hashemi et al. (LLM-Rubric), ACL 2024 |
| In software engineering annotation tasks, six LLMs reached agreement "equal or close to" human-rater agreement on some of ten tasks; model-model agreement suggested as a predictor of which tasks LLMs can annotate, with model confidence for picking samples | Ahmed et al., preprint (arXiv page lists no venue; a search result said MSR 2025, not confirmed) |
| Survey: reliability of LLM-as-a-judge "remains a significant challenge" | Gu et al., preprint |

The correlation figures come from different tasks and metrics (Spearman,
Pearson, percent agreement, RMS error) and cannot be compared with each
other.

**Documented practice (secondary):** Anthropic's agent-evals post: LLM
graders "should be closely calibrated with human experts"; give the judge
an exit ("Unknown"); use structured rubrics; grade each dimension with a
separate isolated judge; spot-check with humans once robust. Anthropic's
platform docs: "evaluate with a different model from the one used to
generate the evaluated output." OpenAI's guide: control for response
length to counter verbosity bias; ask for reasoning before the score; scale
the judge only once it "consistently agrees with human annotations."

**Inference for us:** every agent here is a Claude model, and judging
with a Claude model means judge and judged may share a family. Self-
preference was shown for GPT-4 and Llama 2, not for Claude; I did not
find a Claude-specific measurement. Using a different model family, or
judging pairwise with both orders, are the mitigations the sources name.

## 5. Where our context sits, compared with the others

### 5.1 The contexts the literature describes

| # | Context | Typical size and set-up | Sources |
| :- | :- | :- | :- |
| A | **Large public benchmark studies** | Hundreds to thousands of fixed tasks, automated graders, many models, single or few runs per task, leaderboard goal | SWE-bench (2,294), AgentBench (8 environments), GAIA (466) |
| B | **Infrastructure-scale multi-run studies** | Tens of thousands of rollouts; many repeats per configuration | HAL: 21,730 rollouts, 9 models, 9 benchmarks, about $40,000; Ariño de la Rubia & Pafka: 584 runs, one task type |
| C | **Industry eval practice (developers' own documentation)** | Small suites (20-50 tasks) built from real failures, grown over time; capability evals plus regression evals; continuous; production monitoring, A/B tests, transcript reading | Anthropic, OpenAI docs (secondary) |
| D | **Statistics-methodology papers** | Not a setting but rules for the others: intervals, power, pairing, small-N behavior | Miller, Bowyer et al., Card et al., Dror et al., Agarwal et al., Henderson et al., Kotawala |
| E | **Human-subject randomized trials with practitioners** | Few people, tasks randomized to condition, real work, self-reported outcomes | METR: 16 developers, 246 tasks of about 2 h, per-task randomization, not blinded |
| F | **Single-case / small-n experimental designs** (other fields; some software engineering precedent) | One participant (or a few) as own control; repeated measurement; replication within and across participants | Dallery et al., JMIR 2013 (health); Zendler et al., *Information and Software Technology* 2001 (software engineering; bibliographic record only, abstract not read); What Works Clearinghouse standards (secondary quotation) |
| G | **Human evaluation of generated text** | Crowd or expert raters, many items, agreement statistics | Chiang & Lee, Clark et al., van der Lee et al. |
| H | **LLM-judge validation studies** | Large sets with human labels or known answers | Zheng et al., Thakur et al., Shi et al., Ahmed et al. |

### 5.2 Our setting (stated, then placed)

- One person is the sole user, task author, and rater. A few projects.
- Tasks never repeat identically; difficulty varies a lot (specs differ).
- Treatment = an agent's `model` and `effort` (and the prompt in its
  file, which may change too).
- Outcomes today: tester failures, the person's corrections,
  re-delegations, researcher claims later found wrong (2026-11-17, s.5).
- Subscription plan: usage is approximate and shared across models; cost
  metrics are weaker than in API-billed studies (see the cost-tracking
  report).
- The person knows which configuration produced each output (not blinded).

**Placement (inference):** we are not in A, B, G or H on size. We resemble
**C** in scale and spirit (small, real-failure-driven, iterative) and
**E** in structure (one practitioner's real tasks; randomization of
configuration per task is possible; self-report outcomes; not blinded). We
resemble **F** in the number of units (one participant), with a
difference: F's strength comes from *repeated measurement of the same
outcome under controlled alternation*, while our tasks are unlike each
other. D applies to all of them as a set of warnings.

### 5.3 Comparison: what transfers and what does not

| Feature | Large benchmarks (A) / multi-run (B) | Industry practice (C) | METR-type trial (E) | Single-case (F) | **Ours** | Transfers? Why |
| :- | :- | :- | :- | :- | :- | :- |
| Number of independent tasks | Hundreds to thousands | 20-50 to start | 246 (16 people) | Many repeated measurements of one unit | Tens at most over weeks (inference) | The size-dependent statistics (CLT, leaderboard ranking) do **not** transfer: Bowyer et al. show CLT failing in this range. What transfers is the warning that only large differences can be detected |
| Same tasks under both configurations | Yes, fixed task sets | Yes, fixed suites | No: each task gets one condition, randomized | Conditions alternate over time | Not by default | Pairing (Miller; Bowyer) would help most, but needs replays of the same spec. A choice, not a given |
| Grader | Automated, trusted after audit | Mixed; read transcripts | Self-reported time | Operationally defined measure | Partly automated (tester), partly the person | Auditing graders by reading transcripts transfers fully (C). Zhu et al. show even published graders had flaws |
| Blinding | n/a (automated) | n/a | Not blinded (stated limitation) | Experimenter bias a listed limitation | Not blinded | Transfers as a risk; METR's own result shows beliefs and measurement can disagree in sign |
| Run-to-run variance | Often one run; HAL, Ariño and tau-bench show it matters | Multiple trials advised | Averaged over tasks | Built into the design (baseline stability) | Unknown, not measured | Transfers directly: each source warns a single run is a weak basis. Size of the variance for *our* agents is **not found** |
| Task heterogeneity | Controlled by sampling many tasks | Curated | Controlled with difficulty forecasts in the regression | Not an issue (same unit, same task) | High | Partial: METR adjusted with the developers' difficulty forecasts, which suggests recording a difficulty estimate (inference) per task |
| Generalization | Claimed across tasks | Own product | Explicitly cautioned: "We caution readers against overgeneralizing" | Needs direct and systematic replication (Dallery et al.) | Local to this person and these projects | Do not claim more than local validity; same caution as E and F |
| Data leakage / contamination | Serious concern (Sallou et al.) | Less | Own repositories | n/a | Private, new tasks | **Advantage** (inference): our tasks cannot be in training data in the way public benchmark tasks can |
| Model version drift | Models pinned in papers | Pinned or tracked | Early-2025 tools | n/a | Aliases move | Transfers (Chen et al.); record the exact model identifier for each run (inference) |
| Cost | Measured, often decisive (Kapoor et al.) | Tracked | Not the outcome | n/a | Approximate on subscription | Transfers as a metric; measurement limits are in the cost report |
| Number of raters | Many; agreement statistics | Experts; spot-checks | Developers rate their own time | The experimenter observes | One | Inter-rater agreement is **not computable** with one human; only intra-rater (same person at two times) and person-versus-judge agreement are |

**What transfers (documented basis, with our inference where marked):**
1. Report intervals, not single numbers (Agarwal; Miller; Bowyer).
2. Use Wilson or Bayesian intervals, not CLT, at small N (Bowyer).
3. Randomize the configuration per task if the design compares
   configurations (METR; the alternating-treatment design in Dallery et
   al.).
4. Ground outcomes in a deterministic check where one exists, and read
   transcripts to audit it (Anthropic, secondary; Zhu et al.).
5. Start from real failures and a small set; expect only large effects
   to be visible (Anthropic, secondary).
6. Do not trust the person's impression alone (METR).
7. Freeze the rubric before grading, because criteria drift (Shankar et
   al.; inference for the freezing).
8. Treat cost as a metric alongside quality (Kapoor et al.).

**What does not transfer, and why:**
1. Leaderboard-style absolute scores and rankings: they need hundreds of
   tasks and fixed task sets (Kotawala; Card).
2. Large-sample power calculations as a plan: the numbers in section 2.3
   show our task counts would only detect big gaps.
3. Inter-rater agreement among several humans: we have one rater.
4. The pre/post two-condition comparison as proof: Dallery et al.: "Using
   only two conditions, such as a pre-post design, is not considered
   sufficient to demonstrate experimental control"; a change of configuration
   followed by "it seemed fine" is that weaker form.
5. Standards from other fields as rules: the WWC single-case standards
   ask for at least three demonstrations of an effect and Dallery et al.
   suggest at least five data points per phase; these come from behavioral
   and health research with repeated, homogeneous measures, and our tasks
   are not that.

**Limits of this comparison (stated plainly):** F is a precedent from
other fields; the only software-engineering single-case paper I located
(Zendler et al., 2001) was seen as a bibliographic record without its
abstract. I found no paper applying single-case or N-of-1 designs to
choosing LLM agent configurations. Placing ours near C, E and F is
inference.

## 6. Cautions on the evidence itself

- Most agent-specific papers cited here are preprints; the sample-size
  claims for coding agents rest on one author team's single task
  (Ariño de la Rubia & Pafka).
- Anthropic and OpenAI documents are developer guidance, not studies; they
  cite no numbers for their "20-50 tasks" advice.
- A search-result summary of the Ariño paper stated different details
  (SWE-bench Verified, ten runs, "9 runs for a 2-point gain") from the
  abstract page I fetched (XGBoost task, 52 runs per pairing, no power
  figures in the abstract). I used the fetched abstract and discarded the
  summary's numbers. The same summary attributed single-run spreads of
  2.2-6.0 points to a long monograph (Jarmak, arXiv 2608.13867); its
  abstract reports no such figure, so that number is **unverified and not
  used**.
- The HAL "effort" statement and the Haldar & Hockenmaier statements come
  from abstracts only.

## 7. Options for a method (a proposal for the person to decide, not a design)

Each option names which literature it rests on and what it cannot show.

| Option | Idea | Rests on | Can show | Cannot show |
| :- | :- | :- | :- | :- |
| **A. Observe-only with intervals** | Keep today's signals, record the exact model and effort per task, and report each signal's rate with a Wilson or Bayesian interval per configuration | Bowyer; Agarwal; Miller | A rough rate and how uncertain it is | Cause: tasks and configurations change together |
| **B. Randomized alternation** | When delegating, pick the configuration at random for each task (A or B) and record outcome signals plus a difficulty estimate | METR; alternating-treatment design (Dallery) | An unbiased comparison over time, large effects only | Small effects; needs weeks of tasks |
| **C. Replay of frozen specs** | Re-run a small set of closed specs under both configurations (paired), and several times each | Miller (pairing, power); tau-bench (pass^k); Ariño & Pafka (variance) | Run-to-run variance for *our* agents, a paired difference, pass^k for a spec | Costs more plan usage; replays of specs may not represent future work |
| **D. Add an LLM or person-graded rubric on prose roles** | Pairwise judgment (both orders) or a frozen rubric for `programmer`, `writer`, `deneir`; person re-grades a sample later to estimate self-agreement | Zheng; Wang; Shankar; Haldar & Hockenmaier | A repeatable measure where no test exists | Ground truth; judge bias unless checked |
| **E. Read-the-transcripts first** | Before any numbers, read a sample of runs from each configuration and write down failure types | Anthropic (secondary); Zhu et al. | What fails and whether the checks are fair | Rates |

These can be combined (C for variance and for `engenir`/`tester`; D for
prose roles). Choosing among them, and the margin of loss the person is
willing to accept in exchange for lower cost, is the person's decision.
The sources give no value for that margin: **not found**.

## Sources checked (2026-11-23)

Peer-reviewed (venue stated on the page read):
- Jimenez et al., SWE-bench, ICLR 2024 — https://arxiv.org/abs/2310.06770
- Liu et al., AgentBench, ICLR 2024 — https://arxiv.org/abs/2308.03688
- Zheng et al., Judging LLM-as-a-Judge, NeurIPS 2023 D&B — https://arxiv.org/abs/2306.05685
- Bowyer et al., Don't use the CLT..., ICML 2025 spotlight — https://arxiv.org/abs/2503.01747 (and HTML version)
- Agarwal et al., Deep RL at the edge of the statistical precipice, NeurIPS 2021 (per arXiv page) — https://arxiv.org/abs/2108.13264
- Henderson et al., Deep RL that matters, AAAI 2018 — https://arxiv.org/abs/1709.06560
- Dror et al., ACL 2018 — https://aclanthology.org/P18-1128/
- Card et al., EMNLP 2020 — https://aclanthology.org/2020.emnlp-main.745/
- Sclar et al., ICLR 2024 — https://arxiv.org/abs/2310.11324
- Chiang & Lee, ACL 2023 — https://arxiv.org/abs/2305.01937
- Chen et al., Humans or LLMs as the judge, EMNLP 2024 (per arXiv comments) — https://arxiv.org/abs/2402.10669
- Kim et al., Prometheus, ICLR 2024 — https://arxiv.org/abs/2310.08491
- Hashemi et al., LLM-Rubric, ACL 2024 — https://arxiv.org/abs/2501.00274
- Thakur et al., Judging the Judges, GEM workshop 2025 — https://arxiv.org/abs/2406.12624
- Shi et al., position bias, AACL-IJCNLP 2025 (per arXiv comments) — https://arxiv.org/abs/2406.07791
- Haldar & Hockenmaier, Rating Roulette, EMNLP 2025 (abstract only) — https://arxiv.org/abs/2510.27106
- Sallou et al., ICSE 2024 NIER — https://arxiv.org/abs/2312.08055
- Dallery et al., JMIR 2013 — https://pmc.ncbi.nlm.nih.gov/articles/PMC3636286/ (first 100K characters)
- Zendler et al., *Information and Software Technology* 43(12), 2001, bibliographic record only — https://api.crossref.org/works/10.1016%2FS0950-5849%2801%2900177-X
- van der Lee et al., *Computer Speech & Language* 67, 2021, bibliographic record only (via search results)

Preprints (arXiv page lists no venue, or venue not confirmed):
- Yao et al., tau-bench — https://arxiv.org/abs/2406.12045 (HTML read for the pass^k definition)
- Chen et al., Codex / HumanEval — https://arxiv.org/abs/2107.03374 (ar5iv HTML read for pass@k)
- Kapoor et al., AI Agents That Matter — https://arxiv.org/abs/2407.01502
- Kapoor et al., Holistic Agent Leaderboard (abstract) — https://arxiv.org/abs/2510.11977
- Miller, Adding Error Bars to Evals — https://arxiv.org/abs/2411.00640 (HTML read for the recommendations and formulas)
- Becker et al., METR trial — https://arxiv.org/abs/2507.09089 (HTML, first 100K characters)
- Zhu et al., Agentic Benchmark Checklist — https://arxiv.org/abs/2507.02825
- Mialon et al., GAIA — https://arxiv.org/abs/2311.12983
- Zhuge et al., Agent-as-a-Judge — https://arxiv.org/abs/2410.10934
- Yehudai et al., Survey on evaluation of LLM-based agents — https://arxiv.org/abs/2503.16416
- Atil et al., Non-determinism — https://arxiv.org/abs/2408.04667
- Madaan et al., Quantifying variance — https://arxiv.org/abs/2406.10229
- Ariño de la Rubia & Pafka, Identical runs, different results — https://arxiv.org/abs/2609.33812 (abstract)
- AgentLens — https://arxiv.org/abs/2607.06624 (abstract)
- Kotawala, Resolution diagnostics for paired LLM evaluation — https://arxiv.org/abs/2605.30315 (abstract)
- Chen, Zaharia & Zou, ChatGPT behavior over time — https://arxiv.org/abs/2307.09009
- Wang et al., Not fair evaluators — https://arxiv.org/abs/2305.17926
- Panickssery et al., Self-preference — https://arxiv.org/abs/2404.13076
- Ye et al., Justice or Prejudice — https://arxiv.org/abs/2410.02736
- Liu et al., G-Eval — https://arxiv.org/abs/2303.16634
- Gu et al., Survey on LLM-as-a-Judge — https://arxiv.org/abs/2411.15594
- Shankar et al., Who validates the validators — https://arxiv.org/abs/2404.12272
- Clark et al., All that's human is not gold — https://arxiv.org/abs/2107.00061
- Ahmed et al., Can LLMs replace manual annotation... — https://arxiv.org/abs/2408.05534
- Bowman & Dahl, What will it take to fix benchmarking — https://arxiv.org/abs/2104.02145 (abstract only; four criteria not read)

Secondary (developer documentation or search summaries):
- Anthropic Engineering, Demystifying evals for AI agents (Jan 9 2026) — https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
- Anthropic platform docs, Define success criteria and build evaluations — https://platform.claude.com/docs/en/test-and-evaluate/develop-tests (first 100K characters)
- OpenAI, Evaluation best practices — https://developers.openai.com/api/docs/guides/evaluation-best-practices
- Search summaries only, not verified at the primary source: Landis & Koch 1977 kappa bands; Krippendorff's alpha properties; WWC single-case standards (Kratochwill et al. 2010, "at least three attempts"); Empirical Standards for Software Engineering Research (Ralph et al., arXiv 2010.03525; seen in search results only, content not read)

Fetches that failed and could not be recovered after retry: McHugh 2012
(PMC and Europe PMC blocked), the WWC single-case PDF (binary, unreadable
in two attempts), the PMC standards table, the IOS Press and SAGE pages
for a 2019 single-case developer study (HTTP 403; not used). Redone after
the connection change: McHugh, the WWC PDF, Rating Roulette, the Kotawala
paper, and the SAGE page (the first two and the SAGE page failed again;
the other two succeeded).

Not read: full text of Dror et al. (the protocol steps), van der Lee et
al., Bowman & Dahl's four criteria, Ralph et al., Chatbot Arena
internals, HELM, SWE-bench Verified's audit, the monograph by Jarmak.

## Confidence

- **Confirmed (checked at the paper's page or text):** the definitions of
  pass@k and pass^k; Miller's five recommendations and formulas; Bowyer et
  al.'s recommendations and the 92.5% at N=100 figure; the METR design and
  headline numbers; the bias findings listed in 4.3 (at abstract level);
  the benchmark facts in section 1; Anthropic's and OpenAI's guidance as
  quoted.
- **Likely but not certain:** that variance between identical agent runs
  is large relative to the differences we would care about (Ariño & Pafka,
  tau-bench, Atil) — each is one setting, none is a Claude Code
  multi-agent harness.
- **Inference (ours):** the placement of our context among C, E and F;
  the transfer table; the sample-size illustration in 2.3; that our
  question is a non-inferiority question; that Claude judging Claude
  carries self-preference risk; that per-task pass^k needs replays.
- **Genuinely unsettled / not found:** a standard number of repeated runs;
  a method for single-user agent configuration choice; non-inferiority
  margins for LLM agents; Claude-specific judge bias measurements; run-to-run
  variance of our own agents; intra-rater reliability of a single person
  grading agent output (only a search-summary claim, unverified); the
  quality effect of `effort` for non-coding roles.
- Fast-moving: several sources are 2026 preprints not yet peer-reviewed.
  Re-check before building on them.
