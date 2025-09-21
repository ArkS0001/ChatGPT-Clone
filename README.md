# ChatGPT-Clone
Same.dev
https://same.dev/chat/


* Groq API integration
* **Dropdown menu** for model selection
* **API Key input** (user-provided)
* **Models included**:

  * `meta-llama/llama-4-maverick-17b-128e-instruct`
  * `meta-llama/llama-4-scout-17b-16e-instruct`
  * `moonshotai/kimi-k2-instruct-0905`
  * `openai/gpt-oss-120b`
  * `llama-3.3-70b-versatile`
  * `openai/gpt-oss-20b`
  * `deepseek-r1-distill-llama-70b`
* **Side menu** with settings (API key, memory toggle, chat history management).
* **Futuristic UI** (dark, neon glow, smooth animations).
* **Chat memory** across messages.

Here’s a **React full app scaffold** you can run directly (e.g. `create-vite` or `create-react-app`).

---

### 📂 File Structure

```
chatgpt-groq-clone/
│── src/
│   ├── App.tsx
│   ├── components/
│   │   ├── ChatWindow.tsx
│   │   ├── MessageBubble.tsx
│   │   ├── Sidebar.tsx
│   │   └── Settings.tsx
│   ├── index.css
│   └── main.tsx
│── package.json
```

---

### `App.tsx`

```tsx
import { useState } from "react";
import ChatWindow from "./components/ChatWindow";
import Sidebar from "./components/Sidebar";

export default function App() {
  const [apiKey, setApiKey] = useState("");
  const [model, setModel] = useState("openai/gpt-oss-120b");
  const [messages, setMessages] = useState<any[]>([]);
  const [showSidebar, setShowSidebar] = useState(false);

  return (
    <div className="flex h-screen bg-[#0b0f19] text-white">
      {/* Sidebar */}
      <Sidebar
        apiKey={apiKey}
        setApiKey={setApiKey}
        model={model}
        setModel={setModel}
        showSidebar={showSidebar}
        setShowSidebar={setShowSidebar}
      />

      {/* Main Chat */}
      <ChatWindow
        apiKey={apiKey}
        model={model}
        messages={messages}
        setMessages={setMessages}
        toggleSidebar={() => setShowSidebar(!showSidebar)}
      />
    </div>
  );
}
```

---

### `components/Sidebar.tsx`

```tsx
const MODELS = [
  "meta-llama/llama-4-maverick-17b-128e-instruct",
  "meta-llama/llama-4-scout-17b-16e-instruct",
  "moonshotai/kimi-k2-instruct-0905",
  "openai/gpt-oss-120b",
  "llama-3.3-70b-versatile",
  "openai/gpt-oss-20b",
  "deepseek-r1-distill-llama-70b",
];

export default function Sidebar({
  apiKey,
  setApiKey,
  model,
  setModel,
  showSidebar,
  setShowSidebar,
}: any) {
  return (
    <div
      className={`${
        showSidebar ? "w-64" : "w-0"
      } transition-all duration-300 bg-[#111827] border-r border-cyan-500 overflow-hidden`}
    >
      <div className="p-4 space-y-4">
        <h2 className="text-xl font-bold text-cyan-400">⚙️ Settings</h2>
        <div>
          <label className="text-sm">API Key</label>
          <input
            type="password"
            className="w-full p-2 mt-1 bg-black border border-cyan-500 rounded"
            value={apiKey}
            onChange={(e) => setApiKey(e.target.value)}
            placeholder="Enter Groq API Key"
          />
        </div>
        <div>
          <label className="text-sm">Model</label>
          <select
            className="w-full p-2 mt-1 bg-black border border-cyan-500 rounded"
            value={model}
            onChange={(e) => setModel(e.target.value)}
          >
            {MODELS.map((m) => (
              <option key={m} value={m}>
                {m}
              </option>
            ))}
          </select>
        </div>
      </div>
    </div>
  );
}
```

---

### `components/ChatWindow.tsx`

```tsx
import { useState } from "react";
import MessageBubble from "./MessageBubble";

export default function ChatWindow({
  apiKey,
  model,
  messages,
  setMessages,
  toggleSidebar,
}: any) {
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);

  async function sendMessage() {
    if (!input.trim()) return;

    const userMessage = { role: "user", content: input };
    setMessages([...messages, userMessage]);
    setInput("");
    setLoading(true);

    try {
      const res = await fetch("https://api.groq.com/openai/v1/chat/completions", {
        method: "POST",
        headers: {
          Authorization: `Bearer ${apiKey}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          model,
          messages: [...messages, userMessage],
        }),
      });

      const data = await res.json();
      const botMessage = {
        role: "assistant",
        content: data.choices?.[0]?.message?.content || "No response",
      };
      setMessages((prev: any) => [...prev, botMessage]);
    } catch (err) {
      console.error(err);
      setMessages((prev: any) => [
        ...prev,
        { role: "assistant", content: "❌ Error fetching response." },
      ]);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="flex flex-col flex-1">
      {/* Header */}
      <div className="flex items-center justify-between p-4 bg-[#111827] border-b border-cyan-500">
        <h1 className="text-xl font-bold text-cyan-400">🤖 GroqGPT Clone</h1>
        <button
          onClick={toggleSidebar}
          className="px-3 py-1 bg-cyan-500 text-black rounded hover:bg-cyan-400"
        >
          ⚙️
        </button>
      </div>

      {/* Messages */}
      <div className="flex-1 p-4 overflow-y-auto space-y-3">
        {messages.map((msg: any, i: number) => (
          <MessageBubble key={i} role={msg.role} content={msg.content} />
        ))}
        {loading && <p className="text-cyan-400">⏳ Thinking...</p>}
      </div>

      {/* Input */}
      <div className="flex p-4 bg-[#111827] border-t border-cyan-500">
        <input
          type="text"
          className="flex-1 p-2 bg-black border border-cyan-500 rounded mr-2"
          placeholder="Type a message..."
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={(e) => e.key === "Enter" && sendMessage()}
        />
        <button
          onClick={sendMessage}
          className="px-4 py-2 bg-cyan-500 text-black rounded hover:bg-cyan-400"
        >
          ➤
        </button>
      </div>
    </div>
  );
}
```

---

### `components/MessageBubble.tsx`

```tsx
export default function MessageBubble({ role, content }: any) {
  const isUser = role === "user";
  return (
    <div
      className={`p-3 rounded-2xl max-w-xl ${
        isUser
          ? "ml-auto bg-cyan-500 text-black"
          : "mr-auto bg-[#1f2937] text-cyan-300 border border-cyan-500"
      }`}
    >
      {content}
    </div>
  );
}
```

---

### `index.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  @apply bg-[#0b0f19] text-white;
  font-family: "Orbitron", sans-serif;
}
```

---

✨ Features included:

* **Futuristic neon-cyan dark UI**
* **API key + model selector in sidebar**
* **Chat memory persists in state**
* **ChatGPT-like bubbles**
* **Error handling & loader state**

---

👉 Do you want me to **add localStorage persistence** for chat history + API key, so it remembers across reloads?
