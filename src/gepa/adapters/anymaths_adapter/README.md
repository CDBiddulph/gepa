# ⭐ AnyMaths: GEPA Adapter for Solving Math Word Problems ⭐
**AnyMaths Adapter** is a GEPA Adapter for any dataset that contains math word problems of varying complexity and structure. It is designed to handle a wide range of mathematical tasks, including arithmetic, algebra, reasoning, and more.

---

### ✍️ Preparing the dataset
In `src/gepa/examples/anymaths-bench/train_anymaths.py`, a sample function to prepare a dataset is provided via `init_dataset`. This function demonstrates how to load and preprocess the dataset for training and evaluation. Notably, it includes steps for data augmentation and splitting the dataset into training, validation, and test sets. We recommend to find and download datasets from [Hugging Face dataset hub](https://huggingface.co/datasets).

#### Best format for a custom dataset
If you have a custom dataset, it is best to follow the following schema:
```
{
    "question": ...,
    "solution": ...,
    "answer": ...
}
```
Remarks:
- `question` must be a string/text.
- `solution` must be a string/text.
- `answer` must be a purely numerical, no other text or units associated.

We recommend you to upload your custom dataset to the Hugging Face dataset hub to fully utilize `datasets.load_dataset`.

---

### 🧰 Adapter Design
The AnyMaths Adapter can work for any LiteLLM supported providers (e.g., OpenAI, Google Vertex, HuggingFace, Groq, vLLM, Ollama, etc.). For this instance, we opt to choose Ollama to show that **this adapter can work for local use if one has no access to expensive GPUs or paid APIs.** But, you may freely choose this adapter with any other LiteLLM-supported provider.

---

### ✍️ Preparing the seed prompt
The seed prompt is the initial instruction you provide to the base (target) model. It sets the context for the task at hand and this prompt evolves or changes over time toward maximizing the model's performance. The default failure score (i.e., score if the model outputs are incorrect or does not satisfy a set metric) is zero.

Set the seed prompt in a separate directory under `src/gepa/examples/anymaths-bench/prompt-templates`. Inside this directory is a file `instruction_prompt.txt` which contains the seed prompt.

---

### 🪞 Specifying the reflection LM
The reflection LM is a language model used to generate feedback based on the output by the base model. You can specify the reflection LM by setting the `reflection_lm` argument when calling the `gepa.optimize` function.

A sample `reflection_lm` function call can be found in `src/gepa/examples/anymaths-bench/train_anymaths.py`.

---

### 🏃‍♂️ Running a full AnyMaths Adapter training
To run a full training session using the AnyMaths Adapter, you can use the following command:
```bash
python src/gepa/examples/anymaths-bench/train_anymaths.py --anymaths_dset_name ... --train_size ... --val_size ... --test_size ... --base_lm ... --use_api_base --api_base_url ... --reflection_lm ... --use_api_reflection --api_reflection_url ... --reflection_minibatch_size ... --budget ... --max_litellm_workers ... --seed ...
```
- `--anymaths_dset_name`: The Hugging Face `Dataset` name to use for training (e.g., `"openai/gsm8k"`, `"MathArena/aime_2025"`).
- `--train_size`: The size of the training set to use.
- `--val_size`: The size of the validation set to use.
- `--test_size`: The size of the test set to use.
- `--base_lm`: The base language model to use for GEPA training (e.g. `"ollama/qwen3:4b"`).
- `--use_api_base`: Enable this flag if you want to use the Ollama API for the base model. Otherwise, do not include this in your arguments if you are using provider APIs (e.g., OpenAI, Google Vertex, etc.).
- `--api_base_url`: (Base model) The URL to get completions for the base model. Example: Ollama uses the default `http://localhost:11434`. There is no need to set this up if you are using provider APIs. **Note: API keys and provider credentials must be set beforehand.**
- `--reflection_lm`: The reflection language model to generate feedback from base model outputs (e.g., `"ollama/qwen3:8b"`).
- `--use_api_reflection`: Similar with `--use_api_base`. Enable this flag if you want to use a specific endpoint to get completions from the reflection model.
- `--reflection_minibatch_size`: The minibatch size for the reflection LM to reflect against (default is 8).
- `--max_litellm_workers`: The maximum number of LiteLLM workers to use.
- `--budget`: The budget for the GEPA training (default is 500).
- `--seed`: The seed for the random number generator for reproducibility (default is 0).

#### 📓 Cookbook Training Runs

1. (Purely) **Using Ollama**:
    ```bash
    python src/gepa/examples/anymaths-bench/train_anymaths.py --anymaths_dset_name "openai/gsm8k" --train_size 50 --val_size 50 --test_size 50 --base_lm "ollama/qwen3:4b" --use_api_base --api_base_url "http://localhost:11434" --reflection_lm "ollama/qwen3:8b" --use_api_reflection --api_reflection_url "http://localhost:11434" --reflection_minibatch_size 8 --budget 500 --max_litellm_workers 4 --seed 0
    ```
2. (Purely) **Using Google Vertex** for Gemini users:
    ```bash
    python src/gepa/examples/anymaths-bench/train_anymaths.py --anymaths_dset_name "openai/gsm8k" --train_size 50 --val_size 50 --test_size 50 --base_lm "vertex_ai/gemini-2.5-flash-lite" --reflection_lm "vertex_ai/gemini-2.5-flash" --reflection_minibatch_size 8 --budget 500 --max_litellm_workers 4 --seed 0
    ```
3. **Using Google Vertex as base (target) LM, Ollama as reflection LM**:
    ```bash
    python src/gepa/examples/anymaths-bench/train_anymaths.py --anymaths_dset_name "openai/gsm8k" --train_size 50 --val_size 50 --test_size 50 --base_lm "vertex_ai/gemini-2.5-flash-lite" --reflection_lm "ollama/qwen3:8b" --use_api_reflection --api_reflection_url "http://localhost:11434" --reflection_minibatch_size 8 --budget 500 --max_litellm_workers 4 --seed 0
    ```
4. **Using Ollama as base (target) LM, Google Vertex as reflection LM**:
    ```bash
    python src/gepa/examples/anymaths-bench/train_anymaths.py --anymaths_dset_name "openai/gsm8k" --train_size 50 --val_size 50 --test_size 50 --base_lm "ollama/qwen3:4b" --use_api_base --api_base_url "http://localhost:11434" --reflection_lm "vertex_ai/gemini-2.5-flash" --reflection_minibatch_size 8 --budget 500 --max_litellm_workers 4 --seed 0
    ```

Once the training has completed, you may replace the optimal prompt found in `src/gepa/examples/anymaths-bench/prompt-templates/optimal_prompt.txt`.

---

### 🔬 Model evaluation after GEPA training
`src/gepa/examples/anymaths-bench/eval_default.py` is used to perform model evaluation on the test split. Feel free to modify this script to fit your custom evaluation scheme. Example: `"openai/gsm8k"` - `test`. The evaluation scores will be displayed in the terminal once the evaluation has been completed.

How to run the evaluation script:
1. **Using Ollama**:
    ```bash
    python src/gepa/examples/anymaths-bench/eval_default.py --anymaths_dset_name "openai/gsm8k" --model "ollama/qwen3:4b" --use_api_url --api_url "http://localhost:11434" --batch_size 8 --max_litellm_workers 4
    ```
2. **Use Google Vertex** for Gemini users:
    ```bash
    python src/gepa/examples/anymaths-bench/eval_default.py --anymaths_dset_name "openai/gsm8k" --model "vertex_ai/gemini-2.5-flash-lite" --batch_size 8 --max_litellm_workers 4
    ```

**Note: The model that was used in GEPA training must also be the same model in performing model evaluation.**

---

### 🧪 Experiments
| Dataset | Base LM | Reflection LM | Accuracy, % (Before GEPA) $\uparrow$ | Accuracy, % (After GEPA) $\uparrow$ | GEPA Budget | Train-Val-Test Split Samples Used in GEPA Optimization |
| ------- | ------- | ------------- | ---------------------- | --------------------- | ------------ | ------ |
| `"openai/gsm8k"` | `"ollama/qwen3:4b"` | `"ollama/qwen3:8b"` | 18 | 23 (**+5**) | 500 | 50-50-50 |
| `"openai/gsm8k"` | `"vertex_ai/gemini-2.5-flash-lite"` | `"vertex_ai/gemini-2.5-flash"` | 31 | 33 (**+2**) | 500 | 50-50-50 |

**Notice of WIP**: More tests will be done soon on other models (preferrably, small language models first).

---

### 🏦 Prompt bank of optimal prompts

* Model: `"ollama/qwen3:4b"`, Dataset: `"openai/gsm8k"`, Budget: `500`:
    ```
    ### Task Instruction: Solve Multi-Step Mathematical Problems with Precision and Contextual Understanding

    You are tasked with solving problems that require careful parsing of contextual information, breaking down multi-step calculations, and ensuring accuracy in arithmetic and logical reasoning. Follow these steps to address diverse problem types (e.g., percentages, cost calculations, score determination, and distance computations):

    ---

    #### **1. Parse the Problem**
    - **Identify Key Values**: Extract numbers, percentages, fractions, and relationships (e.g., "40% of 60 students," "6 more than half of Ella\'s score").
    - **Understand Relationships**: Determine if values are additive, multiplicative, or comparative (e.g., "round trips" imply doubling one-way distances, "cost per item" requires multiplication).
    - **Clarify Ambiguities**: Resolve unclear phrasing (e.g., "half the score" refers to half the total items, not half the incorrect answers).

    ---

    #### **2. Break Down the Problem**
    - **Segment into Steps**: Divide the problem into smaller, manageable parts (e.g., calculate individual components before summing).
    - **Apply Formulas**: Use appropriate mathematical operations (e.g., percentage = part/whole × 100, total cost = (item count × price)).
    - **Account for Context**: Adjust calculations based on problem specifics (e.g., "round trip" requires doubling one-way distance, "score" may involve subtracting incorrect answers from total items).

    ---

    #### **3. Perform Calculations**
    - **Use Precise Arithmetic**:
        - For percentages: $ \\text{Percentage} \\times \\text{Total} $.
        - For fractions: $ \\frac{\\text{Numerator}}{\\text{Denominator}} \\times \\text{Value} $.
        - For multi-step operations: Follow order of operations (PEMDAS) and verify intermediate results.
    - **Avoid Common Errors**:
        - Misinterpreting phrases like "half the score" (e.g., half of total items, not half of incorrect answers).
        - Confusing "round trips" (up + down) with single trips.
        - Incorrectly applying percentages to the wrong base (e.g., 40% of students vs. 40% of total score).

    ---

    #### **4. Validate the Answer**
    - **Check Logical Consistency**: Ensure results align with problem constraints (e.g., total students = sum of groups, total cost = sum of individual costs).
    - **Verify Units and Formatting**: Confirm answers match required formats (e.g., boxed numbers, currency symbols, or percentage notation).
    - **Cross-Validate with Examples**: Compare calculations against similar problems (e.g., "If 40% of 60 students = 24, then 60 - 24 = 36").

    ---

    #### **5. Finalize the Response**
    - **Present the Answer Clearly**: Use the exact format requested (e.g., `\boxed{36}` for numerical answers, `$77.00` for currency).
    - **Include Step-by-Step Reasoning**: Explicitly show calculations (e.g., `18 * $2.50 = $45.00`, `8 * $4.00 = $32.00`).
    - **Highlight Key Decisions**: Note critical choices (e.g., "Half of Ella's score = 36 items / 2 = 18 items").

    ---

    ### **Examples of Problem Types**
    1. **Percentage Problems**:
        - *Input*: "40% of 60 students got below B."
        - *Solution*: $ 0.40 \\times 60 = 24 $, $ 60 - 24 = 36 $.
    2. **Cost Calculations**:
        - *Input*: "18 knobs at $2.50 each and 8 pulls at $4.00 each."
        - *Solution*: $ 18 \\times 2.50 + 8 \\times 4.00 = 45 + 32 = 77 $.
    3. **Score Determination**:
        - *Input*: "Ella got 4 incorrect answers; Marion got 6 more than half of Ella's score."
        - *Solution*: Total items = 40, Ella's correct = 36, half = 18, Marion = 18 + 6 = 24.
        
    ---

    ### **Key Niche Information**
    - **Percentages**: Always apply to the total (e.g., 40% of 60 students = 24 students, not 40% of 40 items).
    - **Round Trips**: Double one-way distances (e.g., 30,000 feet up + 30,000 feet down = 60,000 per trip).
    - **Score Calculations**: Subtract incorrect answers from total items (e.g., 40 items - 4 incorrect = 36 correct).
    - **Currency Formatting**: Use decimal points and symbols (e.g., `$77.00`, not `77`).

    ---

    ### **Final Output Format**
    Always conclude with:
        `Final Answer: \boxed{<result>}`
    For non-numeric answers, use:
        `Final Answer: <result>`

    Ensure calculations are explicitly shown and errors are corrected based on problem context.
    ```
* Model: `"vertex_ai/gemini-2.5-flash-lite"`, Dataset: `"openai/gsm8k"`, Budget: `500`:
    ```
    You are an AI assistant that solves mathematical word problems. You will be given a question and you need to provide a step-by-step solution to the problem. Finally, you will provide the answer to the question.

    When outputting the final answer, make sure there are no other text or explanations included, just the answer itself.

    The following fields are what you need to include in your response:
    - final_answer: The final answer to the question.
    - solution_pad: The step-by-step solution to the problem.

    Here are specific guidelines for generating your response:

    1.  **Understand the Problem Thoroughly:** Carefully read and analyze the word problem to ensure a complete understanding of all given information, constraints, and the specific question being asked. Pay close attention to units and how different quantities relate to each other.

    2.  **Formulate the Step-by-Step Solution (solution_pad):**
        *   Develop a clear, logical, and sequential step-by-step solution. Each step should be a distinct operation or deduction required to move closer to the final answer.
        *   Clearly state what is being calculated or determined in each step.
        *   Perform all necessary calculations with high precision and accuracy. Double-check all numerical operations (addition, subtraction, multiplication, division, etc.) to prevent errors.
        *   If the problem involves converting between different forms of a quantity (e.g., converting a monetary value into a count of items, or time units), explicitly show this conversion as a step.
            *   **Domain-Specific Interpretation Example:** If Barry has "$10.00 worth of dimes", first convert this value to the number of dimes (since a dime is $0.10, Barry has $10.00 / $0.10 = 100 dimes). If the problem then states Dan has "half that amount" and asks for the number of dimes Dan has, interpret "half that amount" as half the *number* of dimes Barry has (100 dimes / 2 = 50 dimes), rather than half the monetary value. Always aim for the most logical interpretation that leads to the requested unit in the final answer.
        *   The `solution_pad` field must *only* contain the clean, direct step-by-step solution. Do not include any internal monologues, self-corrections, re-evaluations, alternative thought processes, or debugging notes within this field.

    3.  **Calculate and Output the Final Answer:**
        *   Based on your thoroughly computed step-by-step solution, determine the exact numerical answer to the question.
        *   The `final_answer` field must contain *only* the numerical value. Do not include any currency symbols (e.g., "$"), units (e.g., "dimes", "hours"), or any other descriptive text or explanation in this field. For example, if the answer is 4625 dollars, output `4625`. If the answer is 52 dimes, output `52`.
        *   Ensure the final answer numerically matches the result of your `solution_pad` calculations.'
    ```


---

### 🔍 Observations on the structure of the derived optimal prompt
- For small language models:
    * Goal-oriented: The prompt starts by clearly stating the overall task, which is to solve multi-step math problems with precision.
    * Chain-of-Thought: It breaks down the problem-solving process into a detailed, numbered sequence of five steps: **Parse**, **Break Down**, **Calculate**, **Validate**, and **Finalize**.
    * Instruction Detail: Each step includes specific instructions on how to perform the task, such as identifying key values, applying formulas, and avoiding common errors.
    * Few-shot Learning: The prompt provides concrete **examples** of different problem types (percentage, costs, scores) to show the model how to apply the instructions.
    * Knowledge Base: It includes *key niche information* section that acts as a mini-rulebook, highlighting specific details and common pitfalls like "round trips" and currency formatting.
    * Structured Output: The prompt ends by defining a strict **final output format** to ensure the model's answer is consistent and easy to read.

- For provider models:
    * Fewer tokens: The prompt is more concise, using fewer tokens to convey the same information, which can lead to faster processing and lower costs.
    * Straightforward: Main instruction and output format are placed at the first parts of the prompt. Detailed guidelines are provided in a structured manner to facilitate understanding after the main instruction and output format.

---

### 👨‍🔬 Contributor
This adapter was contributed by **Emmanuel G. Maminta**. [[LinkedIn]](https://linkedin.com/in/egmaminta) [[GitHub]](https://github.com/egmaminta)