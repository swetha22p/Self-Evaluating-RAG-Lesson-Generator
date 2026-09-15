# Self-Evaluating RAG Lesson Generator

An n8n workflow that generates a beginner-friendly lesson about
Retrieval-Augmented Generation (RAG), evaluates it against hard
pass/fail checks, and attempts to revise it when it does not pass.

## Project goal

Generate a clear lesson about **Introduction to RAG** for a learner who
is a 12th-grade graduate in India, has limited English vocabulary, and
has no technical background.

The workflow follows this cycle:

**Generate → Evaluate → Record failure → Revise → Evaluate again**

It also stores failure records and retrieves topic-related past failures
to guide later generations.

## Workflow overview

1.  **Manual Trigger** --- starts the workflow.
2.  **Edit Fields** --- sets the topic, learner profile, maximum
    retries, and run ID.
3.  **Read Memory** --- retrieves previous failure records for the topic
    from the `lesson_failures` Data Table.
4.  **Planner Agent** --- creates a structured lesson plan.
5.  **Generator Agent** --- writes the lesson using the plan, learner
    profile, and available failure memory.
6.  **Current Lesson** --- holds the lesson currently being evaluated.
7.  **Evaluator Agent** --- checks the lesson and returns a structured
    PASS/FAIL decision with check results and feedback.
8.  **Check Evaluation** --- routes the workflow based on the
    evaluator's decision.
9.  **PASS path** --- reads failure records for the current run and
    returns the final lesson, status, retry count, and rejection log.
10. **FAIL path** --- saves the failed lesson and evaluation feedback,
    counts failures for the current run, and checks the retry limit.
11. **Revision Agent** --- if another attempt is allowed, revises the
    lesson using the evaluator's feedback. The revised lesson is
    evaluated again.
12. **Give Up Output** --- returns a failure result when the retry limit
    has been reached.

## Agents

  -----------------------------------------------------------------------
  Agent                               Responsibility
  ----------------------------------- -----------------------------------
  Planner Agent                       Creates a structured plan for the
                                      lesson.

  Generator Agent                     Generates the complete lesson from
                                      the plan and learner profile.

  Evaluator Agent                     Applies the configured hard checks
                                      and returns PASS/FAIL with reasons
                                      and corrections.

  Revision Agent                      Revises a failed lesson using the
                                      evaluator's feedback.
  -----------------------------------------------------------------------

## Evaluation

The Evaluator Agent uses six hard checks configured in the workflow. The
checks are intended to assess:

-   Accuracy and grounding
-   Beginner-friendly language
-   Use of a familiar explanatory example
-   Explanation of technical terms
-   Coverage of the key RAG concepts
-   Clear teaching flow

The lesson is accepted only when the evaluator returns PASS. A PASS
means it passed the checks configured in this workflow; it is not a
guarantee that every possible error has been detected.

## Memory and retry behavior

The workflow stores rejected attempts in the `lesson_failures` Data
Table. The exported workflow uses these fields:

-   `topic`
-   `lesson`
-   `failure_reason`
-   `failed_checks`
-   `run_id`

The workflow retrieves a limited number of previous failure records for
the same topic and supplies them to the planning and generation prompts.

The configured maximum is **two retries**. This allows the initial
generation and up to two revisions. If the lesson still fails after the
allowed revisions, the workflow returns the give-up result.

This is **failure-informed generation with memory and retry-based
revision**. It does not automatically rewrite its own code or evaluation
rubric.

## Requirements

-   An n8n account or instance
-   An OpenRouter API account and API key
-   The workflow JSON file included in this repository
-   A Data Table named `lesson_failures` with the expected columns

> The exported workflow's OpenRouter Chat Model nodes specify
> `openai/gpt-4o-mini`. If you intend to use a different model, update
> the model configuration in the relevant nodes and test the workflow
> again.

## Setup

1.  Import the workflow JSON into your n8n instance.
2.  Configure your OpenRouter credential in n8n. Do not commit API keys
    or other secrets to GitHub.
3.  Create or verify the `lesson_failures` Data Table. Ensure it
    contains the fields listed above, especially `run_id`.
4.  Open the workflow and confirm that the model and credential
    references are valid in your n8n instance.
5.  Save the workflow and run a test execution.

The exported JSON contains workflow configuration, but credentials and
Data Table contents may not transfer to another n8n account. Configure
those resources in the destination instance.

## How to run

1.  Open the imported workflow in n8n.
2.  Set the topic and learner profile in **Edit Fields**, if you want to
    change the defaults.
3.  Click **Execute workflow**.
4.  Inspect the Generator and Evaluator outputs.
5.  If evaluation fails, inspect the saved failure row and the Revision
    Agent output.
6.  Inspect the final output for the lesson status, retries used, and
    rejection log.

## Testing checklist

Before considering the workflow validated, test and record the actual
results for:

-   [ ] **PASS path:** a lesson passes evaluation and reaches Final
    Output.
-   [ ] **Revision path:** a failed lesson is logged, revised, and
    evaluated again.
-   [ ] **Retry limit:** repeated failure reaches Give Up Output after
    the configured attempts.
-   [ ] **Memory retrieval:** after a failure is stored, a later run for
    the same topic retrieves that failure information.
-   [ ] **Fresh import:** the workflow works after credentials and the
    Data Table are configured in the target n8n instance.

Do not report a test as successful unless you have run it and observed
the result.

## Repository contents

-   `README.md` --- project overview, workflow explanation, setup, and
    testing instructions
-   `workflow.json` --- exported n8n workflow
-   `Self_Evaluating_RAG_Lesson_Generator_Submission.docx` --- project
    documentation (if included)
-   `architecture.png` --- workflow architecture diagram (if included)

Update the filenames above to match the exact files in your repository.

## Limitations

-   The lesson quality depends on the language model and the evaluator
    prompts.
-   The evaluator may miss errors; a PASS is not a guarantee of
    correctness.
-   Memory is based on stored failure records retrieved by topic; the
    workflow does not use embedding-based semantic search.
-   The workflow uses bounded retries and may return a failed result if
    the lesson does not pass.
-   Credentials and Data Table setup may need to be recreated when
    importing into another n8n instance.

## Demo

The project demo should show the workflow, a real execution, the
evaluator's PASS/FAIL output, the failure log, the revision loop (if
triggered), and the final result.

## Author

**Swetha P**
