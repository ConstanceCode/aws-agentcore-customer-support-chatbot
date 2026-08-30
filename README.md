# Customer Support Chatbot

## Project Overview

This project implements a customer support chatbot for a fictional online shop using an **Amazon Bedrock AgentCore managed harness**.

The chatbot routes every customer message into one of three support paths:

1. **Bug Report** — collects the required bug information and creates a support ticket.
2. **Platform Question** — answers questions that are covered by the provided FAQ.
3. **Other Request** — redirects requests outside the supported scope to the human support line.

The system prompt defines the routing and conversation behavior. The bug-report path uses an AgentCore Gateway to invoke the bug-report tool, which persists completed reports in Amazon DynamoDB.

---

## 1. Bug Report Path

The bug-report path is defined directly in `system_prompt.txt`.

When a customer reports a bug, the assistant collects the required information before creating a ticket:

* **Bug description**
* **Steps to reproduce**
* **Environment information** such as browser, operating system, or device

The assistant is instructed to ask follow-up questions when information is missing and only invoke the bug-report tool once the required information has been collected.

The completed bug report is sent through the **AgentCore Gateway** to the bug-report tool. The resulting ticket is persisted in the:

`bug-report-tool-stack-bug-reports`

DynamoDB table.

### Example persisted ticket

A successful test created a DynamoDB record containing:

* Ticket ID: `e56999c5-12ca-4dfe-84fb-f50020ba2f76`
* Description: `When I click checkout, nothing happens.`
* Steps to reproduce: `Click the checkout button.`
* Environment: `Windows 11, Chrome 139.`
* Status: `OPEN`

Screenshots are included as evidence of the multi-turn bug-report conversation, tool invocation, and DynamoDB record creation.

---

## 2. Platform Question Path

Platform questions are answered using the provided `online_shop_faq.md` content.

The chatbot was tested with questions that are covered by the FAQ, including questions about:

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

Examples include requests such as:

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

The test suite covers all three required paths:

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

The dataset contains the test prompts, expected/reference responses, and the chatbot responses produced by the harness.

The JSONL dataset was uploaded to Amazon S3 for use by the Bedrock Evaluation job.

---

## 6. Amazon Bedrock Evaluation

An automated Amazon Bedrock Evaluation job was created using **LLM-as-a-judge** evaluation.

### Evaluation configuration

* Evaluation type: Model Evaluation
* Task type: General
* Metric: `Builtin.Correctness`
* Evaluator model: `amazon.nova-pro-v1:0`
* Application model identifier: `my-support-chatbot`

The latest evaluation produced a:

### **Correctness Score: 0.76**

This exceeds the target of **0.75** specified in the project review feedback.

The evaluation result is included as:

`evaluation-result-v3.jsonl`

The evaluation configuration is included as:

`eval-job-v3.json`

A screenshot of the Bedrock Evaluation results is also included as evidence.

---

## 7. Observations

The implementation successfully supports the three required customer-support paths and demonstrates the complete bug-report workflow from information collection through ticket persistence.

The bug-report integration was verified by checking the DynamoDB table and confirming that completed conversations resulted in persisted ticket records containing the description, reproduction steps, environment, ticket ID, and status.

The automated evaluation achieved a **0.76 correctness score**, which is above the requested 0.75 target.

The test results also identified several edge cases where the chatbot did not consistently wait for missing bug-report information before creating a ticket. These results provide clear areas for further prompt and conversation-flow refinement.

---

## 8. Evidence Included

The submission includes the following evidence:

### Implementation

* `system_prompt.txt`
* `flow-tests.json`
* `generate-eval-dataset.py`
* `output_eval_dataset.jsonl`
* `eval-job-v3.json`
* `evaluation-result-v3.jsonl`

### Screenshots

* Bug report conversation showing information collection
* Bug-report tool invocation
* Created DynamoDB bug-report record
* Covered FAQ response
* Uncovered FAQ response and support redirection
* Other Request response and support redirection
* Amazon Bedrock Evaluation results showing the **0.76 correctness score**
