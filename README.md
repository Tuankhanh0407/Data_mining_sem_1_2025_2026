# Phân Tích Dự Án: AI-Powered Presentation Generator

## 1. AI FRAMEWORKS ĐƯỢC SỬ DỤNG

### Core AI/ML Frameworks

| Thành Phần | Framework | Mục Đích |
|-----------|-----------|---------|
| **Retrieval-Augmented Generation** | Custom RAG Pipeline | Kết hợp thông tin từ trích xuất với sinh nội dung |
| **Embedding Model** | Sentence Transformers (paraphrase-multilingual-MiniLM-L12-v2) | Chuyển văn bản thành vector ngữ nghĩa |
| **Vector Database** | FAISS (Facebook AI Similarity Search) | Tìm kiếm tương đồng hiệu quả với kích thước lớn |
| **Information Retrieval** | Dense Semantic Search | Trích xuất top-k tài liệu liên quan nhất |
| **Prompt Engineering** | Advanced Prompting Techniques (Few-shot, Chain-of-Thought) | Hướng dẫn LLM sinh nội dung có cấu trúc |
| **NLP Processing** | Vietnamese Text Normalization | Xử lý tiếng Việt: chuẩn hóa, phân đoạn, phân tách |
| **Text-to-Speech** | gTTS (Google Text-to-Speech) | Chuyển ghi chú thành giọng nói tiếng Việt |

### Supporting Technologies

- **Presentation Generation**: python-pptx (lập trình hóa file PowerPoint)
- **Data Processing**: Pandas, NumPy (xử lý dữ liệu đại quy mô)
- **API Development**: Flask (tạo REST endpoints)
- **Analytics**: Apache Superset (visualize metrics)
- **CMS**: Drupal (tích hợp quản lý nội dung)

---

## 2. CHI TIẾT KIẾN TRÚC HỆ THỐNG

### Data Flow Architecture

```
1. DATA INGESTION
   ↓
   Nguồn: UIT ViNews + Synthetic Data (100-150 documents)
   
2. DATA PREPROCESSING
   ↓
   - Chuẩn hóa văn bản tiếng Việt (dấu thanh, khoảng trắng)
   - Phân đoạn câu và chunk (350-400 ký tự)
   - Kiểm tra chất lượng (null values, duplicates)
   
3. EMBEDDING & INDEXING
   ↓
   - Embedding với Sentence Transformers (384 chiều)
   - Lập chỉ mục FAISS (IndexFlatIP)
   - Lưu trữ metadata
   
4. RAG RETRIEVAL
   ↓
   - User input: topic, intent, audience
   - Query embedding
   - Top-k similarity search
   - Trích xuất 8-10 chunk liên quan
   
5. PROMPT ENGINEERING & GENERATION
   ↓
   - Tạo prompt với Few-shot examples
   - Áp dụng Chain-of-Thought reasoning
   - Sinh nội dung structured (JSON)
   
6. OUTPUT GENERATION
   ↓
   ├─ PowerPoint (.pptx) → python-pptx
   ├─ Speaker Notes
   └─ Audio MP3 (Vietnamese TTS) → gTTS
```

### RAG Pipeline Components

1. **Retriever (Bộ trích xuất)**
   - Nhận: User query
   - Chuyển thành vector embedding (384-dim)
   - Tìm top-k chunks từ FAISS
   - Trả về: Danh sách tài liệu liên quan

2. **Generator (Bộ sinh)**
   - Nhận: Query + Retrieved context + Prompt template
   - Áp dụng prompt engineering (few-shot, chain-of-thought)
   - Sinh: Nội dung slide (tiêu đề, bullet points, ghi chú)
   - Định dạng: JSON structured output

3. **Knowledge Base (Cơ sở tri thức)**
   - Lưu trữ: 100-150 Vietnamese documents
   - Format: JSONL (line-delimited JSON)
   - Chunks: ~400-500 fragments
   - Metadata: chủ đề, mục đích, đối tượng

---

## 3. PROMPT ENGINEERING TECHNIQUES

### Kỹ Thuật Được Áp Dụng

