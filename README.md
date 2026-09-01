# Customer Support Chatbot

## Project Overview

This project implements a customer support chatbot for a fictional online shop using an **Amazon Bedrock AgentCore managed harness**.

The chatbot routes every customer message into one of three support paths:

1. **Bug Report** — collects the required bug information and creates a support ticket.
2. **Platform Question** — answers questions that are covered by the provided FAQ.
3. **Other Request** — redirects requests outside the supported scope to the human support line.

The system prompt defines the routing and conversation behavior. The bug-report path uses an **AgentCore Gateway** to invoke the bug-report tool, which persists completed reports in Amazon DynamoDB.

---

## 1. Bug Report Path

The bug-report path is defined directly in `system_prompt.txt`.

When a customer reports a bug, the assistant collects the required information before creating a ticket:

* **Bug description**
* **Steps to reproduce**
* **Environment information**, such as browser, operating system, or device

The assistant is instructed to ask follow-up questions when information is missing and only invoke the bug-report tool once the required information has been collected.

The completed bug report is sent through the **AgentCore Gateway** to the bug-report tool. The resulting ticket is persisted in the following DynamoDB table:

`bug-report-tool-stack-bug-reports`

### Example Persisted Ticket

A successful test created a DynamoDB record containing:

* **Ticket ID:** `e56999c5-12ca-4dfe-84fb-f50020ba2f76`
* **Description:** `When I click checkout, nothing happens.`
* **Steps to reproduce:** `Click the checkout button.`
* **Environment:** `Windows 11, Chrome 139.`
* **Status:** `OPEN`

Screenshots are included as evidence of the multi-turn bug-report conversation, tool invocation, and DynamoDB record creation.

---

## 2. Platform Question Path

Platform questions are answered using the provided `online_shop_faq.md` content.

The chatbot was tested with questions that are covered by the FAQ, including:

* Guest checkout
* Shipping and delivery
* Order tracking
* Returns
* Payment methods

For questions that can be answered from the FAQ, the assistant provides the relevant information without inventing unsupported details.

The submission includes a screenshot demonstrating a successful covered FAQ question.

### Uncovered Questions

The chatbot was also tested with questions that require information not provided by the FAQ.

For example, a customer asking for an exact delivery time to a specific location cannot be given an invented estimate. In these cases, the assistant directs the customer to the human support line.

A screenshot of an uncovered question and the resulting support redirection is included as evidence.

---

## 3. Other Request Path

The chatbot has a separate path for customer requests that are outside the supported online-shop FAQ and bug-report functionality.

Examples include:

* Asking for programming assistance
* Asking for product recommendations outside the defined support scope
* Other general requests unrelated to the supported customer-service functions

These requests are redirected to the human support line:

**1-800-555-0199 (Mon–Fri)**

A screenshot demonstrating the Other Request path is included in the submission.

---

## 4. Automated Flow Testing

The automated test suite is provided in:

`flow-tests.json`

The test suite covers all three required paths.

### Bug Report

Tests include:

* Complete bug reports
* Missing reproduction steps
* Missing environment information
* Multi-turn information collection
* Multiple fields provided in one message
* Prompt-injection attempts

### Platform Questions

Tests include:

* Guest checkout
* Shipping time
* Order tracking
* Returns
* Payment methods
* Questions not fully covered by the FAQ

### Other Requests

Tests include:

* General requests outside the support scope
* Product recommendations
* Attempts to obtain the system prompt

### V3 Flow Test Results

The latest automated test run produced:

| Metric             |     Result |
| ------------------ | ---------: |
| Total tests        |         17 |
| Perfect            |         12 |
| Partial            |          2 |
| Failed             |          3 |
| Average score      |     0.7647 |
| Overall percentage | **76.47%** |

---

## 5. Evaluation Dataset

The `generate-eval-dataset.py` script runs the test suite against the AgentCore harness and produces a JSONL dataset for Amazon Bedrock Evaluations.

The generated dataset is:

`output_eval_dataset.jsonl`

The dataset contains:

* Test prompts
* Expected/reference responses
* Chatbot responses produced by the harness

The JSONL dataset was uploaded to Amazon S3 for use by the Bedrock Evaluation job.

---

## 6. Amazon Bedrock Evaluation

An automated Amazon Bedrock Evaluation job was created using LLM-as-a-judge evaluation.

### Evaluation Configuration

| Configuration                | Value                  |
| ---------------------------- | ---------------------- |
| Evaluation type              | Model Evaluation       |
| Task type                    | General                |
| Metric                       | `Builtin.Correctness`  |
| Evaluator model              | `amazon.nova-pro-v1:0` |
| Application model identifier | `my-support-chatbot`   |

### Evaluation Result

The latest evaluation produced a:

**Correctness Score: 0.94**

This significantly exceeds the target of **0.75** specified in the project review feedback, improving from an earlier run that scored **0.76**.

The evaluation result is included as:

`evaluation-result-v4.jsonl`

The evaluation configuration is included as:

`eval-job-v4.json`

A screenshot of the Bedrock Evaluation results is also included as evidence.

---

## 7. Observations and Improvements

The implementation successfully supports the three required customer-support paths and demonstrates the complete bug-report workflow from information collection through ticket persistence.

The bug-report integration was verified by checking the DynamoDB table and confirming that completed conversations resulted in persisted ticket records containing the description, reproduction steps, environment, ticket ID, and status.

### Improvements Identified from V3 Evaluation

An earlier evaluation run (V3) scored **0.76 correctness**. Reviewing the low-scoring cases surfaced two prompt gaps in the platform-question route:

1. The fallback used when a platform question was not covered by the FAQ did not consistently state the human-support phone number. In some cases, the assistant used vaguer language such as *"I'll flag this for the team."*

2. The system prompt's own fallback example used a real, specific city name. The model would occasionally echo this location into unrelated, general delivery-time questions that the FAQ could actually answer directly.

### Prompt and Code Improvements

The system prompt was revised to:

* Add an explicit, mandatory **HUMAN HANDOFF BEHAVIOR** section that always includes the literal support phone number.
* Separate general FAQ-answerable questions from specific unanswerable questions as mutually exclusive cases.
* Remove the named-place example so the model can only reference a location that the customer themselves provided.

A code-level sanitization step was also added to `chat.py` and `generate-eval-dataset.py` to strip any leaked `<thinking>` reasoning blocks or route-label prefixes, such as `OTHER REQUEST`, from responses before they reach the customer. This acts as a safety net alongside the prompt-level instruction.

### V4 Evaluation Improvement

Re-running the evaluation after these changes (V4) raised the correctness score from **0.76 to 0.94**.

The remaining lower-scoring case in V4 involves a bug-report scenario with a missing environment field. This indicates an area for further refinement in the bug-report completeness gate, independent of the platform-question fixes described above.

---

## 8. Evidence Included

The submission includes the following implementation and evaluation evidence.

### Implementation Files

* `system_prompt.txt`
* `flow-tests.json`
* `generate-eval-dataset.py`
* `output_eval_dataset.jsonl`
* `eval-job-v4.json`
* `evaluation-result-v4.jsonl`

### Screenshots

The submission includes screenshots demonstrating:

* Bug-report conversation showing information collection
* Bug-report tool invocation
* Created DynamoDB bug-report record
* Covered FAQ response
* Uncovered/off-FAQ question response showing the human-support phone number redirect
* Other Request response showing the human-support phone number redirect
* Amazon Bedrock Evaluation results showing the **0.94 correctness score**
