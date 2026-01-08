# RacoAI Junior AI Engineer Task

## Project Overview
This repository contains the solution for the Junior AI Engineer task: fine-tuning **LLaMA 3.1-8B-Instruct** on the **Bengali Empathetic Conversations** dataset using **LoRA** (Low-Rank Adaptation).

The goal was to implement a robust, Object-Oriented pipeline capable of running on a single T4 GPU within restricted time limits, complete with database logging and evaluation. The pipeline is designed to support full-sequence training under higher-memory GPU environments, with the current configuration optimized for Kaggle T4 hardware constraints.

## 📂 Deliverables

| Deliverable | File / Link |
| :--- | :--- |
| **Source Code** | [`llama_finetuning_final.ipynb`](llama_finetuning_final.ipynb) |
| **Execution Logs (Proof)** | [`kaggle_run/llama-finetuning-run.ipynb`](kaggle_run/llama-finetuning-run.ipynb) |
| **Metrics & Responses** | [`METRICS_AND_RESPONSES.md`](METRICS_AND_RESPONSES.md) |
| **Documentation** | [`DOCUMENTATION.md`](DOCUMENTATION.md) |
| **Dataset** | [`dataset/`](dataset/) |

## 🚀 How to Run

1.  **Install Dependencies:**
    ```bash
    pip install transformers accelerate datasets peft bitsandbytes sentencepiece evaluate rouge-score nltk sacrebleu
    ```

2.  **Open the Notebook:**
    Launch `llama_finetuning_final.ipynb` in Jupyter or Google Colab/Kaggle.

3.  **Configure:**
    *   Set your HuggingFace Token in the `Configuration` section.
    *   Ensure a GPU is available (T4 or better recommended).

4.  **Run All Cells:**
    The notebook will:
    *   Load the data.
    *   Fine-tune the model using LoRA.
    *   Log experiments to `experiments.db` (SQLite).
    *   Evaluate and save the model.

## 📊 Quick Results Summary

*   **Training Loss:** `0.4453`
*   **Perplexity:** `1.76`
*   **Training Time:** ~10 hour (on T4 GPU)
*   **Sequence Length:** 256 (data-driven compromise for T4 memory limits; full-sequence configuration supported with adjusted batch size and checkpointing)

For detailed analysis and sample outputs, see [METRICS_AND_RESPONSES.md](METRICS_AND_RESPONSES.md).

**Note on Sequence Length:**

The original task specifies preserving full sequence length during tokenization. Due to Tesla T4 (16GB VRAM) constraints and the objective of completing a full training cycle within limited time, a data-driven sequence length of 256 was selected, covering approximately 84.2% of samples without truncation.

In a production or multi-GPU environment, this pipeline supports full-sequence training by enabling gradient checkpointing, reducing batch size, and increasing gradient accumulation.

## 🛠 Tech Stack
*   **Model:** LLaMA 3.1 8B Instruct (4-bit quantized)
*   **Library:** HuggingFace Transformers, PEFT (LoRA), BitsAndBytes
*   **Tracking:** SQLite Database (Custom `DatabaseManager` class)
*   **Design:** OOP (Strategy Pattern for fine-tuning methods)

## � References
*   **Original Dataset:** [Bengali Empathetic Conversations Corpus](https://www.kaggle.com/datasets/raseluddin/bengali-empathetic-conversations-corpus)
*   **Kaggle Run (Proof):** [llama-finetuning-final](https://www.kaggle.com/code/nazifatasnimshifa/llama-finetuning-final)

## �👤 Author
**Nazifa Tasnim Shifa**
*   Email: nazifatasnimshifa@gmail.com

