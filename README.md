# Cheat Offline — Your Personal Exam Study AI (Runs 100% Offline)

Fine-tune a small language model on **your own study materials** (PDFs, DOCX, PPTX, notes) and run it completely offline on your **phone**, **laptop**, or even **IoT devices** like Raspberry Pi. No internet needed during the exam — everything runs locally.

## What This Does

1. **Takes your course materials** (lecture slides, notes, textbooks) and fine-tunes a small LLM on them
2. **Exports a tiny model file** (~350MB GGUF) that fits on any device
3. **Runs 100% offline** — no WiFi, no API calls, no cloud — just your device and the model

## Quick Start (Google Colab — Free GPU)

1. Open the notebook: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1s5mplU29DWxLB8bk7-ULm6AGQcuN2HPA)
2. Select **Runtime → Change runtime type → T4 GPU** (free tier works)
3. Run all cells in order — upload your files when prompted
4. Download the `.gguf` file at the end (~350MB)
5. Load it on your phone/device (instructions below)

**Total time: ~10 minutes** on a free Colab GPU.

---

## How It Works

```
Your PDFs/Notes → Text Extraction → Q&A Generation → Fine-tune Qwen 0.5B → Export GGUF → Run on Phone
```

The pipeline:
- **Base model**: `Qwen2.5-0.5B-Instruct` (tiny but capable, 494M params)
- **Fine-tuning**: LoRA with Unsloth (4-bit quantization, trains in ~90 seconds)
- **Export**: GGUF format with Q4_K_M quantization (~350MB file)
- **Training data**: Auto-generates Q&A pairs + chunk completions from your uploaded files

---

## Running on Mobile (Phone/Tablet)

### Option 1: PocketPal AI (Easiest — Android & iOS)

