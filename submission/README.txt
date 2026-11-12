Reasoning Agent - AI Assignment
Arushi Gandhi (M24MA2001)

Files
-----
reasoning_agent.ipynb  self-contained implementation, experiments and traces.
report.pdf             representation, algorithms, measured results and limitations.
submission.csv         id,label predictions; labels are True or False.
README.txt             run instructions.

Requirements
------------
Python 3.10 or newer (tested with Python 3.12), plus:

    pip install pandas datasets huggingface_hub jupyter

The reasoning engines use only the Python standard library. No GPU, pretrained
model, generative model or external theorem prover is used for predictions.

Run
---
Put the test file next to the notebook as test.csv, or set:

    export TEST_PATH=/path/to/test.csv

Run all cells, or execute:

    jupyter nbconvert --to notebook --execute --inplace reasoning_agent.ipynb

The notebook prints public accuracy, reasoning time, diagnostics and traces, then
writes submission.csv. If no test file exists, it writes public evaluation
predictions. The first public run may download about 300 MB from Hugging Face.
Cached Arrow files and the project's local data files are used when available.

To skip the public experiments when predicting a supplied file:

    export RUN_PUBLIC=0

The input file can be CSV, JSON or JSONL. The loader recognises common id,
premises and query column names, including premises-FOL and conclusion-FOL.
Premises may be a list, a Python list string, or formulas separated by newlines
or semicolons. Missing columns produce a clear error.

Semantics and parsing
---------------------
Formal input defaults to classical entailment: an absent fact is not false.
Identifiers preserve case, digits, underscores and hyphens. Ready1 and Ready2
are different predicates. Malformed input is diagnosed, not partially accepted.

If evaluation explicitly uses closed-world rules, set:

    export REASONING_SEMANTICS=cwa

A per-row semantics column overrides that default; accepted values include
classical/owa and closed/cwa. This is a semantic choice, not a speed setting.
The public RuleTaker adapter explicitly selects closed-world semantics.

The optional parsing_profile column accepts formal (default) or annotations.
annotations repairs bracket imbalance, folds case and merges plural/tense
predicate variants. It is used for noisy public FOLIO annotations, and can
change formal meaning. Do not select it for clean formal evaluation input.

Syntax
------
Quantifiers:  forall x.  exists x.  Unicode universal and existential symbols.
Operators:    not ~ !, and &, or |, -> =>, iff <->, xor; Unicode equivalents.
Equality:     a = b, a != b.
Prolog:       successful(X) :- smart(X), kind(X).
Free x,y,z,u,v,w and single capitals are universally closed. Prolog supports
longer capitalised variables too. Lowercase names such as alice are constants.

Counterfactual example:
    B := A
    C := B
    query: do(~A) | A => C

Python API (defined in the notebook)
-----------------------------------
    solve(premises, query)
    solve(premises, query, mode="cwa")
    compiled = compile_kb(premises, semantics="classical")
    compiled.answer_many(queries)

Results contain label, method, status, trace and diagnostics. PROVED is True;
DISPROVED is a completed negative result. UNKNOWN, LIMIT_REACHED, PARSE_ERROR
and UNSUPPORTED remain visible internally and map to False in the required CSV.
The first-order fallback is bounded and incomplete; a timeout is not a
countermodel. Set trace=True to inspect the reasoning.

Evaluation
----------
The notebook evaluates RuleTaker, FOLIO and CounterBench and prints measured
accuracy and inference time. The report records the public baseline comparison,
runtime tradeoffs and examples of reasoning traces. The included CSV contains
public evaluation predictions; a supplied input file is processed by the loader.

Speed revision
--------------
A three-run public benchmark reduces median inference from 10.765704451 s to
8.329301685 s (22.63%) with every prediction unchanged. This is a measurement on
the same laptop and workloads, not a speed guarantee for every query. Search
budgets and semantics are unchanged. Reused backward answers, resolution indexes
and cached term depths reduce overhead. A checked finite countermodel can end
an unsuccessful proof search early; model-search failure falls through to the
existing engines. The original and optimised versions are archived separately.

Learned advisor experiments
---------------------------
The notebook embeds a small fitted structural router and a 12-feature clause
scorer. They guide engine order or proof search; they never predict answer
labels. No pretrained language model or extra inference dependency is needed.

Use solve(..., guidance="router"), "ranker", or "learned" to opt in.
predict_file(path, guidance="router") also supports an explicit experiment.
The default "auto" currently leaves advisors off: the development ablation
showed no consistent validation speed benefit, and labels were unchanged.

Successful learned refutations pass a separate derivation checker. An
unsuccessful guided probe keeps the original fallback stages and budgets.
Resolution(verify=True) also audits an ordinary resolution run. Models cannot
choose semantics, vote on labels or turn an unknown result into a proof.

Training used FOLIO train and generated development problems, with story/template
groups separated internally. Targets were proof-clause participation and solver
runtimes, not gold entailment labels. Reproduction scripts and trace data are
in workspace/learned_guidance and workspace/train_guidance.py. The report
includes the measured ablation and explains why the advisors remain optional.
