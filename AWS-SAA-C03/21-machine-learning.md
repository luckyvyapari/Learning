# 21 — AWS Machine Learning Services

One-line mapping is what the exam needs. Know **input → output** of each.

| Service | What it does | Key points |
|---|---|---|
| **Rekognition** | **Image + video** analysis: objects, people, text, scenes | Face detection/analysis (age range, emotions), **face search & verification**, celebrity recognition, **pathing** (sports), labeling, text detection |
| — Content Moderation | Detect inappropriate/offensive content | Set **minimum confidence threshold**; flag for manual review in **Amazon Augmented AI (A2I)**; social media, broadcast, ads, e-commerce, compliance |
| **Transcribe** | **Speech → text** (ASR, deep learning) | **Redact PII**, automatic language identification; call transcripts, captions/subtitles, searchable media |
| **Polly** | **Text → speech** | **Lexicons** (custom pronunciation: acronyms, stylized words) via `SynthesizeSpeech`; **SSML** (emphasis, phonetics, whisper, breathing, Newscaster style) |
| **Translate** | Language translation | Localize websites/apps at scale |
| **Lex** | Chatbots (same tech as **Alexa**) | **ASR** (speech→text) + **NLU** (intent recognition) |
| **Connect** | Cloud **contact center** | Receive calls, contact flows, integrates CRM; **no upfront cost, ~80% cheaper**. Typical flow: Connect → Lex → Lambda → CRM |
| **Comprehend** | **NLP**: serverless text insights | Language, key phrases/people/places/brands, **sentiment**, tokenization/parts of speech, topic modeling |
| **Comprehend Medical** | Clinical text (notes, discharge summaries, test results) | Detects **PHI** (`DetectPHI`); docs in S3, real-time via Firehose, or narratives via Transcribe |
| **SageMaker AI** | **Build/train/tune/deploy your own ML models** | Fully managed for developers/data scientists |
| **Kendra** | **ML document search** (natural language) | Answers from PDF/HTML/Word/PowerPoint/FAQs; sources S3, RDS, Google Drive, SharePoint, OneDrive; learns from feedback; tunable ranking |
| **Personalize** | Real-time **personalized recommendations** | Same tech as Amazon.com; reads from S3 + real-time data; website, apps, SMS, email; days not months |
| **Textract** | Extract **text, handwriting, forms, tables** from scanned documents | PDFs/images; invoices, medical records, tax forms, IDs/passports |

## Summary one-liners

- Rekognition: face detection, labeling, celebrity recognition
- Transcribe: audio → text (subtitles)
- Polly: text → audio
- Translate: translations
- Lex: conversational bots
- Connect: cloud contact center
- Comprehend: NLP
- SageMaker: ML for every developer/data scientist
- Kendra: ML-powered search engine
- Personalize: recommendations
- Textract: text/data from documents

## Exam Hints

- "Detect objects / faces / inappropriate images" → **Rekognition**
- "Convert call recordings to text, hide PII" → **Transcribe (redaction)**
- "Chatbot / call center bot" → **Lex** (+ **Connect**)
- "Sentiment of customer emails" → **Comprehend**
- "Search company documents with natural language" → **Kendra**
- "Extract fields from scanned forms/invoices" → **Textract**
- "Recommendations like Amazon.com" → **Personalize**
- "Train my own model" → **SageMaker**
