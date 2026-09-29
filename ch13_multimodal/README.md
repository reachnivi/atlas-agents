# Chapter 13 — Multimodal and Voice Agents

> **Atlas v0.13** — a document analyst that **routes each input type to the model best at it** (images → Claude vision, PDFs → Gemini native PDF, text → Claude), plus real-time phone voice agents.

## TL;DR

Real inputs aren't only text: screenshots, scanned PDFs, slide decks, phone calls. Two ideas cover most of it:

1. **Modality routing**: detect the input type and send it to the model/pipeline that handles it best, with a fallback.
2. **Streaming voice bridges**: a phone call is a WebSocket of audio frames. Convert codecs and sample rates in both directions and let a realtime model (OpenAI Realtime or Gemini Live) handle speech-to-speech, including callers interrupting mid-sentence (barge-in).

## Key concepts

| Concept | What it means | Where to see it |
|---|---|---|
| **Modality router** | `image` → Claude vision, `pdf` → Gemini 2.5 Flash, `text` → Claude. | `route_and_analyze()` |
| **Vision input** | Base64-encode the image with its MIME type in a content block. | `analyze_image()` |
| **Fallback path** | If Gemini isn't available, extract PDF text with `pdftotext` and send it to Claude. | `analyze_pdf_fallback()` |
| **Page-as-image** | For visual PDFs (scans, slides, financial tables), render each page to PNG and use vision, then synthesize. | `online/batch_pdf_analyst.py` |
| **Visual diffing** | Compare a design export with a rendered screenshot and get concrete CSS fixes. | `online/screenshot_debugger.py` |
| **Telephony audio** | Twilio streams PCMU @ 8 kHz. OpenAI Realtime wants PCM16 @ 24 kHz, and Gemini Live wants PCM16 @ 16 kHz. Resample both ways. | `online/voice_agent_twilio.py`, `online/gemini_live_agent.py` |
| **Barge-in** | When the caller interrupts, cancel the model's current response (OpenAI: explicit cancel; Gemini: server sets `interrupted=True`). | voice agents |
| **Voice tool calls** | Realtime models can call tools mid-conversation. The server executes them and returns results. | `_execute_tool()` |

## Files

| File | What it shows |
|---|---|
| `multimodal_agent.py` | The chapter project: image/PDF/text routing with fallback. |
| `online/batch_pdf_analyst.py` | Page-by-page vision analysis of complex PDFs (needs poppler). |
| `online/screenshot_debugger.py` | Expected vs. actual screenshot → list of CSS fixes. |
| `online/voice_agent_twilio.py` | FastAPI bridge: Twilio Media Stream ↔ OpenAI Realtime API. |
| `online/gemini_live_agent.py` | The same bridge using the Gemini Live API. |

## What needs to be done

- [ ] `pip install anthropic google-genai`, and set `ANTHROPIC_API_KEY` and `GOOGLE_API_KEY`.
- [ ] Analyze a screenshot with an error in it, a PDF report, and a text file of notes.
- [ ] Remove `GOOGLE_API_KEY` and confirm the PDF fallback path still works.
- [ ] Run `batch_pdf_analyst.py` on a slide deck and compare it with the text-extraction result.
- [ ] (Stretch) Expose the voice agent with a tunnel, point a Twilio number's Voice URL at `/twiml`, and call it.

## Run it

```bash
cd ch13_multimodal
python multimodal_agent.py image screenshot.png "What errors do you see?"
python multimodal_agent.py pdf   report.pdf     "Summarize the key findings"
python multimodal_agent.py text  notes.txt      "What are the action items?"

uvicorn online.voice_agent_twilio:app --port 8080   # or gemini_live_agent:app
# Python 3.13+: pip install audioop-lts
```

## Production notes

- Images and PDF pages are expensive in tokens. Cap page counts (`MAX_PAGES`) and resolution.
- Voice latency budgets are tight (a few hundred ms). Keep tools fast or answer with filler speech first.
- `audioop` was removed in Python 3.13, so install `audioop-lts`.

## Takeaways

1. Route by modality and don't assume one model is best at everything.
2. When text extraction loses the layout, send the page as an image.
3. Voice agents are mostly an audio pipeline problem: codecs, sample rates, streaming, interruption.

**Prev:** [Ch. 12](../ch12_sandboxes/README.md) · **Next:** [Chapter 14 — Guardrails and Agent Safety](../ch14_guardrails/README.md)
