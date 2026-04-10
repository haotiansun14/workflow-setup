# Scientific Research with Claude Code

### A Live Demo for PhD Students in Biomedical Engineering

---

**Demo Project**: Fine-tuning TinyLlama-1.1B with LoRA for PubMed abstract disease classification

We will build a complete biomedical ML pipeline from an empty directory — and along the way, learn every major Claude Code technique for research.

---

## Roadmap

| # | Section | Feature Showcase | 
|---|---------|-----------------|
| 1 | Project Setup | CLAUDE.md, scaffolding | 
| 2 | Skills & Plugins | oh-my-claudecode ecosystem | 
| 3 | Research Planning | `/plan`, `/understand` |
| 4 | Implementation | Worktree, parallel dev | 
| 5 | Training & Monitoring | Remote, signal checks | 
| 6 | Code Review & Debugging | Review workflow, `/ralph` |
| 7 | Reporting & Visualization | HTML slides, reports |
| 8 | Version Control | Git workflow, branched conversations | 
| 9 | Summary & Q&A | Cheat sheet, resources |



# Part 1: Project Setup

> From an empty directory to a fully configured research workspace.

---

## 1.1 Scaffold the Project

We start from nothing. Claude Code creates the entire project structure, dependencies, git repo, and — most importantly — a `CLAUDE.md` file.

**Live Demo:**
```
I'm starting a new ML research project called "biomedical-llm-classifier". This project will fine-tune a lightweight LLM (TinyLlama-1.1B) with LoRA for classifying PubMed abstracts into disease categories (cardiovascular, neurological, oncology, immunology, metabolic).

Please:
1. Initialize a git repo
2. Create a proper Python project structure with:
   - src/biomedlm/ package (data/, model/, training/, evaluation/ modules)
   - configs/ directory for training configs
   - scripts/ for entry points
   - tests/ directory
3. Create a pyproject.toml with dependencies: transformers, peft, datasets, accelerate, torch, scikit-learn, wandb, rich
4. Create a comprehensive CLAUDE.md with project context, coding standards, and research goals
5. Create a .gitignore for Python/ML projects (include model checkpoints, wandb, data caches)
6. Make an initial commit
```

---

## 1.2 CLAUDE.md — Your Project's Brain

`CLAUDE.md` is the single most impactful file in any Claude Code project. It is loaded into **every conversation** automatically. Think of it as documentation that doubles as an instruction set.

**Live Demo:**
```
Show me the CLAUDE.md that was created. Then enhance it by adding:
- A "Research Context" section explaining this is a BME research project for PubMed abstract classification
- A "Coding Standards" section requiring type hints, docstrings, and pytest-style tests
- A "Key Design Decisions" section noting we use LoRA for parameter-efficient fine-tuning
- An "Important Paths" section listing where data, models, and configs live
```

**Key Takeaway:** CLAUDE.md prevents the AI from "forgetting" your project context between sessions. Update it as your project evolves — it's living documentation.

---

# Part 2: Skills & Plugins

> Extending Claude Code with specialized research workflows.

---

## Skills We'll Use Today

| Skill | Command | What It Does |
|-------|---------|-------------|
| **Plan** | `/plan` | Strategic planning with Opus-level reasoning |
| **Ralph** | `/ralph` | Persistent loop — keeps working until verified complete |
| **Autopilot** | `/autopilot` | Full autonomous pipeline: plan, code, test, review |
| **Understand** | `/understand` | Interactive codebase knowledge graph |
| **Team** | `/team` | Spawn parallel worker agents |
| **Frontend Design** | `/frontend-design` | Create stunning HTML presentations |
| **Worktree** | Agent isolation | Parallel dev in separate branches |
| **Remote** | `claude --remote` | Control from your phone |

We'll see each of these in action throughout the demo.

---

# Part 3: Research Planning

> Plan before you code. 30 minutes of planning saves 3 hours of wrong-direction coding.

---

## 3.1 Design the Research Pipeline with `/plan`

The planner uses Opus (the most capable model) for architectural thinking. It considers dependencies, trade-offs, and produces a living document.

**Live Demo:**
```
/plan

I need to plan the implementation of our biomedical LLM classifier with more details. The research pipeline should include:

1. Data Pipeline: Load PubMed abstracts from HuggingFace datasets, preprocess and tokenize, create train/val/test splits stratified by disease category
2. Model Setup: Load TinyLlama-1.1B, configure LoRA adapters (rank=8, alpha=16) targeting attention layers
3. Training Loop: Use HuggingFace Trainer with cosine LR schedule, implement early stopping based on validation F1
4. Evaluation: Per-class precision/recall/F1, confusion matrix, statistical significance testing
5. Experiment Tracking: W&B integration for loss curves, learning rate, GPU utilization

Key constraints:
- Must run on a single GPU (A100 or even T4)
- Training should complete in <30 minutes for the demo
- Code must be modular enough to swap models/datasets easily

Please create a detailed implementation plan.
```

