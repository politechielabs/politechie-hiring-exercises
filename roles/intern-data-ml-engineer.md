# An exercise in TTS dataset preparation

This is a small exercise covering the kind of work you will do in this role.
For any ML engineer, the data used for training matters as much as the model architecture: garbage in, garbage out.
We are training a text-to-speech (TTS) model, and the goal of this exercise is to prepare a **high-quality, single-speaker audio dataset** for it.

**Time budget:** a few hours a day over 3 days. We expect most people to finish sooner.

## Evaluation criteria
The tasks are not hard on their own. What we look for is how much attention you give to the small things that improve overall quality, and how you build the system.

- **Attention to detail.** Small decisions add up: choices on cleaning and preprocessing , chunk boundaries, text normalization, edge cases. How much you care about details shows us how much you can own the work you're given.
- **Data quality judgement.** What you keep, what you drop, and why. How you checked that your output is actually good.
- **Idempotency.** Running a step again should give the same result, should not redo expensive steps unnecessarily, and should not corrupt existing outputs.
- **Modularity.** Each stage (collection, cleaning, transcription, chunking, preprocessing, analysis) should run on its own, with clear inputs and outputs and configurable parameters.
- **Reproducibility.** Someone else should be able to rebuild your dataset by following your README.
- **Reasoning.** Write down your decisions and the trade-offs you considered.
- **Communication.** Send frequent progress updates (commits plus short notes) and keep your README up to date as you go, not a dump on the last day. This shows us how you work, and lets us help if you're stuck.
- **Clarifications.** Reach out if you have any doubts about the deliverables or implementation. Don't just assume, and don't sit on it.
- **Prioritization.** Get the pipeline working end-to-end first, then improve quality. We value all tasks done reasonably well over a few tasks done perfectly.

---

## Audio and data specification
- Target audio format: **24 kHz, mono, 16-bit PCM WAV**.
- One target speaker. The final dataset should contain only that speaker's voice.
- Final manifest: a JSONL file, one line per chunk, with at least these fields:
  ```json
  {"audio_path": "data/chunks/<file>.wav", "audio_text": "<transcript of this chunk>", "duration": 4.82}
  ```
  You may add any other fields you find useful.

## Task 1 - Data collection
Pick a popular Hindi speaker on YouTube as your target speaker.
Collect 4-5 videos of that speaker, each around 5-10 minutes long.

Choose the videos so they cover the problems the cleaning step must handle:
- background music
- reverb / room echo
- speakers other than the target speaker (interviewers, interruptions, overlapping speech)

List your sources in `data/sources.csv` with, at minimum, the URL, title, duration, and the problems present in each video.

## Task 2 - Cleaning
The collected audio is noisy: background music, reverb, and other speakers interrupting.
Clean it so that what remains is usable TTS training data of the target speaker only.

- You are free to choose your tools and models.
- Decide when audio should be repaired and when it should be dropped.
- Report what you removed or changed, and how you verified the result.

## Task 3 - Transcription
Generate transcriptions for the cleaned audio using a suitable ASR.
If CPU-hosted ASR is too slow, you are free to use third-party ASR APIs that offer free credits (for example Deepgram or Speechmatics).

Note which ASR you used and why, and any problems you noticed in its output.

## Task 4 - Chunking and manifest creation
A TTS model needs audio in chunks, each with its matching text.
Think carefully about **how** you chunk and **what chunk sizes** you target. 
Produce the manifest described in the specification above, with `audio_path` pointing to each chunked audio file and `audio_text` holding its corresponding text.

## Task 5 - Preprocessing, normalization and analysis

### 5.1 Preprocessing and normalization
Decide whether any preprocessing is needed to further improve the quality of the manifest from Task 4.
This can apply to the text, the audio, or both.
For each step you apply, explain how it helps the TTS model that will be trained on this manifest.

### 5.2 Stats and distribution analysis
Decide which factors would help someone understand the final dataset, and analyse them.
For example, you might want to find code-mixing cases.

## Scaling up (no implementation)
Suppose we want a dataset of 60-70 hours instead of the ~30-50 minutes in this exercise.
What changes would your setup need? We will discuss this on a call.

---

## Deliverables
- A pipeline in `./src/` that runs end-to-end, from the source list to the final manifest.
- `data/sources.csv`, the final manifest, stats from task 5.2 and around 10 sample chunks.

## Repository requirements
- Do not fork this repo. Work on a private repo and add collaborators: `shashank-srikant`, `dipesh517`, `ayonghan`.
- Once you have created the repo, share the link and mention that you've started the task. Reply-all when responding to emails on the thread.
- Use **Python**, with a `conda` environment.
- Folder layout:
  - `./src/`: source code.
  - `./env/`: the Conda environment file.
  - `./data/`: the data your source accesses. Audio files are too large for git: commit the source list, the manifest, and the sample chunks only, and make sure your scripts can regenerate the rest.
  - `./tests/`: a few unit tests (`unittest`) for the deterministic parts of your pipeline. We care more about what you choose to test than how many tests you write.
- Use a `.gitignore` file so you don't add junk files to the repo.
- `README.md`:
  - The topmost section must be titled `Approach`: your overall strategy, the key tools you considered, how you evaluated the quality of your dataset, and your key decisions with their trade-offs.
  - Include reproducible setup and run instructions.
  - **Keep it updated after every task, not just at the end.** We'd rather see decisions recorded in the moment than reconstructed at the end.
- If you use Claude, Copilot, ChatGPT or similar tools, keep all your prompts in a version-controlled `prompts.md`, so we can see how they evolved over time.
- Commits should show incremental progress; we read the history.
- Use the collected audio only for this exercise. Do not publish the audio, the dataset, or any model trained on it.

---

**All the best!**
