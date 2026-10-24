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