---

## 3.2 Understand a Codebase with `/understand`

When building on existing code — a labmate's project, an open-source repo — `/understand` creates an interactive knowledge graph showing how everything connects.

**Live Demo:**
```
/understand

Analyze this project structure and create a knowledge graph showing how the modules relate to each other. Focus on the data flow from raw PubMed abstracts through preprocessing, training, and evaluation.
```

The output is an interactive HTML dashboard you can explore in a browser. Extremely useful when joining a new research group or inheriting code.

---

---

## 3.3 Worktree — Parallel Development

**This is how you run parallel experiments without going insane with branch switching.**

Git worktrees create a **physically separate directory** for each branch. Unlike regular branches (which share one working directory), worktrees let you have multiple Claude Code sessions running simultaneously — each on a different experiment.

**Live Demo:**
```
I want to demonstrate parallel development with worktrees. Please:

1. Create a worktree branch called "experiment/rank-comparison" where we'll test different LoRA ranks (4, 8, 16, 32)
2. In the main branch, continue implementing the training loop

Explain to the audience what just happened with the worktree — how git worktrees allow truly parallel development without stashing or switching branches.
```

### Designed Worktree Cases

| Scenario | Main Branch | Worktree Branch | Point |
|----------|-------------|-----------------|-------|
| **Hyperparameter search** | Training with rank=8 | Testing rank=4,16,32 | Parallel experimentation |
| **Architecture comparison** | TinyLlama + CLS pooling | TinyLlama + mean pooling | Compare without conflicts |
| **Bug fix during development** | Building evaluation module | Fix data pipeline bug | No stashing needed |

**Tip — Branched Conversations:** You can fork the conversation here to explore alternative architectures (e.g., "what if we used a different model?") without losing this plan. Think of it like git branches for your AI chats.


# Part 4: Implementation Sprint (Optional)

> From plan to code. Watch Claude Code implement complete modules, then see how worktrees enable parallel experiments.

---

## 4.1 Data Pipeline

**Live Demo:**
```
Implement the data pipeline module in src/biomedlm/data/. I need:

1. dataset_loader.py:
   - Function to load PubMed abstracts from HuggingFace (use "pubmed_qa" or create a synthetic dataset for demo)
   - Map abstracts to 5 disease categories (cardiovascular, neurological, oncology, immunology, metabolic)
   - For the demo, create a synthetic dataset generator that produces ~5000 labeled abstracts
   
2. preprocessor.py:
   - Text cleaning (remove special chars, normalize whitespace)
   - Tokenization using AutoTokenizer with padding/truncation to max_length=256
   - Create stratified train/val/test splits (70/15/15)

3. data_module.py:
   - PyTorch Dataset class wrapping the preprocessed data
   - DataLoader factory with configurable batch size

Also create a config file configs/data_config.yaml with all hyperparameters.

Write tests for the dataset loader and preprocessor in tests/test_data.py.
```

Notice: tests are written alongside the implementation, and hyperparameters live in config files — not hardcoded. Good research practice.

---

## 4.2 Model Module

**Live Demo:**
```
Now implement the model module in src/biomedlm/model/:

1. lora_model.py:
   - Function to load TinyLlama-1.1B from HuggingFace
   - Configure LoRA with PEFT: rank=8, alpha=16, dropout=0.1, target_modules=["q_proj", "v_proj"]
   - Add a classification head (5 classes) on top of the base model
   - Function to count trainable vs total parameters (should be <1% trainable)

2. configs/model_config.yaml with model name, LoRA hyperparameters

Write a test that verifies the model loads and the parameter count is correct.
```
---

## 4.3 Training Loop

**Live Demo:**
```
Implement the training module in src/biomedlm/training/:

1. trainer.py:
   - Use HuggingFace Trainer with custom compute_metrics (accuracy, macro F1, per-class F1)
   - Cosine learning rate scheduler with warmup (10% of steps)
   - Early stopping callback based on validation F1 (patience=3)
   - Gradient accumulation for effective batch size of 32
   - Mixed precision (fp16) training
   - Save best checkpoint based on eval_f1

2. callbacks.py:
   - Custom callback that logs training metrics in a structured way
   - Rich progress bar with live metrics display
   - Callback to save training curves as JSON for later plotting

3. configs/training_config.yaml:
   - Learning rate: 2e-4
   - Epochs: 5 (enough for demo)
   - Batch size: 8 (with grad accumulation = 4)
   - Warmup ratio: 0.1
   - Weight decay: 0.01

4. scripts/train.py - entry point that ties everything together; periodically log loss and other metrics into log file.

Make sure the training script prints a clear summary at the start: model params, dataset size, estimated training time.
```

---

# Part 5: Training & Monitoring

