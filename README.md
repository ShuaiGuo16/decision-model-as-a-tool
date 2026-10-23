# Decision Models as Tools

A notebook demo of an LLM agent delegating an investigation to a tool powered by a decision model. The example investigates a delayed order and finds a potential delivery option that meets the customer's deadline.

Open [decision_model_tools_question.ipynb](decision_model_tools_question.ipynb) for the implementation, saved outputs, and cost calculation.

## Run the notebook

You need Python 3.11+ and an Azure Foundry resource with GPT-6-Luna and Microsoft Decision-1 deployments.

1. Create a virtual environment:

   ```sh
   python -m venv .venv
   ```

   Activate it, then install the dependencies:

   ```sh
   python -m pip install -r requirements.txt
   ```

2. Copy [.env.example](.env.example) to `.env` and fill in your Azure endpoint, API key, and deployment names.
3. Open the notebook, select the virtual environment as its Python kernel, and run the cells from top to bottom. Running the notebook makes paid model requests.

The backend uses synthetic records stored in the notebook. No external order system is required, and the lookups do not change orders or reserve stock.

## How it works

- **Decision-model tool:** an OpenAI Agents SDK agent calls `order_specialist(order_id, question)`. Inside the tool, Decision-1 selects backend operations, Python executes them, and the outer LLM uses the returned evidence to write its answer.
- **Conventional agent:** another SDK agent receives the same six backend functions as individual tools and selects the operations itself.

Both use GPT-6-Luna with medium reasoning effort, the same instructions and data, and sequential tool calls. Python prepares the decision model's executable choices from retrieved records; the agent's question passes directly to Decision-1.

## Saved example

The saved outputs are pair 2, selected for the largest speedup among three successful paired trials on October 16, 2026. These are illustrative runs; rerunning produces fresh results.

| Trial | Decision-model tool | Conventional agent | Speedup |
|---|---:|---:|---:|
| 1 | 9.37 s | 21.00 s | 2.24x |
| 2 (saved) | 8.88 s | 25.44 s | 2.87x |
| 3 | 8.11 s | 20.32 s | 2.50x |

In the saved pair, the decision-model version used two Luna calls and six Decision-1 calls; the conventional version used six Luna calls. Both performed five backend lookups and identified the same potential delivery option. The conventional agent additionally checked the rerouting policy.

Estimated model cost was **$0.000529** versus **$0.000703** (24.7% lower). Costs include both models, cached input and cache writes, and reasoning output, using the Azure Global Standard rates recorded on October 16, 2026. The notebook contains the usage records, rates, and source links.

The architecture diagram is available as a [PNG](assets/hybrid_strategy.png) and an [editable SVG](assets/hybrid_strategy.svg).
