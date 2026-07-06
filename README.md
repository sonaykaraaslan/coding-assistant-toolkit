#  BDK Proactive AI Developer Platform

BDK Proactive AI Developer Platform, yazılım geliştirme süreçlerinde geliştiriciye yalnızca komut verildiğinde cevap veren klasik bir AI yardımcıdan öteye geçmeyi amaçlayan, agent tabanlı ve proaktif çalışan bir geliştirici destek sistemidir.

Bu proje; farklı uzmanlık alanlarına sahip AI agent’ları, slash command workflow’ları ve geliştiricinin çalışma ritmini takip eden shadow gözlem katmanı ile daha güvenli, ölçülebilir ve üretim odaklı bir geliştirme deneyimi sunar. Projeye dair teknik detaylar ve mimari akış şemaları için **"Yapay_zeka_entegre_projesi_duzenlendi.docx"** dosyası referans alınabilir.

---

##  Projenin Amacı

Geleneksel AI kod asistanları genellikle reaktif çalışır. Yani geliştirici `/debug`, `/review` veya benzeri bir komut çalıştırmadan sistem devreye girmez. BDK ile amaçlanan proaktif yapı ise şunları sağlar:

- Geliştiricinin kod yazma sürecini arka planda gözlemlemek
- Tekrarlayan build/test hatalarını fark etmek
- Büyük refactor sonrası test çalıştırılmadığında kalite risklerini tespit etmek
- Doğru zamanda doğru workflow’u önermek
- Runtime performans, memory leak ve OOM gibi sorunları kanıta dayalı analiz etmek
- Agent tabanlı uzmanlıklarla yazılım geliştirme sürecini daha sistemli hale getirmek

---

##  Temel Özellikler

### Proaktif Shadow Katmanı
Sistem, geliştiricinin terminal ve dosya değişikliklerinden gelen sinyalleri arka planda takip eder.
- **Build Loop Detection:** Aynı build hatası 3 kez üst üste tekrarlanırsa sistem geliştiricinin aynı hatada takıldığını anlar ve otomatik olarak `/build-fix` workflow’unu önerir.
- **Large Refactor Detection:** 15 dosya değiştiği halde test çalıştırılmadıysa kalite riski oluşturur ve `/review` veya test çalıştırma önerisi verir.
- **Confidence Scoring:** Sistem yanlış alarm üretmemek için deterministik sayaçlar ve ağırlık haritaları kullanır. `shadow_state.json` sayesinde sistem, bir kuralın tetiklenmesi için tam olarak belirlenen eşiğin (`max_streak: 3`, `confidence: 0.95`) aşılmasını bekler ve yalnızca gerçekten yardıma ihtiyaç duyulan anlarda devreye girer.

---

##  Agent Mimarisi

Proje içerisinde farklı sorumluluklara ve niş uzmanlık alanlarına sahip 22 adet uzman agent bulunur:

| Agent | Uzmanlık Alanı | Neden Gerekli? |
|---|---|---|
| `runtime-performance-engineer` | Performans, GC ve Memory Leak | OOMKilled ve container limit uyumsuzluklarına odaklanır. |
| `i18n-specialist` | Çoklu dil, Localization ve RTL | Çoklu dil, yerelleştirme ve sağdan sola (RTL) akış süreçlerini yönetir. |
| `blockchain-developer` | Solidity, Smart Contract ve Web3 | Akıllı kontrat geliştirme ve Web3 cüzdan entegrasyonlarını kapsar. |
| `accessibility-specialist` | WCAG ve Screen Reader | Görme engelliler ve erişilebilirlik standartlarına derinlemesine odaklanır. |
| `data-scientist` | A/B Test ve İstatistiksel Analiz | Kullanıcı hareketlerini analiz eder ve bilimsel A/B test mimarileri tasarlar. |
| `technical-writer-api` | OpenAPI, Swagger ve SDK | Backend kodlarını tarayarak otomatik şemalar ve rehberler üretir. |
| `cost-optimizer` | Cloud Maliyet Optimizasyonu (FinOps) | AWS/GCP/Azure faturalarını inceler ve log seviyelerini optimize eder. |
| `compliance-auditor` | KVKK, GDPR, SOC2 ve HIPAA | Loglardaki hassas verileri ve açık rıza süreçlerini hukuki standartlara göre denetler. |
| `email-notification-engineer` | Transactional Email ve Push | Resend, SendGrid gibi entegrasyonları spama düşmeyecek kuyruk sistemleriyle kurar. |
| `load-testing-engineer` | k6, Locust ve JMeter | Sanal bot ordularıyla sisteme stres testi uygular ve darboğazları raporlar. |