> Launch, monitor, and check from your phone. The most expensive bug in ML research is the one you discover after 8 hours of GPU time.

---

## 5.1 Launch Training

**Live Demo:**
```
Run the training script on <YOUR_CLUSTER_NAME> under <YOUR_PERSONAL_DIR>. Before launching, verify:
1. The synthetic dataset is generated correctly
2. The model loads and LoRA is applied (check trainable parameter count)
3. A quick sanity check: one forward pass works

For training env, first check if any local installed env are useful; if not, set up an env under the working repo.
Then launch training in the background: python scripts/train.py
Use run_in_background so we can continue working while it trains.
If W&B login prompts appear, use WANDB_MODE=offline as fallback.
```

With `run_in_background`, Claude Code keeps running while training proceeds. You can ask about progress at any time.

---

## 5.2 Monitor Progress

**Live Demo:**
```
Check the training progress. Show me:
1. Current epoch and step
2. Training loss trend
3. Validation metrics if available
4. GPU memory usage
5. Estimated time remaining

Format this as a clean summary table.
```

---

## 5.3 Remote — Check Training from Your Phone

Start any Claude Code session with `claude --remote` and you get a URL you can open on **any device** — your phone, tablet, another laptop.

Same full capabilities: read code, review logs, check metrics, even make fixes. Check training from the subway, lunch, or during a lab meeting.

```bash
claude --remote
# → Opens a URL you can access from any browser
```

---

## 5.4 Periodic Signal Check

Don't blindly wait for training to finish. Check every ~25% of training for signal. This discipline catches problems before they waste GPU hours.

**Live Demo:**
```
I want to set up a monitoring routine for our training. Please:

1. Read the training logs and plot the loss curve so far (just print an ASCII plot or save a matplotlib figure)
2. Check if the validation F1 is improving — are we seeing signal?
3. If losses plateaued or validation metrics degraded, suggest what to change (LR, LoRA rank, data issues)
4. Check for common training issues: gradient norms exploding, loss spikes, class imbalance effects

This is what I call "periodic signal check" — a discipline every ML researcher should practice.
```

---

# Part 6: Code Review & Debugging

> Your AI wrote the code. Another AI reviews it. This isn't cheating — this is quality assurance.

---

## 6.1 Code Review Workflow

Code review catches bugs before they waste GPU hours. In research, pay special attention to reproducibility, numerical stability, and data leakage.

**Live Demo:**
```
Review the code we've written so far with claude code and codex (via codex skills). Focus on:

1. Correctness: Are there any bugs in the data pipeline or training loop?
2. Research reproducibility: Are all random seeds set? Is the data split deterministic?
3. Numerical stability: Any potential issues with mixed precision or gradient computation?
4. Code quality: Are the modules properly decoupled? Could someone else reproduce this?

Be thorough — pretend this code will be published as supplementary material for a paper.
```

For even more diverse perspectives, `/mmreview` runs a multi-model review using Claude + GPT + Gemini simultaneously.

---

## 6.2 Debugging — Finding a Data Leakage Bug

Let's simulate a common ML bug and walk through the debugging process.

**Live Demo:**
```
I'm going to simulate a common ML bug scenario. Please:

1. In the preprocessor, introduce a subtle data leakage bug: accidentally include validation data in training tokenizer fitting
2. Show me what the symptoms would look like (suspiciously high val accuracy)
3. Then walk through how you would debug this:
   - What metrics would look suspicious?
   - How would you trace the data flow?
   - What's the fix?
4. Fix the bug and explain the debugging thought process

This demonstrates a real debugging workflow that catches issues ML researchers commonly miss.
```

**Rule of thumb:** If your validation performance looks "too good" — that's usually a bug, not a breakthrough.

---

## 6.3 `/ralph` — Verified Completion

Ralph doesn't just say "done" — it says "done AND verified." It creates a PRD with acceptance criteria and iterates until ALL criteria pass. The difference between demo code and publication code.

**Live Demo:**
```
/ralph

Add comprehensive evaluation to our project:

1. Create src/biomedlm/evaluation/metrics.py:
   - Per-class precision, recall, F1
   - Confusion matrix generation
   - Bootstrap confidence intervals for F1 score (95% CI)
   - McNemar's test for comparing two models

2. Create src/biomedlm/evaluation/analysis.py:
   - Error analysis: find the most commonly misclassified examples
   - Attention visualization for interpretability
   - Generate a structured evaluation report as JSON

3. Create scripts/evaluate.py entry point

4. Tests for all evaluation functions

Don't stop until everything passes tests and type-checks.
```

Watch how Ralph creates a PRD, tracks progress, and only declares completion after verification.

---

# Part 7: Reporting & Visualization

> Your presentation is now version-controlled, diffable, and you can update results by re-running a prompt.

---

## 7.1 HTML Slides — Programmatic Presentations

