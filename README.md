# OpenCloud Models

A practical model guide for [OpenClaude](https://github.com/gutsofpratik/OpenClaude): what to download, which runtime to choose, and how to size your computer. This repository contains **documentation and a machine-readable catalog, not model weights**. OpenCloud is the requested repository name; the application is named OpenClaude.

Source links and file metadata checked **17 October 2026**. This is a curated starting list, not an exhaustive compatibility promise. “Best” depends on memory, workload, runtime, context and latency. New releases may change the choices below.

## Choose a starting model

| Your priority | Starting choice | Why / main tradeoff |
|---|---|---|
| Balanced everyday assistant | Qwen3.5 9B Q4_K_M | Chat, coding and multimodal potential at moderate memory cost |
| Smaller computer | Qwen3.5 4B Q4_K_M | Smaller download; complex tool chains need closer checking |
| Compact larger reasoning model | Bonsai 2 27B PTQ1_0 | Specialized low-bit runtime required; a smaller file does not guarantee faster responses |
| Coding on a larger GPU | Qwen3 Coder 30B A3B Instruct Q4_K_M | Coding specialization; approximately 18.56 GB of weights and no thinking mode |
| Text reasoning with Harmony support | gpt-oss 20B | Use a provider/runtime that implements its response format correctly |
| No suitable local hardware | Cloud model connection | OpenRouter or another compatible API; internet, provider account and potential usage charges |

These are workload recommendations, not a leaderboard or measured OpenClaude speed ranking. Bonsai and Qwen3.5 9B have user-reported use in OpenClaude; this repository does not yet contain reproducible end-to-end benchmarks. Other entries are candidates based on upstream documentation, not certified integrations.

## System planning

**Every RAM/VRAM range below is an estimate for an initial, modest-context setup, not a measured minimum or a guarantee.** A model file is only the weights. Runtime buffers, KV/recurrent state, image projector, graph allocations, operating system and other applications also consume memory. CPU offload needs system RAM and can reduce speed. GPU utilization describes activity, not memory usage.

Start at **8,192 total context tokens**, check real memory use, then increase to 16K/32K if needed. Advertised maximum context is an architectural ceiling, not a promise that it fits your GPU. Input, conversation, tools, images and output share the total context window. Do not set the full advertised maximum by default.

| Model / selected format | Download size (decimal GB) | System RAM planning | GPU VRAM planning |
|---|---:|---|---|
| Qwen3.5 4B Q4_K_M | 2.74 | 8–16 GB | 4–6 GB for short text contexts |
| Qwen3.5 9B Q4_K_M | 5.68 | 16–32 GB | 8–12 GB for modest context |
| Bonsai 2 27B PTQ1_0 | 5.95 | 16 GB initial target; 32 preferred | 8–12 GB initial target; Prism required |
| Qwen3 Coder 30B A3B Q4_K_M | 18.56 | 32–64 GB | 24+ GB initial target; 16 GB needs offload |
| gpt-oss 20B native MXFP4 | See upstream files | 32 GB preferred | Upstream describes 16 GB memory deployment; backend dependent |

For full GPU offload, use a compatible GPU runtime and enough free VRAM. For CPU-only use, budget RAM for weights plus context and expect substantially different speed. Apple Silicon uses shared memory; the Windows desktop build does not establish a released native macOS application. A model's MLX version is not a CUDA file.

## Model details and downloads

### Bonsai 2 27B

“Bonsai 227B” refers here to **Bonsai 2 27B**, not a 227-billion-parameter model. The publisher supplies PTQ1_0 (5.95 GB), PQ2_0 (7.21 GB), and an optional Q8 vision projector (0.63 GB). Use the **Prism llama.cpp fork**, not stock llama.cpp, for these ternary files. Follow its current compatibility guidance rather than renaming files to another quantization type.

Thinking and a 262K architectural context are documented upstream. Image support also needs the matching projector and a compatible OpenClaude vision path. These features are not established just by loading text weights. Choose PTQ1_0 when memory is tight; compare PQ2_0 on your hardware. OpenClaude's earlier 60K configuration was one machine's choice, not a recommended default for everyone.

Sources: [publisher model card](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf), [Prism runtime/setup reference](https://github.com/PrismML-Eng/Bonsai-demo), [runtime binaries](https://github.com/PrismML-Eng/llama.cpp/releases).

### Qwen3.5 9B

Our balanced starting recommendation. The publisher documents a multimodal model with thinking and 262,144 native context tokens. Use the original [Qwen Safetensors checkpoint](https://huggingface.co/Qwen/Qwen3.5-9B) with a compatible Transformers engine, or the [Unsloth GGUF conversion](https://huggingface.co/unsloth/Qwen3.5-9B-GGUF). Unsloth is a third-party quantizer, not the original model publisher. Q4_K_M is a practical starting quantization, not identical to full precision.

For GGUF images, also obtain the matching mmproj from that same conversion repository and configure it in the app. Keep the tokenizer/chat template matched. Check thinking controls through the runtime; do not assume typing /think enables reasoning.

### Qwen3.5 4B

A smaller starting choice when memory or latency matters more than peak task quality. The [publisher checkpoint](https://huggingface.co/Qwen/Qwen3.5-4B) provides text/image processing and thinking; the [Unsloth GGUF conversion](https://huggingface.co/unsloth/Qwen3.5-4B-GGUF) offers smaller local files. The matching image projector adds memory. Its long architectural context does not make a long context practical on a small GPU. Test tool selection, JSON arguments, browser actions and document workflows with your actual tasks before relying on unattended runs.

### Qwen3 Coder 30B A3B Instruct

The [publisher model card](https://huggingface.co/Qwen/Qwen3-Coder-30B-A3B-Instruct) documents 30.5B total parameters with 3.3B active, 262,144 native context, coding/tool-use focus and **non-thinking-only** behavior. It is text-only. Active parameters affect compute; they do not mean only 3.3B parameters need storage. The [Unsloth GGUF conversion](https://huggingface.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF) is a third-party option. Use a compatible recent runtime and its actual tool-call template. This catalog has not validated the complete OpenClaude tool loop for this model.

### gpt-oss 20B

The [official model card and download repository](https://huggingface.co/openai/gpt-oss-20b) describe a text-only reasoning/tool model with native MXFP4 and a 16 GB memory deployment target. **Harmony formatting is required.** A backend that merely opens a Safetensors file is not sufficient. Prefer a current compatible Ollama/LM Studio/provider path until you have validated your chosen built-in engine's formatting and quantization support. Thinking effort is provider dependent; image input is not supported by this model. No OpenClaude hardware benchmark is claimed here.

## Revision-pinned file downloads

Links below point to the exact repository revision checked for this catalog. File bytes and SHA-256 are supplied by Hugging Face's metadata; these hashes are integrity checks, not a security audit or endorsement. Original publisher links and third-party conversions are distinguished above. Model files remain hosted upstream.

**Bonsai 2 27B**

| File | Size | SHA-256 |
|---|---:|---|
| [Ternary-Bonsai-2-27B-PQ2_0.gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf/resolve/b072e1d3b35a0a630cece372c2127528e0994386/Ternary-Bonsai-2-27B-PQ2_0.gguf?download=true) | 7.21 GB | `3907dc1658db1f78a9826bf8d5bcb8dc65db0d466388937af57f2294fae62ec1` |
| [Ternary-Bonsai-2-27B-PTQ1_0.gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf/resolve/b072e1d3b35a0a630cece372c2127528e0994386/Ternary-Bonsai-2-27B-PTQ1_0.gguf?download=true) | 5.95 GB | `53107f530aa52eb00912263ab1ee29bd199261c87cd7b4ad4ca1318c1fe33ee3` |
| [Ternary-Bonsai-2-27B-mmproj-Q8_0.gguf](https://huggingface.co/prism-ml/Ternary-Bonsai-2-27B-gguf/resolve/b072e1d3b35a0a630cece372c2127528e0994386/Ternary-Bonsai-2-27B-mmproj-Q8_0.gguf?download=true) | 0.63 GB | `6807ede61d570bb86ba34b756a0fa109edc33668604de867c6ea6d8f1d631903` |

**Qwen3.5 9B**

| File | Size | SHA-256 |
|---|---:|---|
| [Qwen3.5-9B-Q4_K_M.gguf](https://huggingface.co/unsloth/Qwen3.5-9B-GGUF/resolve/3885219b6810b007914f3a7950a8d1b469d598a5/Qwen3.5-9B-Q4_K_M.gguf?download=true) | 5.68 GB | `03b74727a860a56338e042c4420bb3f04b2fec5734175f4cb9fa853daf52b7e8` |

**Qwen3.5 4B**

| File | Size | SHA-256 |
|---|---:|---|
| [Qwen3.5-4B-Q4_K_M.gguf](https://huggingface.co/unsloth/Qwen3.5-4B-GGUF/resolve/e87f176479d0855a907a41277aca2f8ee7a09523/Qwen3.5-4B-Q4_K_M.gguf?download=true) | 2.74 GB | `00fe7986ff5f6b463e62455821146049db6f9313603938a70800d1fb69ef11a4` |

**Qwen3 Coder 30B A3B Instruct**

| File | Size | SHA-256 |
|---|---:|---|
| [Qwen3-Coder-30B-A3B-Instruct-Q4_K_M.gguf](https://huggingface.co/unsloth/Qwen3-Coder-30B-A3B-Instruct-GGUF/resolve/b17cb02dd882d5b6ab62fc777ad2995f19668350/Qwen3-Coder-30B-A3B-Instruct-Q4_K_M.gguf?download=true) | 18.56 GB | `fadc3e5f8d42bf7e894a785b05082e47daee4df26680389817e2093056f088ad` |

For Qwen vision projectors, use the **Files** tab of the same GGUF repository; choose the matching mmproj and budget additional memory. For Safetensors, download the **whole checkpoint directory**, including every shard, tokenizer, configuration and required processor files. One Safetensors shard is not a complete model. Read upstream requirements before enabling remote Python code.

## Use with OpenClaude

1. **Desktop built-in loader:** Settings → Models → Choose model file (GGUF) or choose the complete checkpoint folder. Select a compatible runtime, context and GPU placement. For Bonsai ternary, configure Prism first. Choose the model in chat; sending the first message triggers loading. Hardware settings remain desktop features.
2. **Ollama:** Download a supported model through Ollama, add its connection under Settings → Models, test it, then choose the model in chat. Ollama controls its model configuration. Desktop can also import downloaded Ollama weights where supported.
3. **LM Studio:** Load a compatible model there, enable its local OpenAI-compatible server, add the LM Studio preset in OpenClaude, test and select it. Provider controls context and placement.
4. **Cloud:** Add the OpenRouter preset or a custom OpenAI-compatible base URL and API key. Use provider model IDs. Manual IDs support endpoints without a /models listing. Selecting an API model does not download weights or consume local GPU inference memory.

The model's native capability, engine support and OpenClaude integration must all agree. A tool-capable checkpoint does not automatically give it unrestricted terminal/computer access. Tools must be available, enabled, and correctly called. Vision needs actual image input and compatible inference, not only a model label. A non-thinking model will not gain reasoning controls from a checkbox.

## Cloud models and external search

OpenClaude's source supports **OpenAI-compatible Chat Completions with SSE streaming**, including OpenRouter at https://openrouter.ai/api/v1. In Settings → Models, add a connection, enter a key, test model discovery and choose a model in chat. Put exact model IDs in the optional list for APIs without /models. Such a test confirms configuration only; an actual generation is needed to verify credentials. “Omni Router” is not treated as a verified service name; if you mean a different router, use its documented compatible endpoint. Native Anthropic Messages, Responses-only and proprietary APIs require a separate adapter; arbitrary APIs are not automatically compatible. See [OpenRouter documentation](https://openrouter.ai/docs/quickstart).

In Settings → Capabilities → Web search, choose **Brave Search API**, **Tavily**, SearXNG, DuckDuckGo, or a custom API. A model API key and a search API key are separate credentials. Search-provider settings are used by the agent's existing web-search tools; no browser extension is required.

A custom search adapter must accept GET parameters q, limit, page, time_range, language and safesearch, and return:

```json
{ "results": [{ "title": "Example", "url": "https://example.com", "content": "A useful snippet" }] }
```

Optional authentication is an Authorization: Bearer header. Use HTTPS; HTTP is allowed only for loopback development endpoints. Services with different contracts need a translation adapter. Brave supports limited paging; Tavily returns up to 20 results per query and needs new queries for broader coverage. Provider quotas and readable-page failures still apply. See [Brave API docs](https://api-dashboard.search.brave.com/app/documentation/web-search) and [Tavily API docs](https://docs.tavily.com/documentation/api-reference/endpoint/search).

Keys are stored in the application's **local backend database**, not published in this catalog or returned by entity APIs. This is not a claim of encrypted credential storage. Protect the app data directory. Cloud prompts and search queries go to the configured third party; review its privacy, retention and billing terms. Do not submit keys in GitHub issues.

These connection additions are in current OpenClaude source; the earlier v0.1.1 installer predates them.

## Performance and troubleshooting

- Slow first reply: loading weights and processing a new prompt are separate from token generation. Measure time to first token and decode tokens/second separately.
- Slow follow-ups: verify prefix-cache reuse, prompt changes, tool-result volume, context and engine behavior. New input/tool results still require evaluation even with a warm cache.
- Out of memory: lower context, close competing model processes, use a smaller model/quantization, or adjust offload. Do not silently inflate weights to FP16 for a compact ternary file.
- Missing thinking: confirm the model actually supports it, the provider emits it, and the runtime template parses it. Some cloud services provide summaries rather than raw reasoning.
- Tools fail: check the model's native tool template and the specific available tool/schema. A successful text reply does not validate tools.
- Vision fails: verify matching projector/processor files and a supported multimodal backend.

## Catalog maintenance and licensing

[models.json](models.json) records source revisions, exact file hashes, estimated hardware profiles and validation labels. Add a model with authoritative sources, exact format/runtime requirements, honest verification status and a reproducible test record. We do not list a model as fully tested based solely on a model-card claim. File sizes are decimal GB; operating systems may display GiB.

Original documentation/catalog contributions use the [MIT license](LICENSE). Model weights, conversions, tokenizers and runtimes retain their upstream licenses; this license does not relicense or redistribute them. This is an independent project, unaffiliated with Anthropic, OpenAI, Qwen, PrismML, Unsloth or the named API providers.