---

##  Yeni Agent: Runtime Performance Engineer

`runtime-performance-engineer`, JVM, Node.js, Python, Go ve .NET gibi farklı runtime ortamlarında performans problemlerini analiz etmek için tasarlanmış özel bir agent’tır.

### Temel Prensipleri
- **Kanıt Olmadan Teşhis Yok:** Asla tahmin yürütmez; her root cause iddiası profiler çıktısı, heap dump, thread dump veya log satırına dayanmalıdır.
- **Dil-Spesifik Bakış:** Proje dosyalarından (`package.json`, `pom.xml`, `go.mod`, `requirements.txt`, `.csproj`) runtime tipini otomatik tespit ederek analizi ona göre özelleştirir.
- **Ölçülebilir Öneriler:** Yalnızca "şunu değiştir" demez; dile uygun bir benchmark iskeleti sunarak doğrulamayı zorunlu kılar.
- **Container Bağlamı:** Kod doğru olsa bile yanlış container konfigürasyonlarını (Örn: JVM `-Xmx` > container RAM limiti uyuşmazlığı) gözden kaçırmaz.

---

##  Workflow’lar ve Slash Komutları

Proje, farklı geliştirme ihtiyaçlarına yönelik 17 adet slash command workflow'u içerir:

| Workflow | Açıklama | Sağladığı Avantaj |
|---|---|---|
| `/diagnose-runtime` | Runtime performans ve memory leak teşhisi yapar | `JMH`, `BenchmarkDotNet`, `pytest-benchmark` gibi araçlarla ölçülebilir düzeltmeler ve benchmark kodu sunar. |
| `/migrate` | Framework veya versiyon geçişlerini planlar | Büyük versiyon geçişlerinde (Vue2→Vue3, Python2→3) kodun tek seferde patlamasını engeller, süreci adıma böler. |
| `/api-mock` | Frontend için mock API ve sunucu üretir | Backend henüz hazır değilken frontend ekibinin durmamasını ve **paralel geliştirme** yapmasını sağlar. |
| `/rollback` | Son deploy’u güvenli şekilde geri alır | Hatalı canlı yayınlardan minimum hasarla ve commit analiziyle geri dönmeyi sağlar. |
| `/dependency-audit` | Zafiyetli veya güncel olmayan paketleri analiz eder | Paket bazlı aktif savunma ve upgrade stratejisi sunar. |
| `/cost-report` | Cloud ve altyapı maliyet raporu çıkarır | Altyapı harcamalarını ve FinOps süreçlerini doğrudan terminale taşır. |
| `/accessibility-scan` | WCAG odaklı erişilebilirlik taraması yapar | Otomatik tarama ve erişilebilirlik düzeltme önerileri sunar. |
| `/db-migration` | Database migration script'leri üretir | Şema değişiklikleri için otomatik migration dosyası ve rollback script'i hazırlar. |
| `/incident-report` | Hata sonrası postmortem raporu oluşturur | Production kesintileri sonrasında olay sonrası kurumsal raporlama sürecini işletir. |
| `/localize` | Metinleri koddan ayıklayıp çeviri dosyalarına aktarır | Çoklu dil paketlerinin üretimini ve yönetimini otomatize eder. |

---

##  `/diagnose-runtime` Workflow Aşamaları

