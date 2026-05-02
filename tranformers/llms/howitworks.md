    # How LLMs Work — My Notes

---

## 1. From Input to Tokens

When we type something, the model doesn't directly understand words. First comes **tokenization**.

```
"The cat sat on the mat"
    ↓
["The"] ["cat"] ["sat"] ["on"] ["the"] ["mat"]
```

Each word (or part of a word) becomes a **token**.

---

## 2. Embeddings — Number Vectors

The model can't directly understand tokens either. So each token is converted into a **number vector**.

```
"cat" → [0.2, -0.8, 1.4, 0.03, ...]  (thousands of numbers)
```

This vector represents the **meaning** of the word. Similar words have similar vectors.

---

## 3. Positional Encoding

Embeddings only carry the word's meaning — but not its **position** in the sentence.

- "Cat ate fish" and "Fish ate cat" have the same words but different meanings.

So each token's vector gets **position information added** to it.

```
"cat" at position 2 → embedding + position signal
```

---

## 4. Transformer Blocks — Where the Real Work Happens

These blocks repeat **N times** (~96 times in GPT-4). Data flows straight down — not in a loop.

### 4a. Multi-Head Attention

Each token **scans all other tokens** to understand context.

```
"bank" near "river" → riverbank
"bank" near "money" → financial bank
```

Attention decides — to understand this token, **which other tokens matter most**.

This doesn't happen just once — multiple "heads" look for different things simultaneously. That's why it's called **Multi**-Head Attention.

### 4b. Feed Forward Layer

Attention figured out the context. Now the Feed Forward layer **applies knowledge**.

- Attention = "Paris has 'capital of France' written near it"
- Feed Forward = "Paris has the Eiffel Tower, people speak French there"

All the model's **memory and knowledge** is stored in the Feed Forward weights — everything it read during training.

### One Block's Flow:

```
Input tokens
    ↓
[Block 1]
  Multi-Head Attention  →  tokens talk to each other
         ↓
  Feed Forward          →  knowledge gets applied
    ↓
[Block 2]
  Multi-Head Attention
         ↓
  Feed Forward
    ↓
... (N times)
    ↓
[Block N]
    ↓
Final representation
```

**Important:** The context doesn't change inside — the **representation gets deeper**.

```
"bank" → Block 1:  just a word
       → Block 5:  "near a river" figured out
       → Block 12: "riverbank, not financial" confirmed
       → Block 20: full nuanced meaning ready
```

---

## 5. Output — Next Token Prediction

After the transformer blocks:

```
Final representation
    ↓
Linear Layer
(scores for ~50,000 words)
    ↓
Softmax
(scores → probabilities, all adding up to 100%)
    ↓
mat   = 68%
floor = 18%
roof  =  8%
...
    ↓
Sampling → 1 token is chosen
```

**Temperature** controls how random the selection is:
- Low temperature → always picks the top word
- High temperature → creative and unpredictable

---

## 6. The Autoregressive Loop — How a Full Sentence Gets Built

One token comes out. It gets added back to the input. Then the next token. This loop continues.

```
Input:  "The cat sat on the mat"
Output: "."          ← 1st token

Input:  "The cat sat on the mat ."
Output: "The"        ← 2nd token

Input:  "The cat sat on the mat . The"
Output: "cat"        ← 3rd token

... until <EOS> token appears
```

**When does it stop:**
1. `<EOS>` token is generated — model decided "done"
2. `max_tokens` limit is hit
3. A stop sequence is encountered

**This is why longer responses are slower** — 100 tokens = 100 full transformer runs. No shortcuts.

---

## 7. Training Phase vs Inference Phase

### Training Phase

The model learns from **labeled data** — both input and expected output exist together in the dataset.

```
Dataset row:
  Input:    "How are you?"
  Expected: "I am fine <eos>"
```

#### Teacher Forcing

There was a problem in training — if the model predicts wrong and then predicts the next token based on that wrong output, **errors accumulate**.

**Solution:** Teacher Forcing — wrong prediction happened? Doesn't matter. **Force feed the expected answer** into the next step, not the model's output.