**Few-shot Learning**
- Cung cấp 2-3 ví dụ về cấu trúc slide mong muốn
- Giúp mô hình hiểu pattern và phong cách sinh nội dung
- Tăng chất lượng output và consistency

**Chain-of-Thought Reasoning**
- Hướng dẫn mô hình suy nghĩ từng bước
- "Đầu tiên... Tiếp theo... Cuối cùng..."
- Cải thiện logic và độ chính xác của nội dung

**Structured Prompts**
- Template rõ ràng về input/output
- Định dạng JSON cho output structured
- Tách biệt thành các phần: tiêu đề, ý chính, ghi chú

### Ví Dụ Prompt Template

```
Topic: {topic}
Intent: {intent}
Audience: {audience}

Based on the following retrieved documents:
{retrieved_context}

Generate a presentation slide with:
1. Clear title (5-10 words)
2. 3-5 bullet points (key concepts)
3. Detailed speaker notes (120-180 words)

Format as JSON:
{
  "title": "...",
  "bullets": ["...", "...", "..."],
  "speaker_notes": "..."
}

Examples:
[Example 1...]
[Example 2...]
```

---

## 4. VIETNAMESE LANGUAGE SUPPORT

### Xử Lý Tiếng Việt

1. **Text Normalization**
   - Chuẩn hóa dấu thanh (tonal marks)
   - Xử lý Unicode Normalization Form C (NFC)
   - Chuẩn hóa khoảng trắng và dấu câu

2. **Multilingual Embeddings**
   - Mô hình: paraphrase-multilingual-MiniLM-L12-v2
   - Hỗ trợ 50+ ngôn ngữ (including tiếng Việt)
   - Embedding chung cho phép tương đồng cross-lingual

3. **Sentence Segmentation**
   - Phân đoạn theo dấu câu: . ? ! ; :
   - Xử lý viết tắt phổ biến tiếng Việt
   - Optimized chunk size: 350-400 ký tự

4. **Text-to-Speech**
   - Sử dụng gTTS (Google Text-to-Speech)
   - Hỗ trợ đầy đủ diacritics tiếng Việt
   - Output: MP3 audio files

---

## 5. HIỆU SUẤT HỆ THỐNG

### Performance Metrics

| Giai Đoạn | Thời Gian | Ghi Chú |
|----------|-----------|--------|
| Build Dataset | 0.434 giây | 120 documents, chuẩn hóa + phân đoạn |
| Create Index | 38.429 giây | Embedding + FAISS indexing |
| RAG Generation | 6.559 giây | Query embedding, retrieval, content generation |
| PowerPoint Creation | 0.314 giây | Formatting + slide generation |
| TTS Synthesis | 97.184 giây | Audio generation (9 slides × ~11 giây/slide) |
| **TOTAL** | **142.92 giây** | End-to-end pipeline |

### System Requirements

- **CPU**: Intel Core i5 / AMD Ryzen 5 equivalent
- **RAM**: 8GB DDR4 (4-5GB sử dụng)
- **Storage**: SSD 500MB/s+ (cho model cache)
- **GPU**: Không cần (CPU-based inference)
- **Network**: 10Mbps (tải model lần đầu)

### Chất Lượng Đầu Ra

- **Retrieval Quality**: 4.3/5.0 (cosine similarity score)
- **Presentation Structure**: 5-10 slides + title slide
- **Speaker Notes**: 120-180 từ/slide
- **Citation Accuracy**: 100% (from retrieved documents)

---

## 6. KIẾN TRÚC MODULAR & DATAOPS

### Module Design

1. **Data Module** (data_pipeline.py)
   - Thu thập, chuẩn hóa, phân đoạn dữ liệu
   - Quality checks tại mỗi bước
   - Error recovery mechanisms

2. **Embedding Module** (embedding_index.py)
   - Tải Sentence Transformer model
   - Tạo embeddings batch-wise
   - Xây dựng FAISS index

3. **RAG Module** (rag_generator.py)
   - Tạo query embedding
   - Thực hiện semantic search
   - Sinh nội dung với prompt engineering