1. **Faz 1 — Runtime Tespiti:** Proje bağımlılık dosyalarından ve `Dockerfile` / `deployment.yaml` manifestlerinden çalışma zamanı ve kaynak limitleri otomatik okunur.
2. **Faz 2 — Statik Tarama:** Unclosed streams (Java), unbounded event listeners (Node.js), circular references (Python) veya goroutine leaks (Go) gibi bilinen leak pattern'leri statik olarak taranır.
3. **Faz 3 — Runtime Veri Talebi:** Kullanıcıdan profiler çıktıları, heap/thread dump'lar ve GC logları istenir. Veri yoksa, dile uygun veri toplama komutları önerilir ve sistem beklemeye geçer.
4. **Faz 4 — Kaynak Konfigürasyonu Kontrolü:** Container limitleri ile runtime heap ayarları arasındaki uyumsuzluklar (Örn: OOMKilled riskleri) denetlenir.
5. **Faz 5 — Rapor Üretimi:** Sürecin sonunda metrik bazlı bir **Health Score**, kanıta dayalı **Root Cause**, ilgili **Evidence** satırları, önceliklendirilmiş **Recommended Fixes** ve doğrulama için dile uygun minimal bir **Benchmark** kodu içeren standart bir rapor sunulur.

---

##  Güncel Mimari Klasör Yapısı

Sistem, Claude Code hooks konfigürasyonu (`settings.json`) üzerinden her araç kullanımı sonrasında sessizce shadow gözlem katmanını tetikler.

```text
.agent/
├── ARCHITECTURE.md              # Sistem haritası
├── CLAUDE.md                    # Claude Code protokol kuralları & Shadow talimatları (YENİ)
├── mcp_config.json.example      # MCP sunucu şablonu
│
├── .claude/
│   ├── settings.json            # Hooks konfigürasyonu (PostToolUse ayarları) (YENİ)
│   └── skills/                  # 20 Slash Command Skill (/build-fix, /review vb.)
│
├── shadow/                      # ─── DEVELOPER COPILOT OBSERVER KATMANI ───
│   ├── shadow-engine.md         # Motorun çalışma mantığı ve dökümantasyonu
│   ├── engine.sh                # Tek Dispatcher (bash/fs sinyallerini yakalayan ana script)
│   │
│   ├── observers/               # Mantıksal İzleyici Kural Grupları
│   │   ├── build-observer.json       # Derleme ve derleyici çıktı izleyicisi
│   │   ├── filesystem-observer.json  # Dosya sistemindeki büyük değişiklik izleyicisi
│   │   └── git-observer.json         # Git konflikt ve branch hareketleri izleyicisi
│   │
│   ├── rules/                   # Tetikleyici Kalıplar ve Ağırlık Skorları
│   │   ├── build-loop.json           # Ardışık 3 derleme hatası yakalama kuralı
│   │   └── large-refactor.json       # Test koşulmadan yapılan 15 dosya değişikliği kuralı
│   │
│   └── state/                    # Ortak Hafıza Alanı (Cross-Event Korelasyonu için)
│       └── shadow_state.json         # Döngü sayaçlarını ve anlık risk skorlarını tutan JSON
│
├── agents/                      # 22 Uzman Agent
├── skills/                      # 56 Core Skill (agent-referenced)
├── skills-extra/                # 37 Extra Skill (niş/opsiyonel)
├── workflows/                   # 17 Workflow Referansı (/build-fix.md, /create.md vb.)
├── rules/                       # Global Kurallar (CLAUDE.md, GEMINI.md)
├── contexts/                    # Mod Bazlı Davranış (dev.md, review.md)
│
├── scripts/                     # Otomasyon Araçları
│   └── hooks/                   # Guardrail script'leri (Güvenlik ağları)
│       ├── dangerous_cmd_check.sh
│       ├── secret_scanner.sh
│       └── lint_check.sh
│
└── .shared/                     # UI/UX kaynakları (CSV data)
