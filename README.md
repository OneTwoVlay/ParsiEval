# ParsiEval — Benchmark for Persian LLM Language Understanding

[![Releases](https://img.shields.io/badge/Releases-download-blue?logo=github)](https://github.com/OneTwoVlay/ParsiEval/releases)

![Persian text and language model](https://upload.wikimedia.org/wikipedia/commons/6/6d/Nasta%27līq_script_of_Persian_Shibani.jpg)

Table of contents
- Overview
- Why ParsiEval
- Dataset at a glance
- Task design and examples
- File and data format
- Evaluation protocol
- Baselines and expected results
- How to download and run (release)
- Quick usage examples
- Tips for running at scale
- Contribution guide
- Citation
- License
- Changelog
- Acknowledgements

Overview 🚀
ParsiEval is a focused benchmark for Persian language understanding. It measures LLMs on comprehension, knowledge, and reasoning in Persian. The suite contains curated multiple-choice questions across four domains: history, literature, general knowledge, and science. Each item tests a specific skill such as fact recall, contextual reading, or multi-step inference.

Why ParsiEval ✨
- Fill a gap. Most public benchmarks emphasize English. ParsiEval focuses on Persian.
- Standardize evaluation. Provide one set of tasks that researchers can use to compare models.
- Measure both knowledge and reasoning. Include items that need recall and items that need inference.
- Short and useful. 364 items make experiments fast while still meaningful.

Dataset at a glance 📊
- Total items: 364 multiple-choice questions.
- Domains:
  - History
  - Literature
  - General Knowledge
  - Science
- Format: Each item uses a stem in Persian plus four options (A, B, C, D). One option is correct.
- Difficulty: Mixed. Items range from basic fact recall to multi-step reasoning.
- License: CC BY-SA (see License section).

Design goals
- Test comprehension of Persian text and facts.
- Keep items short so evaluation runs fast.
- Avoid cultural bias where possible.
- Make answer keys explicit and machine-readable.

Task design and sample items 🧩
Each question follows a clear structure:
- id: Unique string
- context (optional): Short Persian paragraph the question refers to
- question: The actual question in Persian
- choices: List of four choices labeled A–D
- answer: Correct label and text

Example 1 — Fact recall (History)
- question: "کدام پادشاه ایران در زمان صفویه تاجگذاری کرد؟"
- choices: A) شاه طهماسب II, B) شاه عباس اول, C) نادرشاه, D) کریم خان زند
- answer: B

Example 2 — Comprehension (Literature)
- context: Short excerpt from a Persian poem.
- question: "در بیت بالا منظور شاعر از 'سایه' چیست؟"
- choices: A) پناهگاه, B) یاد, C) ترس, D) امید
- answer: A

Example 3 — Reasoning (Science)
- question: "اگر سرعت نور در خلا ثابت و برابر c باشد، چه تغییری در طول موج یک موج نوری وقتی فرکانس آن دو برابر شود رخ می‌دهد؟"
- choices: A) طول موج نصف می‌شود, B) طول موج دو برابر می‌شود, C) طول موج ثابت می‌ماند, D) طول موج صفر می‌شود
- answer: A

Data format (schema) 📦
The official dataset uses JSONL where each line is a JSON object. Fields:
- id: string
- domain: one of ["history","literature","general","science"]
- context: string or null
- question: string
- choices: {"A": string, "B": string, "C": string, "D": string}
- answer: "A"|"B"|"C"|"D"
- difficulty: integer (1-5)
- source: string (optional)

Sample JSON line:
{"id":"p001","domain":"history","context":null,"question":"...","choices":{"A":"...","B":"...","C":"...","D":"..."},"answer":"B","difficulty":2,"source":"curated"}

Evaluation protocol ✅
- Primary metric: Accuracy (percentage of correct answers).
- Secondary metrics:
  - Per-domain accuracy for analysis.
  - Calibration: measure model confidence vs accuracy when models provide probabilities.
  - Few-shot vs zero-shot: report both when applicable.
- Format for predictions: Each prediction file is JSONL with fields:
  - id: matches dataset id
  - prediction: "A"/"B"/"C"/"D"
  - probability: optional float 0–1 when model outputs confidences

Baseline evaluation script
- The release contains an evaluation script that reads the gold JSONL and a prediction JSONL and prints:
  - overall accuracy
  - per-domain accuracies
  - confusion matrix summary
- The evaluation script also supports token-level logging for error analysis.

Baselines and expected results 📈
We provide example baseline scores for reference. These are illustrative.
- Small transformer (few hundred million params) — zero-shot: 28–36% accuracy
- Medium LLM (1–10B) — zero-shot: 40–55% accuracy
- Medium LLM — few-shot: 48–62% accuracy
- Large LLM (commercial) — few-shot/chain-of-thought: 62–78% accuracy

Use these baselines to gauge progress. Report both zero-shot and few-shot results.