One of the most creative Claude Code workflows: generate beautiful, interactive HTML presentations. Version-controllable (unlike PowerPoint), with interactive charts instead of static images.

**Live Demo:**
```
/frontend-design

Create a beautiful, interactive HTML presentation of our experiment results. The presentation should include:

1. Title slide: "BioMedLM: Parameter-Efficient Fine-tuning for Biomedical Text Classification"
2. Problem statement slide with BME context
3. Method slide showing the LoRA architecture (create an SVG diagram)
4. Results slide with:
   - An interactive chart showing training loss curves
   - A confusion matrix heatmap
   - Per-class F1 scores as a bar chart
   - Use placeholder data that looks realistic
5. Comparison slide: full fine-tuning vs LoRA (parameter count, training time, F1)
6. Conclusions slide with key findings

Style: Clean, academic, dark theme with blue accents. Include smooth slide transitions.
Save as results_presentation.html

Make it look like something you'd present at a BME research seminar.
```

Can be hosted on GitHub Pages for free. Update results by re-running the prompt with new data.

---

## 7.2 Experiment Report

**Live Demo:**
```
Create a structured experiment report as a Markdown file (experiment_report.md):

## Experiment Report: BioMedLM v1.0

Include:
1. Experimental Setup (model, dataset, hyperparameters table)
2. Results Summary (best F1, accuracy, per-class breakdown)
3. Training Dynamics (describe loss curve behavior, any anomalies)
4. Ablation Study placeholder (LoRA rank comparison from our worktree branch)
5. Failure Analysis (most common error types)
6. Next Steps & Research Directions
7. Reproducibility Checklist:
   - [ ] Random seeds set
   - [ ] Data splits documented
   - [ ] Hardware/software versions logged
   - [ ] Training config saved
   - [ ] Model checkpoint accessible

Use placeholder data that looks like realistic BME ML results.
```

---

# Part 8: Summary

---

## What We Built

From an empty directory to a complete biomedical text classifier:
- **Data pipeline** — synthetic PubMed abstracts, stratified splits, tokenization
- **LoRA model** — TinyLlama-1.1B with <1% trainable parameters
- **Training loop** — cosine LR, early stopping, mixed precision, W&B tracking
- **Evaluation** — per-class F1, confidence intervals, statistical tests
- **HTML presentation** — interactive charts, version-controlled slides
- **Experiment report** — reproducibility checklist included

---

## Techniques Demonstrated

| Technique | What It Does | Research Use Case |
|-----------|-------------|-------------------|
| **CLAUDE.md** | Persistent project context | Keep AI aligned with research goals |
| **oh-my-claudecode** | Plugin with specialized agents | Architect, reviewer, scientist agents |
| **/plan** | Strategic planning | Design experiments before coding |
| **/understand** | Codebase knowledge graph | Onboarding, code archaeology |
| **/ralph** | Persistent completion loop | Guaranteed task completion |
| **Worktrees** | Parallel development | A/B test different approaches |
| **Code review** | Multi-perspective review | Catch bugs, data leakage, reproducibility |
| **HTML slides** | Programmatic presentations | Interactive, version-controlled results |
| **Remote** | Phone access via URL | Monitor training from anywhere |
| **Branched conversations** | Fork/explore/return | Investigate without losing context |
| **Signal monitoring** | Periodic training checks | Catch training issues early |

---

## Research Best Practices with AI Coding

1. **Plan before you code** — `/plan` saves hours of wrong-direction work
2. **Review your own code** — AI-generated code needs review too
3. **Monitor training signals** — Don't launch and forget
4. **Version control everything** — Commits are your undo button
5. **Reproducibility first** — Seeds, configs, environment pinning
6. **Use the right tool** — `/ralph` for persistence, worktrees for parallelism, remote for mobility

---

## Quick Reference Card

### Essential Commands

| Command | Purpose |
|---------|---------|
| `claude` | Start Claude Code |
| `claude --remote` | Start with phone-accessible URL |
| `claude --resume` | Resume last conversation |
| `/compact` | Compress context to save space |
| `/help` | Show built-in help |

### Top Skills for ML Researchers

| Skill | Command | When to Use |
|-------|---------|-------------|
| Plan | `/plan` | Before any complex task |
| Ralph | `/ralph` | Must-complete-and-verify tasks |
| Understand | `/understand` | Exploring unfamiliar codebases |
| Frontend Design | `/frontend-design` | HTML slides, dashboards, visualizations |

### ML Debugging Checklist

- [ ] Val performance too good? Check for data leakage
- [ ] Loss not decreasing? Check LR, data loading, label encoding
- [ ] Loss NaN? Numerical instability, reduce LR, check data
- [ ] All classes predicted? Check class balance, loss weighting
- [ ] GPU utilization low? Check DataLoader workers, batch size

---