1. Install [PocketPal AI](https://github.com/nicepkg/pocketpal-ai) from Play Store / App Store
2. Open the app → tap **"Import Model"** or the `+` button
3. Select your downloaded `.gguf` file
4. Start chatting — everything runs locally on your phone

**Settings for best results:**
- Temperature: `0.7`
- Max tokens: `512`
- System prompt: `You are a study assistant. Answer questions about the course material accurately and concisely.`

### Option 2: Termux + llama.cpp (Android — Power Users)

```bash
# Install Termux from F-Droid (not Play Store)
pkg update && pkg upgrade
pkg install git cmake make clang

# Build llama.cpp
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make -j4

# Copy your model to Termux storage
cp /storage/emulated/0/Download/Qwen2.5-0.5B-Instruct.Q4_K_M.gguf ./models/

# Run interactive chat
./llama-cli -m ./models/Qwen2.5-0.5B-Instruct.Q4_K_M.gguf \
  -n 256 \
  --temp 0.7 \
  -p "What is backpropagation?"
```

### Option 3: MLC Chat (Android & iOS)

1. Install [MLC Chat](https://mlc.ai/mlc-llm/) from the app store
2. Import your GGUF model
3. Chat offline

---

## Running on Laptop/Desktop (Offline)

### Using LM Studio (GUI — Easiest)

1. Download [LM Studio](https://lmstudio.ai/) (Windows/Mac/Linux)
2. Drag-and-drop your `.gguf` file into the app
3. Start chatting in the built-in interface

### Using llama.cpp (CLI)

```bash
# Clone and build
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make -j

# Run your model
./llama-cli -m /path/to/Qwen2.5-0.5B-Instruct.Q4_K_M.gguf \
  -n 512 \
  --temp 0.7 \
  -p "Explain the bias-variance tradeoff."
```

### Using Ollama

```bash
# Create a Modelfile
echo 'FROM ./Qwen2.5-0.5B-Instruct.Q4_K_M.gguf
PARAMETER temperature 0.7
SYSTEM "You are a study assistant trained on specific course material."' > Modelfile

# Import and run
ollama create my-study-model -f Modelfile
ollama run my-study-model "What is a CNN?"
```

---

## Running on IoT Devices

### Raspberry Pi 4/5 (4GB+ RAM recommended)

```bash
# Install dependencies
sudo apt update && sudo apt install -y git cmake make g++

# Build llama.cpp
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make -j4

# Run the model (Pi 5 is faster, Pi 4 works but slower)
./llama-cli -m ./models/Qwen2.5-0.5B-Instruct.Q4_K_M.gguf \
  -n 128 \
  --temp 0.7 \
  -t 4 \
  -p "What is reinforcement learning?"
```

**Performance on Raspberry Pi:**
- Pi 5 (8GB): ~3-5 tokens/sec — usable for short answers
- Pi 4 (4GB): ~1-2 tokens/sec — slow but functional
- Pi 4 (2GB): Not recommended (model needs ~400MB RAM)

### Jetson Nano / Orin (NVIDIA GPU acceleration)

```bash
# Build llama.cpp with CUDA
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make -j GGML_CUDA=1

# Much faster with GPU offloading
./llama-cli -m ./models/Qwen2.5-0.5B-Instruct.Q4_K_M.gguf \
  -n 256 \
  -ngl 99 \
  --temp 0.7 \
  -p "Explain dimensionality reduction."
```

### ESP32 / Arduino (Microcontrollers)

These are too resource-constrained for even small LLMs. For microcontrollers, consider:
- Pre-generating a Q&A lookup table from the model and storing it in flash
- Using [TinyLlama](https://github.com/jzhang38/TinyLlama) variants quantized to Q2_K (~150MB)
- Running inference on a connected Raspberry Pi and sending results via serial/WiFi

---

## How to Train on Your Own Resources

### Step 1: Prepare Your Materials

Supported file formats:
- **PDF** — lecture slides, textbook chapters, research papers
- **DOCX** — notes, assignments, study guides
- **TXT** — plain text notes, code documentation

Organize your files by subject. More material = better results, but even 5-10 pages of notes work.

### Step 2: Open the Colab Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1s5mplU29DWxLB8bk7-ULm6AGQcuN2HPA)

### Step 3: Upload Your Files

When the upload widget appears (Step 3 in the notebook), select all your study materials. The notebook extracts text from each file automatically.

### Step 4: (Optional) Add Custom Q&A

In Step 5 of the notebook, you can add your own question-answer pairs:

```python
custom_qa = [
    {
        "question": "What are the three types of machine learning?",
        "answer": "Supervised learning (labeled data), unsupervised learning (unlabeled data), and reinforcement learning (reward-based)."
    },
    {
        "question": "Define overfitting.",
        "answer": "Overfitting is when a model learns the training data too well, including noise, and performs poorly on unseen data."
    }
]
```

### Step 5: Train and Export

Run the remaining cells. Training takes ~90 seconds on a free T4 GPU. The notebook:
1. Chunks your text into training examples
2. Auto-generates Q&A pairs from the chunks
3. Fine-tunes the model using LoRA (only trains 5% of parameters)
4. Exports to GGUF format for offline use

### Step 6: Download

The `.gguf` file downloads automatically at the end. Transfer it to your device.

---

## Customizing the Training

You can tweak these parameters in the notebook for different results:

| Parameter | Default | What it does |
|-----------|---------|--------------|
| `MAX_SEQ_LENGTH` | 2048 | Max token length per example (increase for longer answers) |
| `LORA_RANK` | 32 | LoRA rank (higher = more capacity, slower training) |
| `num_epochs` | 5 | Training epochs (more = better memorization, risk of overfitting) |
| `learning_rate` | 2e-4 | How fast the model learns (lower = more stable) |
| `QUANT_METHOD` | Q4_K_M | GGUF quantization (Q4_K_M is best balance of size/quality) |

### Quantization Options

| Method | File Size | Quality | Speed |
|--------|-----------|---------|-------|
| `q2_k` | ~180MB | Lower | Fastest |
| `q4_k_m` | ~350MB | Good | Fast |
| `q5_k_m` | ~430MB | Better | Medium |
| `q8_0` | ~530MB | Best | Slower |
| `f16` | ~1GB | Original | Slowest |

---

## Tips for Best Results

1. **More data = better answers** — upload everything: slides, notes, past papers, textbook chapters
2. **Add custom Q&A** for important formulas, definitions, and key concepts
3. **Use Q4_K_M** quantization — best tradeoff between file size and answer quality
4. **Keep questions specific** — "What is backpropagation?" works better than "Tell me about neural networks"
5. **Set temperature to 0.3-0.5** for factual answers, 0.7-0.9 for more creative responses
6. **The model is small (0.5B)** — it won't write essays, but it handles definitions, explanations, and short answers well

## Project Structure

```
cheat-offiline/
├── README.md              # This file
├── examples/
│   ├── sample_qa.json     # Example custom Q&A format
│   └── modelfile          # Ollama Modelfile template
└── LICENSE                # MIT License
```

## Requirements

- **Training**: Google account (for Colab free tier, T4 GPU)
- **Phone**: 2GB+ free storage, Android 8+ or iOS 14+
- **Laptop**: Any modern OS, 1GB+ free RAM
- **Raspberry Pi**: 4GB+ RAM model recommended

## License

MIT — use it however you want.

---

**Built by [Vivek Reddy](https://github.com/vvvvvivekkk)** | Fine-tune any course, run it anywhere, no internet needed.