How to download and run (release) ⬇️
You can get the release artifacts here:
[![Download Release](https://img.shields.io/badge/Download%20Release-v1.0-brightgreen)](https://github.com/OneTwoVlay/ParsiEval/releases)

The release page contains the dataset and tools. Download the release archive and run the included script. The release includes:
- parsi_eval_v1.0.tar.gz — dataset and examples
- run_parsieval.sh — simple runner and evaluator
- eval_parsieval.py — Python evaluation script
- sample_predictions.jsonl — sample prediction file
- LICENSE.txt

Download and run steps (example)
- Step 1: Visit the releases page at `https://github.com/OneTwoVlay/ParsiEval/releases`.
- Step 2: Download `parsi_eval_v1.0.tar.gz` from the release assets.
- Step 3: Extract the archive: `tar -xzf parsi_eval_v1.0.tar.gz`
- Step 4: Make the runner executable: `chmod +x run_parsieval.sh`
- Step 5: Run the evaluation: `./run_parsieval.sh --pred sample_predictions.jsonl`

If the release link fails or you need a different asset, check the "Releases" section on the repository page.

Quick usage examples 🧪
Zero-shot single prompt (example)
- Use a prompt template that contains the question and choices in Persian.
- Ask the model to return one letter among A–D.
- Parse the output and map it to the label.

Few-shot prompt (example)
- Add 3–5 in-context examples in Persian.
- Keep examples short and balanced across domains.
- Then append the target question and ask for a label.

Programmatic evaluation with Python (high-level)
- Load dataset JSONL.
- Send prompts to model in batches.
- Write predictions to JSONL with id and prediction fields.
- Run the provided `eval_parsieval.py` to compute metrics.

Tips for reliable evaluation 🛠
- Normalize model output. Accept variations like "A", "A)", "گزینه A" and map them to standard labels.
- Cap output length to avoid verbose answers.
- When using probabilistic outputs, use the top label by probability.
- Run multiple seeds for sampling-based models and report mean and standard deviation.
- Log examples where models disagree with gold. Use per-domain logs for error analysis.

Running at scale
- Batch prompts to reduce API overhead.
- Cache embeddings if you use retrieval.
- Use deterministic decoding for accuracy measurement when possible.
- For chain-of-thought tests, keep the test set smaller to reduce cost.

Error analysis and tools 🔎
- The release includes tools for:
  - Confusion matrix generation
  - Per-question difficulty breakdown
  - Example-level annotation export
- Use per-domain breakdown to spot weak areas like literature or science.

Contributing 🙌
We welcome contributions. Useful contributions include:
- More items across domains.
- Better question diversity.
- Translation checks and orthography fixes.
- Additional evaluation scripts.
How to contribute
- Fork the repo.
- Add items in the JSONL format described above.
- Add tests and examples.
- Open a PR with a clear description and at least one example.
Guidelines
- Keep questions self-contained.
- Do not include offensive or private content.
- Add sources where possible.

Citation 📚
If you use ParsiEval in a paper or report, cite the repository. Use this suggested template:
- ParsiEval: A Benchmark for Persian Language Understanding. OneTwoVlay. GitHub repository.

License
- ParsiEval uses a permissive open license. See `LICENSE.txt` in the release for details.

Changelog 📝
- v1.0 — Initial release. 364 items. JSONL format. Evaluation script included.

Acknowledgements
- Contributors who provided domain expertise in Persian literature and history.
- Open datasets and public domain texts used to create context passages.
- Tooling and libraries used in the scripts.

Contact and support
- Open an issue in the repository for bugs or questions.
- For contributions, open a pull request and reference relevant issues.

Final notes about the release link
- Visit the release page: `https://github.com/OneTwoVlay/ParsiEval/releases`
- Download the release asset `parsi_eval_v1.0.tar.gz` from that page and execute the included `run_parsieval.sh` to reproduce evaluation results.
- If the link does not work, check the "Releases" section of the repository page for the latest assets.

Additional resources and examples
- Example prompt templates (Persian)
  - Zero-shot: `سوال: <question>\nگزینه‌ها:\nA) <A>\nB) <B>\nC) <C>\nD) <D>\nپاسخ صحیح را فقط با یکی از حروف A یا B یا C یا D بنویس.`
  - Few-shot: Add 3 example Q/A pairs before the target question.
- Logging format for debugging
  - Save each request and response as a JSON line with timestamp, prompt, raw_response, mapped_label.

Metrics to report in papers
- Overall accuracy
- Accuracy per domain
- Accuracy by difficulty bin
- Model size and decoding strategy (greedy, beam, sampling)
- Number of in-context examples for few-shot

Security and privacy
- Do not include private or sensitive content in contributed items.
- The dataset avoids personal data.

Repository links and quick access
- Primary release page: https://github.com/OneTwoVlay/ParsiEval/releases
- Use the release page to download `parsi_eval_v1.0.tar.gz` and related scripts.