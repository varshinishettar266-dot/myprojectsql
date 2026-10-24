# const GEMINI_MODEL = "gemini-2.5-flash";

export default async (request) => {
  if (request.method !== "POST") {
    return json({ error: "Method not allowed. Use POST." }, 405);
  }

  const apiKey = process.env.GEMINI_API_KEY;
  if (!apiKey) {
    return json({
      error: "The Gemini API key is not configured. Add GEMINI_API_KEY in Netlify environment variables."
    }, 500);
  }

  let body;
  try {
    body = await request.json();
  } catch {
    return json({ error: "Please send valid JSON." }, 400);
  }

  const question = String(body.question || "").trim().slice(0, 3000);
  const schema = String(body.schema || "").trim().slice(0, 5000);
  const dialect = String(body.dialect || "Standard SQL").trim().slice(0, 80);

  if (!question) return json({ error: "Please enter a question." }, 400);

  const prompt = `
You are a careful SQL tutor. Convert the user's plain-English request into a SQL query.
SQL dialect: ${dialect}
Table structure supplied by user (may be empty):
${schema || "No schema supplied. Use reasonable illustrative table/column names and mention assumptions."}

User request:
${question}

Return ONLY valid JSON with exactly these string keys:
{
  "sql": "the SQL query, without markdown fences",
  "explanation": "A concise plain-English explanation. Mention assumptions if no schema was supplied."
}
Do not execute the query. Do not claim it has been tested. If the request is ambiguous, make a sensible assumption and state it in the explanation. Do not include dangerous instructions or credentials.
`;

  try {
    const apiResponse = await fetch(
      `https://generativelanguage.googleapis.com/v1beta/models/${GEMINI_MODEL}:generateContent?key=${encodeURIComponent(apiKey)}`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          contents: [{ parts: [{ text: prompt }] }],
          generationConfig: {
            temperature: 0.2,
            responseMimeType: "application/json"
          }
        })
      }
    );

    const result = await apiResponse.json();
    if (!apiResponse.ok) {
      const message = result?.error?.message || "Gemini API request failed.";
      return json({ error: message }, apiResponse.status === 429 ? 429 : 502);
    }

    const generatedText = result?.candidates?.[0]?.content?.parts
      ?.map((part) => part.text || "")
      .join("")
      .trim();

    if (!generatedText) {
      return json({ error: "The AI returned an empty response. Please try again." }, 502);
    }

    let parsed;
    try {
      parsed = JSON.parse(generatedText);
    } catch {
      const cleaned = generatedText.replace(/^```(?:json)?\s*/i, "").replace(/\s*```$/, "");
      try {
        parsed = JSON.parse(cleaned);
      } catch {
        return json({ error: "The AI response could not be read. Please try again." }, 502);
      }
    }

    if (typeof parsed.sql !== "string" || typeof parsed.explanation !== "string") {
      return json({ error: "The AI response did not include SQL and an explanation." }, 502);
    }

    return json({
      sql: parsed.sql.trim(),
      explanation: parsed.explanation.trim()
    });
  } catch {
    return json({ error: "Unable to reach the AI service. Check your connection and try again." }, 502);
  }
};

function json(data, status = 200) {
  return new Response(JSON.stringify(data), {
    status,
    headers: {
      "Content-Type": "application/json; charset=utf-8",
      "Cache-Control": "no-store"
    }
  });
}
myprojectsql
if I can give a text in english laungaue it converts into sql
@import url('https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=Manrope:wght@400;500;600;700;800&display=swap');

