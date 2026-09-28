# Indonesian News Summarizer with NER

Final project for COMP6885001 Natural Language Processing, Binus University 2025/2026.

This system takes a URL from an Indonesian news portal, scrapes the article content, generates an abstractive summary, and identifies named entities such as people, organizations, and locations mentioned in the article.

There is a demo video at https://youtu.be/punOeU7rH1M.

## Team

- Christopher Setyawan (2802459670)
- Gracias Kumara Winata (2802459683)
- Vincensius Kevin Mulyono (2802476424)

## Models

| Component | Model |
|---|---|
| Summarization | cahya/bert2gpt-indonesian-summarization, used as released (it scored higher than both of our fine-tunes, see Evaluation) |
| Named Entity Recognition | cahya/bert-base-indonesian-NER |

## Project Structure

```
news_summarizer/
├── backend/
│   └── main.py            - FastAPI backend
├── frontend/
│   ├── index.html         - Landing page
│   └── app.html           - Main app interface
├── models/
│   ├── summarizer.py      - Summarization pipeline
│   ├── ner.py             - NER pipeline
│   └── finetuned_liputan6 - Fine-tuned model weights (not included, see below)
├── scraper/
│   └── news_scraper.py    - Web scraper for Indonesian news portals
├── app.py                 - Streamlit version (legacy)
├── finetune.py            - Fine-tuning script for IndoSum
├── finetune_liputan6.py   - Fine-tuning script for Liputan6
├── evaluate_liputan6.py   - ROUGE on Liputan6, including the LEAD-2 baseline
├── evaluate_rouge.py      - ROUGE on IndoSum
├── evaluate_lead3.py      - LEAD-3 baseline on IndoSum
├── evaluate_ner.py        - NER evaluation on NERGrit
└── requirements.txt
```

## How to Run

Install dependencies:

```bash
pip install -r requirements.txt
```

Start the server:

```bash
uvicorn backend.main:app --port 8000
```

Open your browser at `http://localhost:8000`.

## Supported News Portals

- detik.com
- kompas.com
- tribunnews.com
- tempo.co
- Other portals via generic parser

## Evaluation Results

Summarization evaluated on Liputan6 canonical test set (200 samples):

| Model | ROUGE-1 | ROUGE-2 | ROUGE-L |
|---|---|---|---|
| LEAD-2 Baseline | 32.11 | 18.88 | 26.86 |
| BERT2GPT (no fine-tune) | 39.23 | 21.47 | 31.95 |
| BERT2GPT + IndoSum | 35.83 | 18.96 | 29.86 |
| BERT2GPT + Liputan6 | 35.83 | 18.96 | 29.86 |

NER evaluated on IndoNLU NERGrit test set (200 samples):

| Model | Precision | Recall | F1 |
|---|---|---|---|
| IndoBERT-NER | 0.1774 | 0.3353 | 0.2320 |

The lower NER score is expected due to label schema mismatch between the model (PER/ORG/LOC/GPE/DATE/EVT) and the NERGrit test set (PERSON/ORGANISATION/PLACE).

### Notes on the evaluation

The model we ship was not fine-tuned by us, and it had an advantage in this test. Its model card says `cahya/bert2gpt-indonesian-summarization` was trained on `id_liputan6`, so evaluating it on Liputan6 is in-domain. That is most of the reason it beats our IndoSum fine-tune. We kept it because it scored best, not because it was ours.

Rows 3 and 4 are identical to two decimals on all three metrics. That most likely means the same checkpoint got evaluated twice, so treat them as one result until the Liputan6 fine-tune is re-run on its own.

There is no IndoSum score here on purpose. An early run came out around ROUGE-1 91 on IndoSum, but its reference summaries are mostly sentences copied straight from the article, so any model that copies scores close to perfect. That number says more about the dataset than about the model. The comparison worth reading is Liputan6 against the LEAD-2 baseline above.

The NER score is low because of the evaluation as much as the model. Ours outputs PER, ORG, LOC, GPE, DATE and EVT while NERGrit expects PERSON, ORGANISATION and PLACE, so correct entities get counted as wrong. Mapping the labels before scoring is the next thing to fix.

## Datasets

- Liputan6: https://github.com/fajri91/sum_liputan6
- IndoSum: https://www.kaggle.com/datasets/linkgish/indosum
- IndoNLU NERGrit: https://huggingface.co/datasets/indonlp/indonlu
