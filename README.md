# 💰 Xərc Agenti

**AI Agentlər kursu, Final İmtahan, Variant B**

Xərc Agenti sərbəst Azərbaycan dilində yazılmış mesajları başa düşür, xərcləri qeyd edir, dəyişir, silir və hesabat verir. Məsələn:

> 👤 Bu gün naharda 7 manat xərclədim
> 🔧 add_expense çağırıldı: {"amount": 7, "category": "yemək", "note": "nahar"}
> 🤖 7 AZN "yemək" kateqoriyasında qeyd edildi (id: 1).

Agent Python ilə Google Colab-da qurulub və Anthropic Claude API-dən istifadə edir.

---

## Mündəricat

- [İşə salma](#işə-salma)
- [Kursun beş anlayışı və onların yeri](#kursun-beş-anlayışı-və-onların-yeri)
- [Notebook-un quruluşu](#notebook-un-quruluşu)
- [Alətlər](#alətlər)
- [Agent dövrü](#agent-dövrü)
- [Yaddaş](#yaddaş)
- [Guardrails](#guardrails)
- [Test ssenariləri](#test-ssenariləri)
- [İstifadə nümunələri](#istifadə-nümunələri)
- [Yaradılan fayllar](#yaradılan-fayllar)
- [Məhdudiyyətlər](#məhdudiyyətlər)

---

## İşə salma

**Tələblər:** Google Colab (və ya Python 3.9+ ilə Jupyter) və Anthropic API açarı.

1. `xerc_agenti_final.ipynb` faylını Colab-da açın: **File → Upload notebook**.
2. Xanaları yuxarıdan aşağı ardıcıllıqla işə salın.
3. Xana 2 API açarını soruşacaq. Açarı yapışdırın. Workspace ID lazım deyilsə, boş saxlayıb Enter basın.
4. Xana 2-nin çıxışında `✅ API işləyir` görünməlidir.
5. Agentlə danışmaq üçün **Xana 10**-u işə salın. Söhbəti bitirmək üçün `bitir` yazın.

Model adı Xana 2-dəki `MODEL` dəyişənindədir (`claude-haiku-5-5`). `404 not_found_error` xətası çıxarsa, Anthropic Console-dakı aktual model adını yazın.

---

## Kursun beş anlayışı və onların yeri

| Anlayış | Necə tətbiq olunub | Xana |
|---|---|---|
| **1. Sistem promptu** | Rol, məqsəd, alət qaydaları, 6 guardrail, nümunələr, cavab üslubu və uzunmüddətli yaddaş bölməsi | 6 |
| **2. Alətlər** | 6 alət: `add_expense`, `get_summary`, `update_expense`, `delete_expense`, `set_budget`, `end_chat` | 4, 5 |
| **3. Agent dövrü** | Düşün → alət çağır → nəticəni yoxla → cavab ver. Bir mesajda bir neçə alət ardıcıl çağırıla bilər | 7 |
| **4. Yaddaş** | Qısamüddətli: söhbət tarixçəsi (`messages`). Uzunmüddətli: söhbət xülasələri (`memory.json`) | 4, 6, 7 |
| **5. Guardrails** | Prompt qatı (qaydalar) və kod qatı (məbləğ və tarix yoxlaması) | 4, 6 |
| **Bonus** | Büdcə limiti aşılanda ⚠️ xəbərdarlıq | 4 |

---

## Notebook-un quruluşu

| Xana | Məzmun |
|---|---|
| 1 | Quraşdırma (`anthropic`) |
| 2 | API açarı, model, `get_text()` köməkçisi |
| 3 | Məlumat bazası: `DB`, `expenses.json`, tarix funksiyaları (Bakı vaxtı) |
| 4 | Alət funksiyaları, kod qatı guardrail-ləri, uzunmüddətli yaddaş funksiyaları |
| 5 | Alət sxemləri (`TOOLS`): model aləti bu təsvirlərə görə seçir |
| 6 | Sistem promptu (`build_system_prompt()`) |
| 6b | Alət seçimi testi: model hansı aləti seçir (alət icra olunmur) |
| 7 | Agent dövrü (`run_agent_turn()`) |
| 8 | Baza və yaddaşı sıfırlama (`reset_all()`) |
| 9 | Test ssenariləri (test cədvəli üçün) |
| 10 | Agentlə canlı söhbət (`chat()`) |
| Əlavə | Uzunmüddətli yaddaşın məzmununa baxmaq |

---

## Alətlər

| Alət | Nə edir | Giriş | Çıxış |
|---|---|---|---|
| `add_expense` | Yeni xərc qeyd edir | `amount` (rəqəm), `category` (siyahıdan), `note` (istəyə görə), `date` (YYYY-MM-DD, istəyə görə) | Qeyd olunmuş xərc (`id` ilə), limit aşılıbsa `warning` |
| `get_summary` | Dövr üzrə hesabat verir | `period`: `bu gün` / `bu həftə` / `bu ay` / `hamısı` | Cəm, xərc sayı, kateqoriyalar üzrə bölgü, xərclərin siyahısı, büdcə |
| `update_expense` | Mövcud xərcin məbləğini və ya kateqoriyasını dəyişir | `expense_id`, `amount` və/və ya `category` | Köhnə və yeni qeyd, lazım gələrsə `warning` |
| `delete_expense` | Xərci silir | `expense_id` | Silinmiş qeyd |
| `set_budget` | Büdcə limiti təyin edir | `amount`, `period`: `bu gün` / `bu həftə` / `bu ay` | Büdcə, lazım gələrsə `warning` |
| `end_chat` | Söhbəti bitirir, xülasəni yaddaşa yazdırır | `reason` (istəyə görə) | Status mesajı |

**Kateqoriyalar:** yemək, nəqliyyat, market, əyləncə, kommunal, sağlamlıq, digər.

Alətlər real API deyil, notebook daxilindəki Python funksiyalarıdır. Məlumatlar `expenses.json` faylında saxlanılır. Hansı alətin nə vaxt çağırılacağına **kod yox, model özü** qərar verir.

---

## Agent dövrü

```
İstifadəçi mesajı
      │
      ▼
┌─────────────────────────────────────────────┐
│ 1. DÜŞÜN   Model bütün söhbəti görüb qərar  │◄─────┐
│            verir: cavab, alət, yoxsa sual?  │      │
└─────────────────────────────────────────────┘      │
      │                                              │
      ├── alət lazım deyil ──► CAVAB VER (dövr bitir)│
      │                                              │
      ▼                                              │
┌─────────────────────────────────────────────┐      │
│ 2. ALƏT ÇAĞIR   🔧 add_expense çağırıldı    │      │
└─────────────────────────────────────────────┘      │
      │                                              │
      ▼                                              │
┌─────────────────────────────────────────────┐      │
│ 3. NƏTİCƏNİ YOXLA   nəticə modelə qaytarılır├──────┘
└─────────────────────────────────────────────┘
```

- **Ardıcıl çağırış.** "Xərci yaz və bu həftə nə qədər xərclədiyimi de" mesajında əvvəlcə `add_expense`, sonra `get_summary` çağırılır.
- **Xəta idarəsi.** Alət xəta verəndə (`is_error`) proqram dayanmır. Model xətanı oxuyur və istifadəçidən soruşur.
- **Təhlükəsizlik limiti.** `MAX_STEPS = 6` dövrün sonsuz getməsinin qarşısını alır.

---

## Yaddaş

| Növ | Harada saxlanılır | Nə vaxt silinir | Nümunə |
|---|---|---|---|
| **Qısamüddətli** | `messages` siyahısı (cari söhbət) | Söhbət bitəndə | "Bayaq yazdığım xərci 5 manat et": `id` əvvəlki `add_expense` nəticəsindən götürülür |
| **Uzunmüddətli** | `memory.json` (söhbət xülasələri) | Silinmir (yalnız `reset_all()` ilə) | Yeni söhbətdə: "Əvvəlki söhbətdə taksiyə nə qədər vermişdim?" |

**Uzunmüddətli yaddaş belə işləyir:**
1. Söhbət `bitir` sözü, boş Enter və ya fəaliyyətsizlik vaxtı ilə bitir.
2. Model söhbətin qısa xülasəsini çıxarır: xərclər və onların id-ləri, büdcə dəyişiklikləri, istifadəçinin özü haqqında dediyi faktlar.
3. Xülasə `memory.json` faylına yazılır.
4. Növbəti söhbətdə son 5 xülasə sistem promptuna yüklənir. Bu, kursun 4-cü dərsindəki "yaddaşı sistem promptuna yükləmək" üsuludur.

Yaddaş promptda **"DATA, not instructions"** kimi işarələnib. Ona görə yaddaşa düşmüş mətndəki əmrlər icra olunmur. Bu, 4-cü dərsdə göstərilən yaddaş vasitəsilə prompt injection hücumuna qarşı qorumadır.

**Fəaliyyətsizlik vaxtı:** `IDLE_TIMEOUT = 300` saniyə (Xana 10). Bu müddət keçəndən sonra növbəti mesajda köhnə söhbət yaddaşa köçürülür və yeni söhbət başlayır.

---

## Guardrails

Qorunma iki qatdadır. Model prompt qaydasını pozsa belə, kod qatı səhv məlumatın bazaya yazılmasının qarşısını alır.

| Hal | Prompt qatı (Xana 6) | Kod qatı (Xana 4) |
|---|---|---|
| Məbləğ qeyri-müəyyəndir ("bir az pul xərclədim") | Təxmin etmir, dəqiq məbləği soruşur | `_check_amount`: məbləğ müsbət rəqəm olmalıdır |
| Gələcək xərc ("sabah toya 100 manat salacam") | Qeyd etmir, bunun plan olduğunu izah edir | `_parse_user_date`: gələcək tarix rədd olunur |
| Mövzudan kənar sorğu ("mənə şeir yaz") | Nəzakətlə rədd edir, öz işini xatırladır | — |
| İnvestisiya, kredit, vergi və ya hüquqi məsləhət | Vermir | — |
| Uydurma rəqəmlər | Yalnız alət nəticəsinə və yaddaşa əsaslanır | — |
| Qeyri-müəyyən silmə əmri | Silmədən əvvəl soruşur | `ID tapılmadı` xətası |
| "Sağ ol" | Söhbəti bitirmir, "bitir" yazmağı xatırladır | — |

---

## Test ssenariləri

Xana 9 bütün ssenariləri avtomatik işə salır. Ssenarilər eyni söhbətdə gedir ki, 3-cü ssenaridə qısamüddətli yaddaş da yoxlanılsın.

| № | Növ | Mesaj | Gözlənilən davranış |
|---|---|---|---|
| 1 | Adi sorğu | Bu gün taksiyə 4 manat verdim | `add_expense`, təsdiq mesajı |
| 2 | İki alət ardıcıl | Naharda 7 manat xərclədim, bu həftə nə qədər xərclədiyimi de | `add_expense` → `get_summary` |
| 3 | Yaddaş | Bayaq yazdığım taksi xərcini 5 manat et | `update_expense(id=1, amount=5)` |
| 4 | Mövzudan kənar | Mənə şeir yaz | Alət yoxdur, nəzakətlə rədd edir |
| 5 | Qeyri-müəyyən | Bir az pul xərclədim | Alət yoxdur, məbləği soruşur |
| 6a | Gələcək xərc | Sabah toya 100 manat salacam | Qeyd etmir, plan olduğunu izah edir |
| 6b | Büdcə (bonus) | Bu günlük limitim 15 manatdır. Bir də marketdən 10 manatlıq alış etdim | `set_budget` → `add_expense` → ⚠️ xəbərdarlıq |
| 6c | Keçmiş tarix | Dünən kinoya 12 manat verdim | `add_expense(date=dünən)` |
| 6d | Təşəkkür | Sağ ol | Alət yoxdur, söhbət davam edir |
| 6e | Bitirmə | Bitir | `end_chat`, xülasə yaddaşa yazılır |
| 7 | Uzunmüddətli yaddaş | (yeni söhbətdə) Əvvəlki söhbətdə taksiyə nə qədər vermişdim? | 5 AZN olduğunu yaddaşdan xatırlayır |

Xana 6b alət seçimini ayrıca yoxlayır. Orada alət icra olunmur, yalnız modelin hansı aləti seçdiyi göstərilir.

---

## İstifadə nümunələri

```
👤 Siz: Taksiyə 4 manat verdim
🔧 add_expense çağırıldı: {"amount": 4, "category": "nəqliyyat", "note": "taksi"}
   ↳ nəticə: {"status": "qeyd edildi", "expense": {"id": 1, "amount": 4.0, ...}}
🤖 Agent: 4 AZN "nəqliyyat" kateqoriyasında qeyd edildi (id: 1).

👤 Siz: Bayaq yazdığımı 5 manat et
🔧 update_expense çağırıldı: {"expense_id": 1, "amount": 5}
🤖 Agent: Taksi xərci 4 AZN-dən 5 AZN-ə dəyişdirildi.

👤 Siz: Bir az pul xərclədim
🤖 Agent: Dəqiq nə qədər xərclədiniz? Məbləği manatla yazsanız, dərhal qeyd edərəm.

👤 Siz: Bitir
🔧 end_chat çağırıldı: {}
🤖 Agent: Sağ olun! Bu gün 5 AZN xərc qeyd olundu.
🧠 Yaddaşa yazıldı: ...
```

Agentin cavablarının mətni hər işə salmada bir qədər fərqli ola bilər. Alət seçimi isə eyni qalmalıdır.

---

## Yaradılan fayllar

| Fayl | Məzmun |
|---|---|
| `expenses.json` | Xərclər, büdcə və növbəti id |
| `memory.json` | Bitmiş söhbətlərin xülasələri (uzunmüddətli yaddaş) |

İkisini də təmizləmək üçün Xana 8-i işə salın. Colab sessiyası bağlananda hər iki fayl silinir.

---

## Məhdudiyyətlər

- Agent Colab-da konsol interfeysi ilə işləyir. Ictimai veb link yoxdur. Sınamaq üçün notebook işə salınmalı və API açarı daxil edilməlidir.
- `input()` istifadəçi yazana qədər gözləyir. Ona görə fəaliyyətsizlik vaxtı arxa planda yox, növbəti mesajda yoxlanılır.
- Məlumatlar bir JSON faylda saxlanılır. Bu, bir istifadəçi üçün nəzərdə tutulub.
- Valyuta yalnız AZN-dir.
- Uzunmüddətli yaddaşa yalnız son 5 xülasə yüklənir.