:root {
  --bg: #080d18;
  --panel: #101827;
  --panel-2: #0b1220;
  --line: #243247;
  --text: #f3f6ff;
  --muted: #9ba9bd;
  --accent: #a6f36a;
  --accent-dark: #213b1b;
  --purple: #a7a2ff;
}
* { box-sizing: border-box; }
html { scroll-behavior: smooth; }
body {
  margin: 0; background: radial-gradient(ellipse at 50% 0%, #17233c 0, var(--bg) 50%);
  color: var(--text); font-family: Manrope, system-ui, sans-serif; min-height: 100vh;
}
button, textarea, select { font: inherit; }
.topbar {
  max-width: 1180px; margin: auto; padding: 24px 26px; display: flex;
  align-items: center; justify-content: space-between; border-bottom: 1px solid #ffffff0d;
}
.brand { display: flex; gap: 11px; align-items: center; color: var(--text); text-decoration: none; font-weight: 800; letter-spacing: -.4px; }
.brand em { color: var(--muted); font-style: normal; font-weight: 500; }
.brand-icon { width: 34px; height: 34px; border-radius: 10px; background: var(--accent); color: #10200a; display: grid; place-items: center; font-weight: 900; }
.top-link { color: #c4cee0; text-decoration: none; font-size: 13px; }
.top-link:hover { color: var(--accent); }
.page { max-width: 1180px; padding: 0 26px; margin: auto; }
.hero { text-align: center; padding: 74px 0 42px; }
.eyebrow, .section-label { color: var(--accent); font: 500 11px 'DM Mono', monospace; letter-spacing: 2px; }
.pulse { display: inline-block; width: 7px; height: 7px; border-radius: 50%; background: var(--accent); margin-right: 7px; box-shadow: 0 0 12px #a6f36a88; }
h1 { font-size: clamp(42px, 6vw, 70px); letter-spacing: -4px; line-height: 1.05; margin: 20px 0 18px; }
h1 span { color: var(--accent); }
.hero-copy { max-width: 590px; margin: auto; color: var(--muted); line-height: 1.8; font-size: 15px; }
.workspace { display: grid; grid-template-columns: 1fr 1fr; gap: 18px; align-items: stretch; }
.panel { background: linear-gradient(145deg, #121c2d, #0d1523); border: 1px solid var(--line); border-radius: 18px; padding: 24px; min-width: 0; box-shadow: 0 22px 55px #0002; }
.panel-heading { display: flex; align-items: center; justify-content: space-between; gap: 10px; margin-bottom: 25px; }
.panel-heading > div { display: flex; align-items: center; gap: 11px; }
.step { color: var(--accent); font: 500 12px 'DM Mono', monospace; }
h2, h3, p { margin-top: 0; }
.panel h2 { font-size: 16px; margin: 0; letter-spacing: -.3px; }
.optional, .result-tag { font: 10px 'DM Mono', monospace; letter-spacing: 1px; color: var(--muted); }
label { display: block; color: #d5dceb; font-size: 12px; font-weight: 700; margin: 0 0 9px; }
textarea, select {
  width: 100%; border: 1px solid #2a3950; border-radius: 10px; background: #080f1c;
  color: var(--text); padding: 13px 14px; outline: none; resize: vertical; line-height: 1.6; font-size: 13px;
}
textarea:focus, select:focus { border-color: #a6f36a99; box-shadow: 0 0 0 3px #a6f36a12; }
textarea::placeholder { color: #64748b; }
.examples { display: flex; flex-wrap: wrap; gap: 7px; align-items: center; margin: 13px 0 24px; }
.examples > span { color: var(--muted); font-size: 11px; width: 100%; margin-bottom: 2px; }
.example { background: #172337; border: 1px solid #2a3950; color: #c6d2e4; padding: 7px 9px; border-radius: 7px; font-size: 10px; cursor: pointer; }
.example:hover { border-color: var(--accent); color: var(--accent); }
.options { display: grid; grid-template-columns: .8fr 1.2fr; gap: 13px; margin-bottom: 20px; }
.options select { height: 44px; }
.options textarea { min-height: 44px; }
.muted { color: #8290a4; font-weight: 400; }
.primary-button { width: 100%; display: flex; justify-content: space-between; align-items: center; padding: 14px 17px; border: 0; border-radius: 10px; background: var(--accent); color: #12200d; font-weight: 800; cursor: pointer; transition: transform .15s, opacity .15s; }
.primary-button:hover { transform: translateY(-1px); }
.primary-button:disabled { cursor: wait; opacity: .7; }
.privacy-note { color: #8f9eb3; font-size: 10px; margin: 13px 0 0; text-align: center; }
.status { font-size: 12px; margin-top: 10px; color: var(--purple); min-height: 16px; }
.output-card { border: 1px solid #27374c; border-radius: 12px; overflow: hidden; background: #080f1a; }
.code-toolbar { display: flex; align-items: center; justify-content: space-between; background: #131f30; border-bottom: 1px solid #27374c; padding: 11px 13px; }
.code-toolbar > span { color: #8d9db2; font: 11px 'DM Mono', monospace; }
.code-toolbar i { display: inline-block; width: 7px; height: 7px; background: #56667b; border-radius: 50%; margin-right: 4px; }
.code-toolbar b { margin-left: 7px; font-weight: 400; }
.copy-button { border: 1px solid #35465d; background: #1a2a3d; color: #dbe6f5; border-radius: 6px; padding: 5px 9px; cursor: pointer; font-size: 10px; }
.copy-button:disabled { opacity: .45; cursor: not-allowed; }
pre { margin: 0; padding: 19px 16px; min-height: 150px; overflow: auto; white-space: pre-wrap; overflow-wrap: anywhere; }
code { font: 12px/1.8 'DM Mono', monospace; color: #c6f5a5; }
.explanation { margin-top: 23px; }
.explanation h3 { font-size: 13px; margin-bottom: 10px; }
.bulb { color: var(--accent); margin-right: 7px; }
.explanation p { font-size: 12px; color: #aebbd0; line-height: 1.8; margin-bottom: 0; white-space: pre-wrap; }
.warning { margin-top: 20px; border: 1px solid #51472d; background: #29241780; border-radius: 9px; padding: 12px; color: #c8b995; font-size: 10px; line-height: 1.7; }
.warning strong { color: #f1d69a; }
.how { padding: 82px 0 60px; text-align: center; }
.how h2 { font-size: clamp(25px, 4vw, 38px); letter-spacing: -1.5px; margin: 15px 0 32px; }
.steps { display: grid; grid-template-columns: repeat(3, 1fr); gap: 15px; text-align: left; }
.steps article { padding: 22px; background: #0d1523; border: 1px solid var(--line); border-radius: 14px; }
.steps article > span { color: var(--accent); font: 12px 'DM Mono', monospace; }
.steps h3 { font-size: 15px; margin: 16px 0 9px; }
.steps p { color: var(--muted); font-size: 12px; line-height: 1.8; margin: 0; }
footer { display: flex; justify-content: space-between; gap: 15px; padding: 24px 0 32px; border-top: 1px solid #ffffff12; color: #8795aa; font-size: 11px; }
footer span:first-child { color: #d8e0ef; font-weight: 700; }
@media (max-width: 800px) {
  .workspace { grid-template-columns: 1fr; }
  .hero { padding-top: 58px; }
  .panel { padding: 20px; }
}
@media (max-width: 520px) {
  .topbar { padding: 18px 16px; }
  .brand { font-size: 12px; }
  .brand-icon { width: 30px; height: 30px; }
  .page { padding: 0 15px; }
  h1 { letter-spacing: -2px; }
  .hero-copy { font-size: 13px; }
  .options { grid-template-columns: 1fr; }
  .steps { grid-template-columns: 1fr; }
  footer { flex-direction: column; }
  .panel-heading { align-items: flex-start; }
  .optional, .result-tag { padding-top: 3px; }
}
const questionInput = document.getElementById("question");
const schemaInput = document.getElementById("schema");
const dialectInput = document.getElementById("dialect");
const generateButton = document.getElementById("generate");
const sqlOutput = document.getElementById("sql-output");
const explanationOutput = document.getElementById("explanation-output");
const statusOutput = document.getElementById("status");
const copyButton = document.getElementById("copy-sql");

document.querySelectorAll(".example").forEach((button) => {
  button.addEventListener("click", () => {
    questionInput.value = button.dataset.question;
    questionInput.focus();
  });
});

generateButton.addEventListener("click", generateSQL);

async function generateSQL() {
  const question = questionInput.value.trim();
  const schema = schemaInput.value.trim();
  const dialect = dialectInput.value;

  if (!question) {
    statusOutput.textContent = "Please type a question first.";
    questionInput.focus();
    return;
  }

  generateButton.disabled = true;
  generateButton.querySelector("span").textContent = "Generating…";
  statusOutput.textContent = "Asking the AI to prepare your query…";
  copyButton.disabled = true;

  try {
    const response = await fetch("/.netlify/functions/generate-sql", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ question, schema, dialect })
    });

    const data = await response.json();
    if (!response.ok) throw new Error(data.error || "Could not generate SQL.");

    sqlOutput.textContent = data.sql || "No SQL was returned.";
    explanationOutput.textContent = data.explanation || "No explanation was returned.";
    copyButton.disabled = !data.sql;
    statusOutput.textContent = "Query generated. Review it before using it.";
  } catch (error) {
    statusOutput.textContent = error.message || "Something went wrong. Please try again.";
  } finally {
    generateButton.disabled = false;
    generateButton.querySelector("span").textContent = "Generate SQL";
  }
}

copyButton.addEventListener("click", async () => {
  const sql = sqlOutput.textContent;
  try {
    await navigator.clipboard.writeText(sql);
    copyButton.textContent = "Copied!";
    setTimeout(() => { copyButton.textContent = "Copy"; }, 1500);
  } catch {
    statusOutput.textContent = "Copy failed. Please select and copy the SQL manually.";
  }
});
SQL in Plain English
A beginner-friendly website that converts a plain-English question into a SQL query and a short explanation using the Gemini API.
Files
index.html — website structure
style.css — responsive visual design
script.js — form handling, example prompts, API request, and copy button
netlify/functions/generate-sql.mjs — secure serverless function that calls Gemini
netlify.toml — Netlify publish and function configuration
Deploy to Netlify
Upload these project files to a GitHub repository.
In Netlify, choose Add new project → Import an existing project.
Connect GitHub and select this repository.
For this static site, the publish directory is .. The included netlify.toml sets it automatically.
In Netlify, open Site configuration → Environment variables (the exact menu label may vary).
Add GEMINI_API_KEY with your own Gemini API key as the value.
Redeploy the site after adding the environment variable.
Get an API key from Google AI Studio:Push your ideas to the web | Netlify https://www.netlify.com/
Security
Do not place the real Gemini API key in script.js, HTML, or any public GitHub file.
Use the Netlify environment variable GEMINI_API_KEY; the serverless function reads it on the server.
The app generates SQL text only. It does not connect to or execute SQL on a database.
Review all AI-generated queries before using them, especially UPDATE, DELETE, or other data-changing statements.
Local development
Install Node.js, then run:
npm install -g netlify-cli
netlify dev
Set GEMINI_API_KEY in the local environment before running netlify dev. Never commit a real key.
