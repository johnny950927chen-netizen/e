<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>從 RAG 到 Agentic Workflow：AI 如何學會「翻書查證」？</title>
  <style>
    :root {
      --bg-primary: #0f172a;
      --bg-card: #1e293b;
      --bg-highlight: #334155;
      --text-main: #f8fafc;
      --text-muted: #94a3b8;
      --accent: #38bdf8;
      --accent-green: #34d399;
      --accent-amber: #fbbf24;
      --accent-rose: #fb7185;
      --border: #334155;
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Noto Sans TC", sans-serif;
      background-color: var(--bg-primary);
      color: var(--text-main);
      height: 100vh;
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    /* 頂部進度條 */
    .progress-container {
      width: 100%;
      height: 6px;
      background: var(--bg-card);
    }
    .progress-bar {
      height: 100%;
      background: linear-gradient(90deg, var(--accent), var(--accent-green));
      width: 12.5%;
      transition: width 0.3s ease;
    }

    /* 頂部狀態列 */
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 14px 32px;
      border-bottom: 1px solid var(--border);
      font-size: 0.9rem;
      color: var(--text-muted);
    }
    .wiki-badge {
      background: rgba(56, 189, 248, 0.15);
      color: var(--accent);
      border: 1px solid rgba(56, 189, 248, 0.4);
      padding: 4px 12px;
      border-radius: 999px;
      font-weight: 600;
      font-size: 0.82rem;
    }

    /* 簡報主舞台 */
    main {
      flex: 1;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 24px 48px;
      overflow-y: auto;
    }

    .slide {
      display: none;
      width: 100%;
      max-width: 1020px;
      animation: fadeIn 0.35s ease;
    }
    .slide.active {
      display: block;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(8px); }
      to { opacity: 1; transform: translateY(0); }
    }

    .slide-tag {
      display: inline-block;
      color: var(--accent-green);
      font-size: 0.85rem;
      font-weight: 700;
      letter-spacing: 0.05em;
      text-transform: uppercase;
      margin-bottom: 8px;
    }

    h1 {
      font-size: 2.5rem;
      line-height: 1.25;
      margin-bottom: 16px;
      background: linear-gradient(90deg, #ffffff, #94a3b8);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }

    h2 {
      font-size: 1.9rem;
      margin-bottom: 14px;
      color: #ffffff;
    }

    .lead {
      font-size: 1.15rem;
      color: var(--text-muted);
      line-height: 1.65;
      margin-bottom: 24px;
    }

    /* 卡片網格 */
    .grid-2 {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
      margin-top: 16px;
    }
    .grid-3 {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 18px;
      margin-top: 16px;
    }

    .card {
      background: var(--bg-card);
      border: 1px solid var(--border);
      border-radius: 14px;
      padding: 22px;
      line-height: 1.6;
    }
    .card h3 {
      font-size: 1.15rem;
      margin-bottom: 10px;
      color: var(--accent);
    }
    .card p, .card li {
      font-size: 0.98rem;
      color: #e2e8f0;
    }
    .card ul {
      padding-left: 20px;
    }
    .card li {
      margin-bottom: 6px;
    }

    /* 生活化類比框 */
    .analogy-box {
      background: rgba(251, 191, 36, 0.1);
      border-left: 4px solid var(--accent-amber);
      padding: 16px 20px;
      border-radius: 0 12px 12px 0;
      margin-bottom: 20px;
    }
    .analogy-box strong {
      color: var(--accent-amber);
      display: block;
      margin-bottom: 4px;
    }

    /* 流程步驟橫排 */
    .flow-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      margin: 22px 0;
    }
    .flow-step {
      flex: 1;
      background: var(--bg-card);
      border: 1px solid var(--accent);
      border-radius: 12px;
      padding: 16px;
      text-align: center;
    }
    .flow-step .step-num {
      font-size: 0.8rem;
      color: var(--accent);
      font-weight: 700;
    }
    .flow-step .step-title {
      font-size: 1.08rem;
      font-weight: 700;
      margin: 6px 0;
    }
    .flow-arrow {
      font-size: 1.5rem;
      color: var(--text-muted);
      font-weight: bold;
    }

    /* 互動對比切換按鈕 */
    .demo-box {
      background: var(--bg-card);
      border: 1px solid var(--border);
      border-radius: 14px;
      padding: 20px;
      margin-top: 14px;
    }
    .tab-btns {
      display: flex;
      gap: 10px;
      margin-bottom: 14px;
    }
    .tab-btn {
      background: var(--bg-highlight);
      color: var(--text-main);
      border: none;
      padding: 8px 16px;
      border-radius: 8px;
      cursor: pointer;
      font-weight: 600;
      font-size: 0.9rem;
    }
    .tab-btn.active {
      background: var(--accent);
      color: #0f172a;
    }
    .demo-output {
      background: #0b1120;
      padding: 16px;
      border-radius: 10px;
      font-size: 0.96rem;
      line-height: 1.6;
      border-left: 4px solid var(--accent-green);
    }

    /* 底部控制列 */
    footer {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 14px 32px;
      border-top: 1px solid var(--border);
      background: rgba(15, 23, 42, 0.95);
    }
    .nav-controls {
      display: flex;
      gap: 12px;
      align-items: center;
    }
    button.nav-btn {
      background: var(--bg-card);
      color: var(--text-main);
      border: 1px solid var(--border);
      padding: 8px 18px;
      border-radius: 8px;
      cursor: pointer;
      font-size: 0.92rem;
      font-weight: 600;
      transition: all 0.2s;
    }
    button.nav-btn:hover:not(:disabled) {
      border-color: var(--accent);
      color: var(--accent);
    }
    button.nav-btn:disabled {
      opacity: 0.4;
      cursor: not-allowed;
    }
    .source-ref {
      font-size: 0.85rem;
      color: var(--text-muted);
    }

    @media (max-width: 768px) {
      .grid-2, .grid-3, .flow-row { grid-template-columns: 1fr; flex-direction: column; }
      h1 { font-size: 1.8rem; }
      main { padding: 16px 20px; }
    }
  </style>
</head>
<body>

  <div class="progress-container">
    <div class="progress-bar" id="progressBar"></div>
  </div>

  <header>
    <div><strong>教學簡報</strong>｜RAG 與 Agentic Workflow 核心原理</div>
    <div class="wiki-badge" id="wikiSourceTop">知識庫來源：INDEX.md</div>
  </header>

  <main>
    <!-- Slide 1: 封面 -->
    <section class="slide active" data-wiki="INDEX.md（知識庫總覽）">
      <div class="slide-tag">入門概念導覽</div>
      <h1>為什麼 AI 需要「翻書」與「自我檢查」？</h1>
      <p class="lead">
        帶你從零理解現代 AI 系統的三個核心進化：<br>
        從<strong>單純聊天（LLM）</strong> → <strong>開卷查資料（RAG）</strong> → <strong>會自我修正的工作流（Agentic Graph）</strong>。
      </p>
      <div class="grid-3">
        <div class="card">
          <h3>1. 解決「瞎掰」問題</h3>
          <p>認識大語言模型為什麼會產生幻覺，以及如何用外部知識庫補齊最新資訊。</p>
        </div>
        <div class="card">
          <h3>2. 理解「語意搜尋」</h3>
          <p>看懂電腦如何把文章切成卡片、轉成數字向量，精準找出相關段落。</p>
        </div>
        <div class="card">
          <h3>3. 掌握「迴圈與圖譜」</h3>
          <p>了解 AI 如何像編輯團隊一樣，經過草稿、審查、退回修改才輸出最終答案。</p>
        </div>
      </div>
    </section>

    <!-- Slide 2: LLM 的限制 -->
    <section class="slide" data-wiki="01-llm-limitations.md">
      <div class="slide-tag">第一步：認識問題根源</div>
      <h2>大語言模型（LLM）的兩大先天限制</h2>
      <div class="analogy-box">
        <strong>💡 生活化類比：記憶力超強但「不能翻書」的考生</strong>
        考生只能憑考前背進腦袋的知識作答。遇到考前沒背過的公司內部規定，或考試當天才發生的新聞，他為了不交白卷，就會用極具說服力的語氣編造答案。
      </div>
      <div class="grid-2">
        <div class="card">
          <h3>⏳ 知識截止日（Knowledge Cutoff）</h3>
          <p>模型一旦訓練完成，權重就固定了。它不知道訓練時間之後的新事件，也完全沒有存取過你的個人筆記或公司內部文件。</p>
        </div>
        <div class="card">
          <h3>🌀 幻覺（Hallucination）</h3>
          <p>LLM 的本質是「預測下一個最順暢的字詞」，而不是「查詢資料庫」。當缺乏事實依據時，它會優先維持語氣通順而捏造細節。</p>
        </div>
      </div>
    </section>

    <!-- Slide 3: RAG 核心架構 -->
    <section class="slide" data-wiki="02-rag-architecture.md">
      <div class="slide-tag">第二步：把閉卷考變成開卷考</div>
      <h2>RAG（檢索增強生成）是如何運作的？</h2>
      <div class="analogy-box">
        <strong>💡 生活化類比：開卷考試（Open-Book Exam）</strong>
        不強迫 AI 死背所有冷門知識，而是規定它在回答前，必須先去指定的資料庫翻找相關段落，再根據找到的內容整理出答案。
      </div>
      <div class="flow-row">
        <div class="flow-step">
          <div class="step-num">STAGE 1（離線準備）</div>
          <div class="step-title">Indexing 建立索引</div>
          <p style="font-size:0.88rem; color:var(--text-muted);">將文件切段並轉為向量存入資料庫</p>
        </div>
        <div class="flow-arrow">→</div>
        <div class="flow-step">
          <div class="step-num">STAGE 2（線上查詢）</div>
          <div class="step-title">Retrieval 檢索</div>
          <p style="font-size:0.88rem; color:var(--text-muted);">根據問題找出語意最接近的前 K 筆資料</p>
        </div>
        <div class="flow-arrow">→</div>
        <div class="flow-step">
          <div class="step-num">STAGE 3（結合輸出）</div>
          <div class="step-title">Generation 生成</div>
          <p style="font-size:0.88rem; color:var(--text-muted);">限制 AI 僅依據檢索到的資料作答</p>
        </div>
      </div>
    </section>

    <!-- Slide 4: Chunking 與 Embedding -->
    <section class="slide" data-wiki="03-chunking-and-embedding.md">
      <div class="slide-tag">第三步：資料怎麼存才找得到？</div>
      <h2>文件切分（Chunking）與語意向量（Embedding）</h2>
      <div class="grid-2">
        <div class="card">
          <h3>🗂️ Chunking：把書拆成「重點索引卡」</h3>
          <p style="margin-bottom:10px;">整本書直接塞給 AI 會超出記憶容量，且雜訊太多會讓模型分心。因此必須把長文切成小段落（Chunks）。</p>
          <ul>
            <li><strong>切太小</strong>：失去前後文脈絡，看不懂代名詞。</li>
            <li><strong>切太大</strong>：混入太多無關雜訊。</li>
            <li><strong>Overlap（重疊）</strong>：相鄰卡片保留 10%～20% 重疊，避免關鍵句子在邊界被腰斬。</li>
          </ul>
        </div>
        <div class="card">
          <h3>🧭 Embedding：概念座標圖書館</h3>
          <p style="margin-bottom:10px;">神經網路會把文字轉換成一串高維度數字（向量），代表它的「語意座標」。</p>
          <ul>
            <li><strong>傳統關鍵字搜尋</strong>：搜「筆電當機」找不到「電腦畫面凍結」。</li>
            <li><strong>語意向量搜尋</strong>：意思相近的句子在座標空間距離很近，即使字面不同也能精準命中！</li>
          </ul>
        </div>
      </div>
    </section>

    <!-- Slide 5: 互動體驗 RAG 差異 -->
    <section class="slide" data-wiki="01-llm-limitations.md & 02-rag-architecture.md">
      <div class="slide-tag">概念對照演示</div>
      <h2>點擊切換：沒有 RAG vs. 啟用 RAG 的回答差異</h2>
      <p class="lead">假設使用者詢問：「我們公司 2026 年遠端工作補助上限是多少？」</p>
      <div class="demo-box">
        <div class="tab-btns">
          <button class="tab-btn active" onclick="showDemo('no-rag', this)">❌ 單純 LLM（無外部資料）</button>
          <button class="tab-btn" onclick="showDemo('with-rag', this)">✅ 啟用 RAG（檢索內部 Wiki）</button>
        </div>
        <div class="demo-output" id="demoContent">
          <strong>系統狀態：</strong>未連接外部文件，模型僅憑訓練記憶猜測。<br><br>
          <strong>AI 回答：</strong>「一般科技公司通常提供每月 1,000 至 3,000 元不等的遠端設備津貼，建議您向人資部門確認具體金額。」<br>
          <span style="color:var(--accent-rose);">⚠️ 問題：看似合理，但完全不知道該公司的真實規定（若硬猜具體數字則構成幻覺）。</span>
        </div>
      </div>
    </section>

    <!-- Slide 6: Agentic Loop -->
    <section class="slide" data-wiki="04-agentic-loop.md">
      <div class="slide-tag">第四步：從單次回答到自我修正</div>
      <h2>Agentic Loop：寫完草稿先檢查再交卷</h2>
      <div class="analogy-box">
        <strong>💡 生活化類比：考卷交卷前的自我驗算</strong>
        單次 RAG 就像「翻書寫完就直接交卷」；Agentic Loop 則像「寫完草稿後，拿著檢核表逐項比對原始資料，發現漏寫或寫錯就擦掉重寫，確認沒問題才交卷」。
      </div>
      <div class="grid-3">
        <div class="card">
          <h3>1. Generate（生成）</h3>
          <p>根據目前檢索到的 Wiki 條目，產出初版回答或教學草稿。</p>
        </div>
        <div class="card">
          <h3>2. Critique（審查）</h3>
          <p>對照檢核清單檢查：有無捏造事實？是否漏掉關鍵類比？初學者看得懂嗎？</p>
        </div>
        <div class="card">
          <h3>3. Revise（修正與停止）</h3>
          <p>若未通過，把具體錯誤原因退回重寫；並設定最大重試次數（如最多 3 次）防止無限迴圈。</p>
        </div>
      </div>
    </section>

    <!-- Slide 7: Graph Workflow -->
    <section class="slide" data-wiki="05-graph-workflow.md">
      <div class="slide-tag">第五步：工程化控管流程</div>
      <h2>Graph 工作流：狀態機與條件路由</h2>
      <p class="lead">當流程變複雜時，我們用「有向圖（Graph）」把每個步驟變成可控的標準作業線：</p>
      <div class="grid-3">
        <div class="card">
          <h3>📦 State（共享狀態）</h3>
          <p>貫穿整條產線的資料夾，統一存放「已讀取的 Wiki」、「目前草稿」、「審查意見」與「重試次數」。</p>
        </div>
        <div class="card">
          <h3>⚙️ Nodes（節點）</h3>
          <p>每個節點只做一件明確的事（例如：專門抓資料的節點、專門寫作的節點、專門抓錯的節點）。</p>
        </div>
        <div class="card">
          <h3>🔀 Conditional Edges（條件邊）</h3>
          <p>決定流程走向的紅綠燈：<strong>審查通過 → 進入 HTML 排版</strong>；<strong>審查不通過 → 沿退回箭頭重寫</strong>。</p>
        </div>
      </div>
    </section>

    <!-- Slide 8: 總結 -->
    <section class="slide" data-wiki="INDEX.md（全知識庫總結）">
      <div class="slide-tag">學習總結</div>
      <h2>一張圖帶走今天的核心觀念</h2>
      <div class="card" style="margin-bottom:18px;">
        <ul>
          <li style="margin-bottom:12px;"><strong>LLM 的局限（WIKI-01）</strong>：模型只有靜態參數記憶，缺乏最新與私有知識，容易產生幻覺。</li>
          <li style="margin-bottom:12px;"><strong>RAG 開卷架構（WIKI-02 & 03）</strong>：透過 Chunking 切卡片與 Embedding 語意搜尋，讓 AI 先查資料再回答。</li>
          <li style="margin-bottom:12px;"><strong>Agentic Loop（WIKI-04）</strong>：加入「生成 → 檢查 → 修正」的反思迴圈，大幅降低單次輸出的錯誤率。</li>
          <li><strong>Graph 工作流（WIKI-05）</strong>：用 State、Nodes 與條件分支把 AI 流程工程化，確保品質穩定可控。</li>
        </ul>
      </div>
      <p style="color:var(--accent-green); font-weight:600; text-align:center;">
        🎉 恭喜完成導覽！可使用下方按鈕或鍵盤 ← / → 方向鍵隨時回顧各章節。
      </p>
    </section>
  </main>

  <footer>
    <div class="source-ref" id="wikiSourceBottom">對應條目：INDEX.md（知識庫總覽）</div>
    <div class="nav-controls">
      <button class="nav-btn" id="prevBtn" onclick="changeSlide(-1)" disabled>← 上一頁</button>
      <span id="pageIndicator" style="font-size:0.9rem; color:var(--text-muted); min-width:60px; text-align:center;">1 / 8</span>
      <button class="nav-btn" id="nextBtn" onclick="changeSlide(1)">下一頁 →</button>
    </div>
  </footer>

  <script>
    const slides = document.querySelectorAll('.slide');
    const progressBar = document.getElementById('progressBar');
    const pageIndicator = document.getElementById('pageIndicator');
    const prevBtn = document.getElementById('prevBtn');
    const nextBtn = document.getElementById('nextBtn');
    const wikiSourceTop = document.getElementById('wikiSourceTop');
    const wikiSourceBottom = document.getElementById('wikiSourceBottom');

    let currentSlide = 0;

    function updateSlide() {
      slides.forEach((slide, index) => {
        slide.classList.toggle('active', index === currentSlide);
      });

      const total = slides.length;
      const progress = ((currentSlide + 1) / total) * 100;
      progressBar.style.width = `${progress}%`;
      pageIndicator.textContent = `${currentSlide + 1} / ${total}`;

      prevBtn.disabled = currentSlide === 0;
      nextBtn.disabled = currentSlide === total - 1;

      const wikiRef = slides[currentSlide].getAttribute('data-wiki');
      wikiSourceTop.textContent = `知識庫來源：${wikiRef}`;
      wikiSourceBottom.textContent = `對應條目：${wikiRef}`;
    }

    function changeSlide(direction) {
      const nextIndex = currentSlide + direction;
      if (nextIndex >= 0 && nextIndex < slides.length) {
        currentSlide = nextIndex;
        updateSlide();
      }
    }

    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowRight' || e.key === ' ') {
        changeSlide(1);
      } else if (e.key === 'ArrowLeft') {
        changeSlide(-1);
      }
    });

    function showDemo(type, btn) {
      document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      const output = document.getElementById('demoContent');

      if (type === 'no-rag') {
        output.style.borderLeftColor = 'var(--accent-rose)';
        output.innerHTML = `
          <strong>系統狀態：</strong>未連接外部文件，模型僅憑訓練記憶猜測。<br><br>
          <strong>AI 回答：</strong>「一般科技公司通常提供每月 1,000 至 3,000 元不等的遠端設備津貼，建議您向人資部門確認具體金額。」<br>
          <span style="color:var(--accent-rose);">⚠️ 問題：看似合理，但完全不知道該公司的真實規定（若硬猜具體數字則構成幻覺）。</span>
        `;
      } else {
        output.style.borderLeftColor = 'var(--accent-green)';
        output.innerHTML = `
          <strong>系統狀態：</strong>已從內部知識庫檢索到 <code>HR-Policy-2026.md (Chunk #4)</code>。<br><br>
          <strong>AI 回答：</strong>「根據《2026 遠端工作管理辦法》第 4 條，正職員工每月可申請最高 <strong>新台幣 2,500 元</strong> 的網路與設備補助，需於次月 5 日前檢附發票申報。」<br>
          <span style="color:var(--accent-green);">✅ 優勢：有明確出處、數字精準，且完全杜絕憑空捏造。</span>
        `;
      }
    }

    updateSlide();
  </script>
</body>
</html>
