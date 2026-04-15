# OmniCX – Omnichannel Customer Support AI with Agentic Workflow

## 1. Background / Context
Customer service bisnis saat ini terpecah: WhatsApp, email, live chat, telepon, dan media sosial. Agen harus buka banyak tab, copy-paste jawaban, dan tidak ada memori lintas kanal. Pelanggan frustrasi karena harus mengulang cerita.

## 2. Problem Statement
- Respon time lambat (bisa >1 jam di email/WA).
- Tidak ada personalisasi karena agen tidak tahu riwayat customer dari kanal lain.
- Biaya operasional CS besar (gaji agen untuk pertanyaan repetitif).
- Pelanggan marah ketika harus menjelaskan masalah berulang kali.

## 3. Goals / Objectives
- Menurunkan Average Handle Time (AHT) dari 8 menit menjadi 2 menit.
- Mengotomatisasi 70% pertanyaan umum (FAQ, tracking order, reset password).
- Menjaga konsistensi jawaban di semua kanal (WA, web chat, email).
- Memberikan handoff mulus ke agen manusia jika perlu (dengan konteks lengkap).

## 4. Proposed Solution
**Agentic AI Platform** menggunakan LLM (GPT-4o / Llama 3) dengan tool calling. Sistem memiliki:
- Memori lintas kanal (customer 360 view).
- Tool integration: cek order, update tiket, lookup knowledge base.
- Autonomous decision: jika confidence rendah, transfer ke human.

## 5. System Design / Architecture
[Channels: WhatsApp, Telegram, Web Chat, Email]
↓
[Unified Ingress] → [Message Queue (Kafka)]
↓
[Orchestrator] (LangChain / LangGraph)
├── [Intent Classifier] (small BERT)
├── [Agent with Tools] (LLM)
│ ├── Tool: get_order_status
│ ├── Tool: search_kb (RAG)
│ ├── Tool: create_ticket
│ └── Tool: transfer_to_human
└── [Response Generator]
↓
[Channel Egress] → Customer

**Database:** PostgreSQL (customer, order, ticket), Vector DB (Pinecone) untuk KB.

## 6. Detailed Design
**Agentic Workflow (LangGraph):**
1. Customer bertanya "Pesanan saya #123 di mana?"
2. Intent classifier → `track_order`.
3. Agent memanggil tool `get_order_status(order_id=123)` → API ke sistem ERP.
4. Jika order tidak ditemukan, agent akan tanya lagi: "Mohon cek nomor pesanan".
5. Jika customer marah (deteksi sentiment negatif tinggi), agent langsung transfer ke human + lampirkan chat history.

**RAG untuk knowledge base:**
- Dokumen FAQ, panduan produk di-chunk dan di-embed.
- Setiap pertanyaan, retrieve top 3 chunk relevan sebagai konteks LLM.

**Handoff ke manusia:**
- Human agent masuk ke dashboard yang sama, melihat full history.
- Setelah selesai, AI bisa belajar dari solusi human (feedback loop).

**Edge cases:**
- Customer mengirim pesan panjang bertele-tele: agent akan meringkas dan mengkonfirmasi.
- Bahasa campuran (Inggris-Indonesia): LLM handle dengan baik.
- Tool gagal karena API down: agent akan bilang "Sedang gangguan teknis, akan saya teruskan ke tim".

## 7. Trade-offs & Alternatives
| Alternatif | Kelebihan | Kekurangan |
|------------|-----------|-------------|
| Rule-based chatbot (Rasa) | Deterministik, murah | Tidak bisa handle pertanyaan di luar rule |
| Pure LLM tanpa tools | Fleksibel | Halusinasi, tidak bisa akses data real-time |

**Dipilih Agentic (LLM + tools)** karena bisa melakukan aksi nyata (cek order, buka tiket) dan lebih pintar.

## 8. Risks & Mitigation
| Risiko | Mitigasi |
|--------|-----------|
| LLM hallucination tentang data order | Selalu validasi output tool, jangan percaya LLM untuk data faktual tanpa tool |
| Biaya LLM mahal | Gunakan small model untuk intent classification, LLM hanya untuk reasoning kompleks |
| Customer kesulitan bicara dengan bot | Beri opsi "Bicara dengan manusia" di setiap pesan |

## 9. Metrics / Evaluation
- **Automation rate**: % percakapan yang selesai tanpa human.
- **CSAT** (customer satisfaction) untuk AI-only: minimal 4.2/5.
- **Handoff rate**: <30% percakapan perlu human.
- **Latency end-to-end**: < 3 detik untuk 90% pesan.

## 10. Implementation Plan
- [ ] Minggu 1: Setup LangGraph + koneksi ke WhatsApp Business API & Telegram
- [ ] Minggu 2: Implement 3 tools dasar (get_order, search_kb, transfer_human)
- [ ] Minggu 3: RAG pipeline dengan dokumen contoh (10 halaman FAQ)
- [ ] Minggu 4: Human dashboard (React) untuk monitoring & takeover
- [ ] Minggu 5: UAT dengan 1 bisnis retail (100 percakapan per hari)
- [ ] Minggu 6: Fine-tune prompt & evaluation

**Tech stack:** Python (LangChain, LangGraph), FastAPI, PostgreSQL, Pinecone, Redis, Docker, LLM via Together.ai / OpenAI.
**Kompleksitas:** Hard
**Role:** Full Stack AI Developer