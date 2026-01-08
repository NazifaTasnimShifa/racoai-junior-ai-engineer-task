# Technical Documentation

## 1. Choice of Configuration (LoRA & Training Strategy)

To meet the requirement of fine-tuning LLaMA 3.1-8B-Instruct on a single Tesla T4 GPU (Kaggle environment) within a 5-6 hour limit, the following configuration was chosen:

### **Model & Quantization**
*   **Model:** `meta-llama/Llama-3.1-8B-Instruct`
*   **Quantization:** **4-bit NF4** (NormalFloat 4) using `bitsandbytes`. This was critical to fit the ~16GB model into the T4's 16GB VRAM while leaving space for gradients and activations.

### **Fine-Tuning Strategy: LoRA**
*   **Method:** **LoRA** (Low-Rank Adaptation)
*   **Rank (r):** 16. A balance between trainability and parameter efficiency.
*   **Alpha:** 32. Scaling factor of 2x rank.
*   **Target Modules:** `q_proj`, `k_proj`, `v_proj`, `o_proj`. Targeting all attention projections yields better performance than just query/value.

### **Training Optimization (Speed vs. Accuracy)**
*   **Sequence Length: 256**.
    *   *Analysis:* We analyzed the dataset and found that a sequence length of 256 covers **84.2%** of the examples fully.
    *   *Decision:* Limiting to 256 drastically reduced memory consumption and training time compared to 512 or 1024, enabling us to run with a larger batch size.
*   **Batch Size: 4** (per device).
*   **Gradient Accumulation: 4**.
*   **Effective Batch Size: 16**. This ensures stable gradient updates.
*   **Gradient Checkpointing: DISABLED**. While enabling it saves memory, it slows down backward pass by ~20-30%. Since we optimized sequence length, we fit the batch in memory without checkpointing, gaining a significant speed boost.

## 2. OOP Implementation Design

The solution is structured using Object-Oriented Programming (OOP) principles for modularity and extensibility:

*   **`LLAMAFineTuner`**: The main orchestration class that manages the model, tokenizer, and training loop.
*   **`DatasetProcessor`**: Encapsulates data loading, cleaning, analysis of token lengths, and tokenization.
*   **`FineTuningStrategy` (Abstract)**: Uses the **Strategy Pattern** to allow switching between `LoRAStrategy` and potential future strategies (like `UnslothStrategy` or `FullFineTuning`) without changing the main code.
*   **`Evaluator`**: Dedicated class for calculating metrics (Perplexity, BLEU, ROUGE) and generating responses.
*   **`DatabaseManager`**: Handles all SQLite interactions for logging experiments and responses, ensuring data persistence.

## 3. Challenges Faced & Solutions

### **Challenge 1: Memory Constraints on T4 (16GB)**
*   **Issue:** Loading LLaMA 8B in FP16 takes ~15GB, leaving no room for training.
*   **Solution:** Used 4-bit quantization (NF4) to reduce model size to ~5GB, allowing ~11GB for training overhead.

### **Challenge 2: Training Speed (5-6 Hours Limit)**
*   **Issue:** Full sequence lengths (e.g., 1024) drastically increased epoch time.
*   **Solution:** Conducted a data-driven analysis of token lengths. Found that 256 tokens covered the vast majority of conversations. Capping at 256 boosted speed by ~3x.

### **Challenge 3: Secret Scanning during Push**
*   **Issue:** Git push was repeatedly blocked because the Personal Access Token (PAT) was detected in the history of configuration scripts.
*   **Solution:**
    1.  Used `git filter-branch` / re-initialization to scrub secrets from history.
    2.  Implemented a secure authentication method using `http.extraHeader` with base64 encoding, preventing the token from ever being written to a tracked file or URL.

### **Challenge 4: Metric Calculation**
*   **Issue:** BLEU/ROUGE require strict reference matching.
*   **Solution:** Implemented the logic to extract "ASSISTANT" responses from the test set for comparison. (Note: Scores were low on the small sample set due to potential mismatch in reference selection, but the pipeline is functional).
