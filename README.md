
<html lang="zh-CN">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>学生文书递交入口</title>
  <style>
    :root {
      color-scheme: light;
      --bg: #f5f6f1;
      --panel: #ffffff;
      --ink: #182322;
      --muted: #687271;
      --line: #d9ded8;
      --green: #20745e;
      --green-soft: #e0f0e8;
      --red: #b34031;
      --red-soft: #f8e1dc;
      --blue: #285f99;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      background: var(--bg);
      color: var(--ink);
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
      letter-spacing: 0;
      display: grid;
      place-items: start center;
      padding: 28px 16px;
    }

    main {
      width: min(880px, 100%);
      display: grid;
      gap: 18px;
    }

    header h1 {
      margin: 0;
      font-size: 34px;
      line-height: 1.1;
    }

    header p {
      margin: 8px 0 0;
      color: var(--muted);
      line-height: 1.7;
    }

    form, .result {
      background: var(--panel);
      border: 1px solid var(--line);
      border-radius: 8px;
      padding: 20px;
      box-shadow: 0 12px 30px rgba(24, 35, 34, .08);
    }

    .form-grid {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 14px;
    }

    label {
      display: grid;
      gap: 6px;
      color: var(--muted);
      font-size: 13px;
      font-weight: 720;
    }

    label.full { grid-column: 1 / -1; }

    input, select, textarea {
      width: 100%;
      border: 1px solid var(--line);
      border-radius: 8px;
      background: #fbfcfa;
      color: var(--ink);
      min-height: 42px;
      padding: 9px 11px;
      font: inherit;
    }

    textarea {
      min-height: 94px;
      resize: vertical;
    }

    .actions {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      align-items: center;
      margin-top: 16px;
    }

    button {
      border: 1px solid transparent;
      border-radius: 8px;
      min-height: 42px;
      padding: 0 16px;
      font: inherit;
      font-weight: 720;
      cursor: pointer;
    }

    .primary {
      background: var(--green);
      color: #fff;
    }

    .ghost {
      background: #eef2ee;
      color: var(--ink);
      border-color: var(--line);
    }

    .note {
      color: var(--muted);
      font-size: 13px;
      line-height: 1.6;
    }

    .result {
      display: none;
      border-color: #b9dac9;
      background: var(--green-soft);
    }

    .result strong { color: var(--green); }

    .error {
      display: none;
      background: var(--red-soft);
      color: var(--red);
      border: 1px solid #ebb2a8;
      border-radius: 8px;
      padding: 12px;
      font-weight: 720;
    }

    @media (max-width: 720px) {
      body { padding: 18px 12px; }
      header h1 { font-size: 28px; }
      .form-grid { grid-template-columns: 1fr; }
      label.full { grid-column: auto; }
    }
  </style>
</head>
<body>
  <main>
    <header>
      <h1>学生文书递交入口</h1>
      <p>填写文书信息并上传 Word 文档，生成提交包后发给Lucy老师（仅限Lucy老师的学生使用该端口）。</p>
    </header>

    <form id="submissionForm">
      <div class="form-grid">
        <label>
          学生姓名
          <input name="studentName" required placeholder="请填写中文姓名">
        </label>
        <label>
          学校名称
          <input name="school" required placeholder="例如：Washington University in St. Louis">
        </label>
        <label>
          国家/地区
          <select name="country">
            <option value="">待确认</option>
            <option>美国</option>
            <option>英国</option>
            <option>加拿大</option>
            <option>香港地区</option>
            <option>德国</option>
            <option>荷兰</option>
            <option>其他</option>
          </select>
        </label>
        <label>
          文书类型
          <select name="essayType" required>
            <option>小文书</option>
            <option>Common App 主文书</option>
            <option>UC PIQ</option>
            <option>UCAS Personal Statement</option>
            <option>Video Essay 脚本</option>
            <option>Interview 素材</option>
            <option>其他</option>
          </select>
        </label>
        <label class="full">
          文书题目
          <textarea name="prompt" required placeholder="粘贴学校题目；如果是主文书，可写 Common App 主文书。"></textarea>
        </label>
        <label>
          版本
          <input name="version" required value="V1" placeholder="例如：V1 / V2 / Final">
        </label>
        <label>
          Word 文档
          <input name="essayFile" type="file" accept=".doc,.docx,application/msword,application/vnd.openxmlformats-officedocument.wordprocessingml.document" required>
        </label>
        <label class="full">
          给老师的备注
          <textarea name="notes" placeholder="例如：这是第二版，主要修改了结尾。"></textarea>
        </label>
      </div>

      <div class="error" id="errorBox"></div>

      <div class="actions">
        <button class="primary" type="submit">生成递交包</button>
        <button class="ghost" type="reset">清空</button>
        <span class="note">支持 .doc 和 .docx。</span>
      </div>
    </form>

    <section class="result" id="resultBox">
      <strong>提交包已生成。</strong>
      <p class="note">请把刚下载的 .json 文件发给升学老师。</p>
    </section>
  </main>

  <script>
    const form = document.querySelector("#submissionForm");
    const errorBox = document.querySelector("#errorBox");
    const resultBox = document.querySelector("#resultBox");

    form.addEventListener("submit", async (event) => {
      event.preventDefault();
      errorBox.style.display = "none";
      resultBox.style.display = "none";

      const values = Object.fromEntries(new FormData(form).entries());
      const file = form.elements.essayFile.files?.[0];
      if (!file || !isWordFile(file)) {
        showError("请上传 .doc 或 .docx 格式的 Word 文档。");
        return;
      }

      try {
        const payload = {
          schema: "essay-submission-v1",
          submittedAt: new Date().toISOString(),
          studentName: values.studentName.trim(),
          school: values.school.trim(),
          country: values.country,
          essayType: values.essayType,
          prompt: values.prompt.trim(),
          version: values.version.trim() || "V1",
          notes: values.notes.trim(),
          file: {
            name: file.name,
            type: file.type || "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
            size: file.size,
            base64: await fileToBase64(file)
          }
        };
        downloadJson(payload, submissionFilename(payload));
        resultBox.style.display = "block";
      } catch (error) {
        showError("提交包生成失败，请换一个浏览器或重新选择文件。");
      }
    });

    function isWordFile(file) {
      const name = file.name.toLowerCase();
      return name.endsWith(".doc") || name.endsWith(".docx");
    }

    function fileToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(String(reader.result).split(",")[1]);
        reader.onerror = reject;
        reader.readAsDataURL(file);
      });
    }

    function downloadJson(payload, filename) {
      const blob = new Blob([JSON.stringify(payload, null, 2)], { type: "application/json;charset=utf-8" });
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.href = url;
      link.download = filename;
      link.click();
      URL.revokeObjectURL(url);
    }

    function submissionFilename(payload) {
      const safe = `${payload.studentName}-${payload.school}-${payload.version}`
        .replace(/[\\/:*?"<>|]+/g, "-")
        .replace(/\s+/g, "-");
      return `${safe || "essay-submission"}.essay-submission.json`;
    }

    function showError(message) {
      errorBox.textContent = message;
      errorBox.style.display = "block";
    }
  </script>
</body>
</html>