4. **Presentation Module** (presentation_generator.py)
   - Tạo structure slide (title, content)
   - Sinh speaker notes
   - Tạo file PowerPoint (.pptx)

5. **Audio Module** (audio_synthesizer.py)
   - Chuyển text thành MP3
   - Xử lý Vietnamese pronunciation
   - Merge audio files

6. **Logging Module** (bilogging.py)
   - Ghi metrics CSV
   - Tích hợp Apache Superset
   - Performance monitoring

### DataOps Best Practices

✓ Automated quality checks  
✓ Error recovery & resilience  
✓ Comprehensive logging  
✓ Performance monitoring  
✓ Modular architecture  
✓ Scalability (supports larger datasets)  
✓ Open-source stack (zero licensing costs)  

---

## 7. TÍCH HỢP VỚI EXTERNAL SYSTEMS

### Drupal CMS Integration
- REST API endpoints (Flask)
- Quản lý nội dung centralized
- Phân phối presentation files

### Apache Superset Analytics
- CSV-based metrics logging
- Real-time dashboards
- Performance analysis & optimization

---

## 8. NGÔN NGỮ & CÔNG NGHỆ CHÍNH

```
Programming Language:  Python 3.13.5
Core Libraries:
  - sentence-transformers      (embedding)
  - faiss-cpu                 (vector search)
  - python-pptx               (PowerPoint)
  - gTTS                      (text-to-speech)
  - Flask                     (API)
  - numpy, pandas             (data processing)
  - requests                  (HTTP client)

Data Format:
  - JSONL                     (raw documents)
  - CSV                       (metrics logging)
  - NumPy arrays              (embeddings)
  - .pptx                     (presentations)
  - .mp3                      (audio output)
```

---

## 9. ĐÓNG GÓP KHOA HỌC & THỰC TIỄN

### Khoa Học
- ✓ RAG implementation cho tiếng Việt (low-resource language)
- ✓ Advanced prompt engineering cho content generation
- ✓ DataOps pipeline từ data ingestion đến output

### Thực Tiễn
- ✓ Giải quyết plagiarism problem (42.6% plagiarism rate in VN education)
- ✓ Tiết kiệm thời gian: giảm 50% công sức sinh biên dùng
- ✓ Nâng cao chất lượng: nội dung dựa trên source credible
- ✓ Open-source & cost-effective

---

## 10. HẠN CHẾ VÀ HƯỚNG PHÁT TRIỂN

### Hạn Chế Hiện Tại
- Dataset nhỏ (100-150 docs) → cần mở rộng sang các lĩnh vực khác
- gTTS chất lượng tiêu chuẩn → cần premium TTS (ElevenLabs, Azure)
- Không hỗ trợ hình ảnh/biểu đồ → cần thêm image generation

### Phát Triển Tương Lai
1. **Advanced RAG**: Hierarchical retrieval, iterative refinement, knowledge graphs
2. **Stronger LLM**: Integrate GPT-4, Claude, Llama-based models
3. **Multimodal**: Image search, diagram generation, data visualization
4. **Web UI**: User-friendly interface, drag-and-drop content management
5. **Premium Audio**: Better pronunciation & voice variety
6. **Plagiarism Detection**: Academic integrity verification
7. **Expanded Domains**: Healthcare, legal, scientific fields

---

## SUMMARY

**Dự án này sử dụng:**
- ✅ **RAG (Retrieval-Augmented Generation)** - core framework
- ✅ **Sentence Transformers + FAISS** - semantic search
- ✅ **Prompt Engineering** - advanced content generation
- ✅ **Vietnamese NLP** - language-specific processing
- ✅ **DataOps Pipeline** - production-grade system design
- ✅ **Microservices Architecture** - modular & scalable

**Kỹ năng nổi bật:**
Data Mining → Machine Learning → NLP → Software Engineering → System Design

**Giá trị:**
Giải pháp hoàn chỉnh từ dữ liệu đến sản phẩm cuối cùng, áp dụng công nghệ AI tân tiến cho tiếng Việt.