```
Step 1: Input = "How are you?" + <start>
        Model predicts → "We"  ← wrong!
        Loss calculated

Step 2: Input = "How are you?" + <start> + "I"   ← "We" ignored, "I" forced
        Model predicts → "am"

Step 3: Input = "How are you?" + <start> + "I am"
        Model predicts → "fine"

Step 4: Input = "How are you?" + <start> + "I am fine"
        Model predicts → <eos>  ✓
```

Teacher forcing makes training **fast and stable** because all tokens get processed in parallel.

#### Full Training Loop

```
Training Data (Input + Expected Output)
    ↓
Forward Pass
  ├── Input + Expected answer fed together (teacher forcing)
  └── Transformer blocks (Attention + Feed Forward)
    ↓
Model's Predicted Output
(a probability distribution at each position)
    ↓
Loss Calculated
(predicted vs expected — how wrong was it?)
    ↓
Backpropagation
(loss travels backwards — each weight's "blame" is figured out)
    ↓
Weights Updated
(gradient descent — small adjustments)
    ↓
↻ Repeat billions of times
    ↓
Loss → 0 — Model trained!
```

#### What is Loss?

If the model said `"I" = 72%` and the expected answer was `"I"` — small loss.
If the model said `"I" = 12%` — large loss.

Loss is calculated **at every token position simultaneously**:

```
Position 1: predicted "I"    vs expected "I"    → loss1
Position 2: predicted "am"   vs expected "am"   → loss2
Position 3: predicted "fine" vs expected "fine" → loss3
Position 4: predicted <eos>  vs expected <eos>  → loss4

Total Loss = loss1 + loss2 + loss3 + loss4
```

#### What does Backpropagation do?

It sends the loss **travelling backwards** — all the way to each weight.
It tells each weight — "how responsible were you for this mistake?"
Then gradient descent adjusts that weight slightly.

### Inference Phase

```
Only Input → Model → Generate output (one token at a time)
```

- No expected output
- No loss
- No backprop
- Weights are fixed — only forward pass

**Training vs Inference:**

| | Training | Inference |
|---|---|---|
| Input | Question + Expected Answer | Only the Question |
| Processing | Teacher Forcing (parallel) | Autoregressive (one token at a time) |
| After output | Loss → Backprop → Weight update | Directly output |
| Speed | Fast (parallel) | Slow (sequential) |
| Weights | Get updated | Fixed |

---

## 8. Encoder-Decoder vs Decoder-Only

### Encoder-Decoder Architecture (Classic — original 2017 Transformer)

```
"Hi, How are you?"
        ↓
    Encoder
    (understand the input, build a context vector)
        ↓
    Context Vector
        ↓
    Decoder  ←  <start> I am fine (expected output)
    (two types of attention happen here)
        ↓
    <EOS>
```

**Two types of Attention in the Decoder:**
1. **Self Attention** — tokens within "I am fine" talk to each other
2. **Cross Attention** — decoder's tokens talk to the encoder's context ("how does my answer relate to the input?")

Cross Attention was important — the decoder always knew what the input contained.

**Best for:** Translation, Summarization (clearly separate input and output)

### Decoder-Only Architecture (Modern — GPT, Claude, LLaMA)

```
"Hi How are you? <start> I am fine"
           ↓
        Decoder only
        (everything in one place)
           ↓
         <EOS>
```

No separate encoder. Input and expected output fed together.
No cross attention — one attention handles everything.

**Best for:** Chat, Generation, Coding

### Comparison

| | Encoder-Decoder | Decoder-Only |
|---|---|---|
| Examples | T5, BART, original Transformer | GPT, Claude, LLaMA |
| Training | Input in encoder, expected output in decoder separately | Everything together |
| Attention | Self + Cross Attention | Only Self Attention |
| Best for | Translation, summarization | Chat, generation |

---

## Quick Summary — The Full Flow

```
INPUT TEXT
    ↓
Tokenization          → split words into tokens
    ↓
Embeddings            → convert tokens into number vectors
    ↓
Positional Encoding   → add position information
    ↓
[Transformer Block × N]
  Multi-Head Attention → tokens understand context from each other
  Feed Forward        → knowledge gets applied
    ↓
Linear + Softmax      → get probabilities (over 50k words)
    ↓
Sample 1 Token        → choose based on temperature
    ↓
↻ Token added back to input — repeat until <EOS>
```

---

*These notes are based on a piyush garg video and my conversation with claude*