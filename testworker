worker.js
import fetch from "node-fetch"

const SUPABASE_URL = process.env.SUPABASE_URL
const SUPABASE_SERVICE_KEY = process.env.SUPABASE_SERVICE_KEY
const KIMI_API_KEY = process.env.KIMI_API_KEY

async function getTask() {
  const res = await fetch(
    `${SUPABASE_URL}/rest/v1/task_queue?status=eq.pending&limit=1`,
    {
      headers: {
        apikey: SUPABASE_SERVICE_KEY,
        Authorization: `Bearer ${SUPABASE_SERVICE_KEY}`,
      },
    }
  )

  const data = await res.json()
  return data[0]
}

async function markComplete(id, result) {
  await fetch(`${SUPABASE_URL}/rest/v1/task_queue?id=eq.${id}`, {
    method: "PATCH",
    headers: {
      "Content-Type": "application/json",
      apikey: SUPABASE_SERVICE_KEY,
      Authorization: `Bearer ${SUPABASE_SERVICE_KEY}`,
    },
    body: JSON.stringify({
      status: "complete",
      result,
    }),
  })
}

async function runKimi(prompt) {
  const res = await fetch("https://api.moonshot.cn/v1/chat/completions", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${KIMI_API_KEY}`,
    },
    body: JSON.stringify({
      model: "moonshot-v1-32k",
      messages: [
        {
          role: "system",
          content:
            "You are a senior engineer finishing a web platform. Fix code and complete tasks cleanly.",
        },
        {
          role: "user",
          content: prompt,
        },
      ],
    }),
  })

  const data = await res.json()
  return data.choices[0].message.content
}

async function loop() {
  console.log("Worker running...")

  while (true) {
    try {
      const task = await getTask()

      if (!task) {
        await new Promise((r) => setTimeout(r, 5000))
        continue
      }

      console.log("Processing task:", task.id)

      const result = await runKimi(task.payload)

      await markComplete(task.id, result)
    } catch (e) {
      console.error(e)
    }

    await new Promise((r) => setTimeout(r, 2000))
  }
}

loop()
