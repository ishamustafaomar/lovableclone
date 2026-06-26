# lovable-clone — Full Source Code

This document contains every source file in the project, in one place, for handing to Lovable.

Build artifacts (`.next/`), `node_modules`, lockfiles, and binary assets are excluded.


## File index

- `readMe.md`
- `README.md`
- `CLAUDE.md`
- `gitignore`
- `package.json`
- `generateWithClaudeCode.ts`
- `test-claude-code.ts`
- `tictactoe.html`
- `lovable-ui/package.json`
- `lovable-ui/tsconfig.json`
- `lovable-ui/next.config.mjs`
- `lovable-ui/postcss.config.mjs`
- `lovable-ui/tailwind.config.ts`
- `lovable-ui/next-env.d.ts`
- `lovable-ui/.gitignore`
- `lovable-ui/app/layout.tsx`
- `lovable-ui/app/page.tsx`
- `lovable-ui/app/globals.css`
- `lovable-ui/app/generate/page.tsx`
- `lovable-ui/app/connect4/page.tsx`
- `lovable-ui/app/hello-world/page.tsx`
- `lovable-ui/app/api/generate/route.ts`
- `lovable-ui/app/api/generate-daytona/route.ts`
- `lovable-ui/components/Navbar.tsx`
- `lovable-ui/components/MessageDisplay.tsx`
- `lovable-ui/lib/claude-code.ts`
- `lovable-ui/scripts/generate-in-daytona.ts`
- `lovable-ui/scripts/get-preview-url.ts`
- `lovable-ui/scripts/start-dev-server.ts`
- `lovable-ui/scripts/test-preview-url.ts`
- `lovable-ui/scripts/remove-sandbox.ts`

---


## `readMe.md`

````markdown
# Lovable Clone

Thank you so much for checking out this project! 🙏  
We appreciate your interest and hope you enjoy exploring and building with it.

## Getting Started

Before you begin, please make sure to **replace the API keys** in your `.env` file:

- Get your Anthropic API key from: [Anthropic Console](https://console.anthropic.com/dashboard)
- Get your Daytona API key from: [Daytona Dashboard](https://www.daytona.io/)

Add these keys to your `.env` file as follows:

``` .env
ANTHROPIC_API_KEY=your_anthropic_api_key
DAYTONA_API_KEY=your_daytona_api_key
```

## Install & Run

From the `lovable-ui` directory, install all dependencies and start the development server:


```bash
cd lovable-ui

npm install
npm run dev
```

This will launch the app locally (by default at http://localhost:3000).
````


## `README.md`

```markdown
# lovableclone
clone of the clone of lovable
```


## `CLAUDE.md`

```markdown
## Project Goals
- I am building a lovable clone, but I want to use the claude-code sdk

## Preferences
- don't try to run the script with your own bash tool. Write the script and tell me how to execute it, asking me for its output instead.

## Progress so far
- We have a website that takes in a prompt, uses claude code SDK to write code. But currently, it directly modifies my websites code by adding it as a page. The next task we are going to work on is making the code gen happen in an isolated environment and opening the dev server there.
- We have created a way to create sandboxes using daytona and preview them in using the getPreviewLink() function. The script scripts/test-preview-url.ts confirms this.
```


## `gitignore`

```text
node_modules/
.env
.env.local
dist/
*.log
```


## `package.json`

```json
{
  "name": "lovable-clone",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "",
  "dependencies": {
    "@anthropic-ai/claude-code": "^1.0.39",
    "@types/node": "^24.0.10",
    "tsx": "^4.20.3",
    "typescript": "^5.8.3"
  }
}
```


## `generateWithClaudeCode.ts`

```ts
import { query, type SDKMessage } from "@anthropic-ai/claude-code";

interface CodeGenerationResult {
  success: boolean;
  messages: SDKMessage[];
  error?: string;
}

export async function generateCodeWithClaude(prompt: string): Promise<CodeGenerationResult> {
  try {
    const messages: SDKMessage[] = [];
    const abortController = new AbortController();
    
    // Execute the query and collect all messages
    for await (const message of query({
      prompt: prompt,
      abortController: abortController,
      options: {
        maxTurns: 10, // Allow multiple turns for complex builds
        // Grant all necessary permissions for code generation
        allowedTools: [
          "Read",
          "Write",
          "Edit",
          "MultiEdit",
          "Bash",
          "LS",
          "Glob",
          "Grep",
          "WebSearch",
          "WebFetch"
        ]
      }
    })) {
      messages.push(message);
      
      // Log each message for debugging
      console.log(`[${message.type}]`, message);
    }
    
    return {
      success: true,
      messages: messages
    };
    
  } catch (error: any) {
    console.error("Error generating code:", error);
    return {
      success: false,
      messages: [],
      error: error.message
    };
  }
}
```


## `test-claude-code.ts`

```ts
import { generateCodeWithClaude } from "./generateWithClaudeCode";

async function testClaudeCodeSDK() {
  console.log("Testing Claude Code SDK by generating a Tic-Tac-Toe game...\n");
  
  const prompt = `Create a simple Tic-Tac-Toe game in a single HTML file with:
  - Complete game logic in JavaScript
  - Player vs Player gameplay
  - Nice CSS styling with a modern look
  - Win detection and game reset functionality
  - Display current player turn
  - Highlight winning combination
  
  Save it as tictactoe.html in the current directory.`;
  
  const result = await generateCodeWithClaude(prompt);
  
  if (result.success) {
    console.log("\n✅ Code generation completed successfully!");
    console.log(`Total messages: ${result.messages.length}`);
    
    // Log summary of actions taken
    const toolUses = result.messages.filter(m => m.type === 'tool_use');
    console.log(`\nTool uses: ${toolUses.length}`);
    toolUses.forEach((msg: any) => {
      console.log(`- ${msg.name} ${msg.input?.file_path || ''}`);
    });
  } else {
    console.error("\n❌ Code generation failed:", result.error);
  }
}

// Run the test
testClaudeCodeSDK().catch(console.error);
```


## `tictactoe.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tic-Tac-Toe</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }

        .container {
            background: white;
            padding: 2rem;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
            text-align: center;
        }

        h1 {
            color: #333;
            margin-bottom: 1rem;
            font-size: 2.5rem;
        }

        .status {
            font-size: 1.25rem;
            color: #666;
            margin-bottom: 1.5rem;
            min-height: 30px;
        }

        .board {
            display: grid;
            grid-template-columns: repeat(3, 120px);
            grid-template-rows: repeat(3, 120px);
            gap: 10px;
            margin: 0 auto 2rem;
            background: #333;
            padding: 10px;
            border-radius: 10px;
        }

        .cell {
            background: white;
            border: none;
            font-size: 2.5rem;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            color: #333;
        }

        .cell:hover:not(.taken) {
            background: #f0f0f0;
            transform: scale(0.95);
        }

        .cell.taken {
            cursor: not-allowed;
        }

        .cell.x {
            color: #667eea;
        }

        .cell.o {
            color: #764ba2;
        }

        .cell.winner {
            background: #ffe66d;
            animation: pulse 0.5s ease-in-out infinite alternate;
        }

        @keyframes pulse {
            from { transform: scale(1); }
            to { transform: scale(1.05); }
        }

        .reset-btn {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 1.1rem;
            border-radius: 30px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }

        .reset-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
        }

        .reset-btn:active {
            transform: translateY(0);
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Tic-Tac-Toe</h1>
        <div class="status" id="status">Player X's turn</div>
        <div class="board" id="board">
            <button class="cell" data-index="0"></button>
            <button class="cell" data-index="1"></button>
            <button class="cell" data-index="2"></button>
            <button class="cell" data-index="3"></button>
            <button class="cell" data-index="4"></button>
            <button class="cell" data-index="5"></button>
            <button class="cell" data-index="6"></button>
            <button class="cell" data-index="7"></button>
            <button class="cell" data-index="8"></button>
        </div>
        <button class="reset-btn" id="resetBtn">New Game</button>
    </div>

    <script>
        const cells = document.querySelectorAll('.cell');
        const statusDisplay = document.getElementById('status');
        const resetBtn = document.getElementById('resetBtn');
        
        let currentPlayer = 'X';
        let gameActive = true;
        let gameState = ['', '', '', '', '', '', '', '', ''];
        
        const winningConditions = [
            [0, 1, 2],
            [3, 4, 5],
            [6, 7, 8],
            [0, 3, 6],
            [1, 4, 7],
            [2, 5, 8],
            [0, 4, 8],
            [2, 4, 6]
        ];

        function handleCellClick(e) {
            const cell = e.target;
            const index = parseInt(cell.getAttribute('data-index'));
            
            if (gameState[index] !== '' || !gameActive) {
                return;
            }
            
            gameState[index] = currentPlayer;
            cell.textContent = currentPlayer;
            cell.classList.add('taken', currentPlayer.toLowerCase());
            
            checkResult();
        }

        function checkResult() {
            let roundWon = false;
            let winningCombination = [];
            
            for (let i = 0; i < winningConditions.length; i++) {
                const [a, b, c] = winningConditions[i];
                if (gameState[a] === '' || gameState[b] === '' || gameState[c] === '') {
                    continue;
                }
                if (gameState[a] === gameState[b] && gameState[b] === gameState[c]) {
                    roundWon = true;
                    winningCombination = [a, b, c];
                    break;
                }
            }
            
            if (roundWon) {
                statusDisplay.textContent = `Player ${currentPlayer} wins!`;
                gameActive = false;
                highlightWinningCells(winningCombination);
                return;
            }
            
            if (!gameState.includes('')) {
                statusDisplay.textContent = "It's a draw!";
                gameActive = false;
                return;
            }
            
            currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
            statusDisplay.textContent = `Player ${currentPlayer}'s turn`;
        }

        function highlightWinningCells(combination) {
            combination.forEach(index => {
                cells[index].classList.add('winner');
            });
        }

        function resetGame() {
            currentPlayer = 'X';
            gameActive = true;
            gameState = ['', '', '', '', '', '', '', '', ''];
            statusDisplay.textContent = `Player ${currentPlayer}'s turn`;
            
            cells.forEach(cell => {
                cell.textContent = '';
                cell.classList.remove('taken', 'x', 'o', 'winner');
            });
        }

        cells.forEach(cell => cell.addEventListener('click', handleCellClick));
        resetBtn.addEventListener('click', resetGame);
    </script>
</body>
</html>
```


## `lovable-ui/package.json`

```json
{
  "name": "lovable-ui",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@anthropic-ai/claude-code": "^1.0.39",
    "@daytonaio/sdk": "^0.21.5",
    "dotenv": "^17.0.1",
    "next": "14.2.3",
    "react": "^18",
    "react-dom": "^18"
  },
  "devDependencies": {
    "@types/node": "^20",
    "@types/react": "^18",
    "@types/react-dom": "^18",
    "autoprefixer": "^10.0.1",
    "postcss": "^8",
    "tailwindcss": "^3.4.1",
    "typescript": "^5"
  }
}
```


## `lovable-ui/tsconfig.json`

```json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```


## `lovable-ui/next.config.mjs`

```js
/** @type {import('next').NextConfig} */
const nextConfig = {};

export default nextConfig;
```


## `lovable-ui/postcss.config.mjs`

```js
/** @type {import('postcss-load-config').Config} */
const config = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

export default config;
```


## `lovable-ui/tailwind.config.ts`

```ts
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      backgroundImage: {
        "gradient-radial": "radial-gradient(var(--tw-gradient-stops))",
        "gradient-conic":
          "conic-gradient(from 180deg at 50% 50%, var(--tw-gradient-stops))",
      },
    },
  },
  plugins: [],
};
export default config;
```


## `lovable-ui/next-env.d.ts`

```ts
/// <reference types="next" />
/// <reference types="next/image-types/global" />

// NOTE: This file should not be edited
// see https://nextjs.org/docs/basic-features/typescript for more information.
```


## `lovable-ui/.gitignore`

```text
# See https://help.github.com/articles/ignoring-files/ for more about ignoring files.

# dependencies
/node_modules
/.pnp
.pnp.js
.yarn/install-state.gz

# testing
/coverage

# next.js
/.next/
/out/

# production
/build

# misc
.DS_Store
*.pem

# debug
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# local env files
.env*.local
.env

# vercel
.vercel

# typescript
*.tsbuildinfo
next-env.d.ts
```


## `lovable-ui/app/layout.tsx`

```tsx
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
  title: "Lovable Clone - AI-Powered Code Generation",
  description: "Build applications faster with AI-powered code generation",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```


## `lovable-ui/app/page.tsx`

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import Navbar from "@/components/Navbar";

export default function Home() {
  const router = useRouter();
  const [prompt, setPrompt] = useState("");

  const handleGenerate = () => {
    if (!prompt.trim()) return;

    // Navigate to generate page with prompt
    router.push(`/generate?prompt=${encodeURIComponent(prompt)}`);
  };

  return (
    <main className="min-h-screen relative overflow-hidden bg-black">
      {/* Navbar */}
      <Navbar />

      {/* Background image */}
      <div
        className="absolute inset-0 z-0 bg-cover bg-center"
        style={{ backgroundImage: "url('/gradient.png')" }}
      />

      {/* Content */}
      <div className="relative z-10 flex flex-col items-center justify-center min-h-screen px-4 sm:px-6 lg:px-8">
        <div className="max-w-4xl mx-auto text-center">
          {/* Hero Section */}
          <h1 className="text-4xl sm:text-4xl md:text-4xl font-bold text-white mb-6">
            Build something with Lovable-clone
          </h1>
          <h3 className="text-xl sm:text-xl text-gray-300 mb-12 max-w-2xl mx-auto">
            BUILT WITH CLAUDE CODE
          </h3>

          <p className="text-xl sm:text-xl text-gray-300 mb-12 max-w-2xl mx-auto">
            Turn your ideas into production-ready code in minutes. Powered by
            Claude's advanced AI capabilities.
          </p>

          {/* Input Section */}
          <div className="relative max-w-2xl mx-auto">
            <div className="relative flex items-center bg-black rounded-2xl border border-gray-800 shadow-2xl px-2">
              {/* Textarea */}
              <textarea
                placeholder="Ask Lovable to create a prototype..."
                value={prompt}
                onChange={(e) => setPrompt(e.target.value)}
                onKeyDown={(e) => {
                  if (e.key === "Enter" && !e.shiftKey) {
                    e.preventDefault();
                    handleGenerate();
                  }
                }}
                className="flex-1 px-5 py-4 bg-transparent text-white placeholder-gray-500 focus:outline-none text-lg resize-none min-h-[120px] max-h-[300px]"
                rows={3}
              />

              {/* Send button */}
              <button
                onClick={handleGenerate}
                disabled={!prompt.trim()}
                className="flex-shrink-0 mr-3 p-3 bg-gray-800 hover:bg-gray-700 text-white rounded-xl focus:outline-none focus:ring-2 focus:ring-gray-600 disabled:opacity-50 disabled:cursor-not-allowed transition-all duration-200 group"
              >
                {false ? (
                  <svg
                    className="animate-spin h-5 w-5 text-white"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                  >
                    <circle
                      className="opacity-25"
                      cx="12"
                      cy="12"
                      r="10"
                      stroke="currentColor"
                      strokeWidth="4"
                    ></circle>
                    <path
                      className="opacity-75"
                      fill="currentColor"
                      d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
                    ></path>
                  </svg>
                ) : (
                  <svg
                    className="h-5 w-5 group-hover:scale-110 transition-transform"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                  >
                    <path
                      strokeLinecap="round"
                      strokeLinejoin="round"
                      strokeWidth={2}
                      d="M5 10l7-7m0 0l7 7m-7-7v18"
                    />
                  </svg>
                )}
              </button>
            </div>

            {/* Example prompts */}
            <div className="mt-8 flex flex-wrap justify-center gap-3">
              <button
                onClick={() =>
                  setPrompt(
                    "Create a modern blog website with markdown support"
                  )
                }
                className="px-4 py-2 text-sm text-gray-400 bg-gray-800/50 backdrop-blur-sm rounded-full hover:bg-gray-700/50 transition-colors border border-gray-700"
              >
                Blog website
              </button>
              <button
                onClick={() =>
                  setPrompt("Build a portfolio website with project showcase")
                }
                className="px-4 py-2 text-sm text-gray-400 bg-gray-800/50 backdrop-blur-sm rounded-full hover:bg-gray-700/50 transition-colors border border-gray-700"
              >
                Portfolio site
              </button>
              <button
                onClick={() =>
                  setPrompt(
                    "Create an e-commerce product catalog with shopping cart"
                  )
                }
                className="px-4 py-2 text-sm text-gray-400 bg-gray-800/50 backdrop-blur-sm rounded-full hover:bg-gray-700/50 transition-colors border border-gray-700"
              >
                E-commerce
              </button>
              <button
                onClick={() =>
                  setPrompt(
                    "Build a dashboard with charts and data visualization"
                  )
                }
                className="px-4 py-2 text-sm text-gray-400 bg-gray-800/50 backdrop-blur-sm rounded-full hover:bg-gray-700/50 transition-colors border border-gray-700"
              >
                Dashboard
              </button>
            </div>
          </div>
        </div>
      </div>

      <style jsx>{`
        @keyframes blob {
          0% {
            transform: translate(0px, 0px) scale(1);
          }
          33% {
            transform: translate(30px, -50px) scale(1.1);
          }
          66% {
            transform: translate(-20px, 20px) scale(0.9);
          }
          100% {
            transform: translate(0px, 0px) scale(1);
          }
        }
        .animate-blob {
          animation: blob 7s infinite;
        }
        .animation-delay-2000 {
          animation-delay: 2s;
        }
        .animation-delay-4000 {
          animation-delay: 4s;
        }
      `}</style>
    </main>
  );
}
```


## `lovable-ui/app/globals.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --foreground-rgb: 255, 255, 255;
  --background-rgb: 0, 0, 0;
}

body {
  color: rgb(var(--foreground-rgb));
  background: rgb(var(--background-rgb));
}

/* Custom scrollbar for textarea */
textarea::-webkit-scrollbar {
  width: 8px;
}

textarea::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
}

textarea::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 4px;
}

textarea::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.3);
}
```


## `lovable-ui/app/generate/page.tsx`

```tsx
"use client";

import { useState, useEffect, useRef } from "react";
import { useSearchParams, useRouter } from "next/navigation";
import Navbar from "@/components/Navbar";

interface Message {
  type: "claude_message" | "tool_use" | "tool_result" | "progress" | "error" | "complete";
  content?: string;
  name?: string;
  input?: any;
  result?: any;
  message?: string;
  previewUrl?: string;
  sandboxId?: string;
}

export default function GeneratePage() {
  const searchParams = useSearchParams();
  const router = useRouter();
  const prompt = searchParams.get("prompt") || "";
  
  const [messages, setMessages] = useState<Message[]>([]);
  const [previewUrl, setPreviewUrl] = useState<string | null>(null);
  const [isGenerating, setIsGenerating] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const messagesEndRef = useRef<HTMLDivElement>(null);
  const hasStartedRef = useRef(false);
  
  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  };
  
  useEffect(() => {
    scrollToBottom();
  }, [messages]);
  
  useEffect(() => {
    if (!prompt) {
      router.push("/");
      return;
    }
    
    // Prevent double execution in StrictMode
    if (hasStartedRef.current) {
      return;
    }
    hasStartedRef.current = true;
    
    setIsGenerating(true);
    generateWebsite();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [prompt, router]);
  
  const generateWebsite = async () => {
    try {
      const response = await fetch("/api/generate-daytona", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ prompt }),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.error || "Failed to generate website");
      }

      const reader = response.body?.getReader();
      const decoder = new TextDecoder();

      if (!reader) {
        throw new Error("No response body");
      }

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        const chunk = decoder.decode(value);
        const lines = chunk.split("\n");

        for (const line of lines) {
          if (line.startsWith("data: ")) {
            const data = line.slice(6);

            if (data === "[DONE]") {
              setIsGenerating(false);
              break;
            }

            try {
              const message = JSON.parse(data) as Message;
              
              if (message.type === "error") {
                throw new Error(message.message);
              } else if (message.type === "complete") {
                setPreviewUrl(message.previewUrl || null);
                setIsGenerating(false);
              } else {
                setMessages((prev) => [...prev, message]);
              }
            } catch (e) {
              // Ignore parse errors
            }
          }
        }
      }
    } catch (err: any) {
      console.error("Error generating website:", err);
      setError(err.message || "An error occurred");
      setIsGenerating(false);
    }
  };
  
  const formatToolInput = (input: any) => {
    if (!input) return "";
    
    // Extract key information based on tool type
    if (input.file_path) {
      return `File: ${input.file_path}`;
    } else if (input.command) {
      return `Command: ${input.command}`;
    } else if (input.pattern) {
      return `Pattern: ${input.pattern}`;
    } else if (input.prompt) {
      return `Prompt: ${input.prompt.substring(0, 100)}...`;
    }
    
    // For other cases, show first meaningful field
    const keys = Object.keys(input);
    if (keys.length > 0) {
      const firstKey = keys[0];
      const value = input[firstKey];
      if (typeof value === 'string' && value.length > 100) {
        return `${firstKey}: ${value.substring(0, 100)}...`;
      }
      return `${firstKey}: ${value}`;
    }
    
    return JSON.stringify(input).substring(0, 100) + "...";
  };

  return (
    <main className="h-screen bg-black flex flex-col overflow-hidden relative">
      <Navbar />
      {/* Spacer for navbar */}
      <div className="h-16" />
      
      <div className="flex-1 flex overflow-hidden">
        {/* Left side - Chat */}
        <div className="w-[30%] flex flex-col border-r border-gray-800">
          {/* Header */}
          <div className="p-4 border-b border-gray-800">
            <h2 className="text-white font-semibold">Lovable</h2>
            <p className="text-gray-400 text-sm mt-1 break-words">{prompt}</p>
          </div>
          
          {/* Messages */}
          <div className="flex-1 overflow-y-auto p-4 space-y-4 overflow-x-hidden">
            {messages.map((message, index) => (
              <div key={index}>
                {message.type === "claude_message" && (
                  <div className="bg-gray-900 rounded-lg p-4">
                    <div className="flex items-center gap-2 mb-2">
                      <div className="w-6 h-6 bg-purple-600 rounded-full flex items-center justify-center">
                        <span className="text-white text-xs">L</span>
                      </div>
                      <span className="text-white font-medium">Lovable</span>
                    </div>
                    <p className="text-gray-300 whitespace-pre-wrap break-words">{message.content}</p>
                  </div>
                )}
                
                {message.type === "tool_use" && (
                  <div className="bg-gray-900/50 rounded-lg p-3 border border-gray-800 overflow-hidden">
                    <div className="flex items-start gap-2 text-sm">
                      <span className="text-blue-400 flex-shrink-0">🔧 {message.name}</span>
                      <span className="text-gray-500 break-all">{formatToolInput(message.input)}</span>
                    </div>
                  </div>
                )}
                
                {message.type === "progress" && (
                  <div className="text-gray-500 text-sm font-mono break-all">
                    {message.message}
                  </div>
                )}
              </div>
            ))}
            
            {isGenerating && (
              <div className="flex items-center gap-2 text-gray-400">
                <div className="animate-spin rounded-full h-4 w-4 border-b-2 border-gray-400"></div>
                <span>Working...</span>
              </div>
            )}
            
            {error && (
              <div className="bg-red-900/20 border border-red-700 rounded-lg p-4">
                <p className="text-red-400">{error}</p>
              </div>
            )}
            
            <div ref={messagesEndRef} />
          </div>
          
          {/* Bottom input area */}
          <div className="p-4 border-t border-gray-800">
            <div className="flex items-center gap-2">
              <input
                type="text"
                placeholder="Ask Lovable..."
                className="flex-1 px-4 py-2 bg-gray-900 text-white rounded-lg border border-gray-800 focus:outline-none focus:border-gray-700"
                disabled={isGenerating}
              />
              <button className="p-2 text-gray-400 hover:text-gray-300">
                <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" />
                </svg>
              </button>
              <button className="p-2 text-gray-400 hover:text-gray-300">
                <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10" />
                </svg>
              </button>
            </div>
          </div>
        </div>
        
        {/* Right side - Preview */}
        <div className="w-[70%] bg-gray-950 flex items-center justify-center">
          {!previewUrl && isGenerating && (
            <div className="text-center">
              <div className="w-16 h-16 bg-gray-800 rounded-2xl flex items-center justify-center mb-4">
                <div className="w-12 h-12 bg-gray-700 rounded-xl animate-pulse"></div>
              </div>
              <p className="text-gray-400">Spinning up preview...</p>
            </div>
          )}
          
          {previewUrl && (
            <iframe
              src={previewUrl}
              className="w-full h-full"
              title="Website Preview"
            />
          )}
          
          {!previewUrl && !isGenerating && (
            <div className="text-center">
              <p className="text-gray-400">Preview will appear here</p>
            </div>
          )}
        </div>
      </div>
    </main>
  );
}
```


## `lovable-ui/app/connect4/page.tsx`

```tsx
"use client";

import { useState, useCallback } from "react";

type Player = 1 | 2;
type Cell = 0 | Player;
type Board = Cell[][];

const ROWS = 6;
const COLS = 7;
const EMPTY = 0;
const PLAYER1 = 1;
const PLAYER2 = 2;

export default function Connect4() {
  const [board, setBoard] = useState<Board>(() => 
    Array(ROWS).fill(null).map(() => Array(COLS).fill(EMPTY))
  );
  const [currentPlayer, setCurrentPlayer] = useState<Player>(PLAYER1);
  const [winner, setWinner] = useState<Player | null>(null);
  const [isDraw, setIsDraw] = useState(false);

  const checkWinner = useCallback((board: Board, row: number, col: number, player: Player): boolean => {
    // Check horizontal
    let count = 0;
    for (let c = 0; c < COLS; c++) {
      if (board[row][c] === player) {
        count++;
        if (count === 4) return true;
      } else {
        count = 0;
      }
    }

    // Check vertical
    count = 0;
    for (let r = 0; r < ROWS; r++) {
      if (board[r][col] === player) {
        count++;
        if (count === 4) return true;
      } else {
        count = 0;
      }
    }

    // Check diagonal (top-left to bottom-right)
    const startRow1 = Math.max(0, row - col);
    const startCol1 = Math.max(0, col - row);
    count = 0;
    for (let i = 0; i < Math.min(ROWS - startRow1, COLS - startCol1); i++) {
      if (board[startRow1 + i][startCol1 + i] === player) {
        count++;
        if (count === 4) return true;
      } else {
        count = 0;
      }
    }

    // Check diagonal (top-right to bottom-left)
    const startRow2 = Math.max(0, row - (COLS - 1 - col));
    const startCol2 = Math.min(COLS - 1, col + row);
    count = 0;
    for (let i = 0; i < Math.min(ROWS - startRow2, startCol2 + 1); i++) {
      if (board[startRow2 + i][startCol2 - i] === player) {
        count++;
        if (count === 4) return true;
      } else {
        count = 0;
      }
    }

    return false;
  }, []);

  const checkDraw = useCallback((board: Board): boolean => {
    return board[0].every(cell => cell !== EMPTY);
  }, []);

  const dropPiece = useCallback((col: number) => {
    if (winner || isDraw || board[0][col] !== EMPTY) return;

    const newBoard = board.map(row => [...row]);
    
    // Find the lowest empty row in the column
    let row = ROWS - 1;
    while (row >= 0 && newBoard[row][col] !== EMPTY) {
      row--;
    }

    if (row < 0) return;

    newBoard[row][col] = currentPlayer;
    setBoard(newBoard);

    if (checkWinner(newBoard, row, col, currentPlayer)) {
      setWinner(currentPlayer);
    } else if (checkDraw(newBoard)) {
      setIsDraw(true);
    } else {
      setCurrentPlayer(currentPlayer === PLAYER1 ? PLAYER2 : PLAYER1);
    }
  }, [board, currentPlayer, winner, isDraw, checkWinner, checkDraw]);

  const resetGame = useCallback(() => {
    setBoard(Array(ROWS).fill(null).map(() => Array(COLS).fill(EMPTY)));
    setCurrentPlayer(PLAYER1);
    setWinner(null);
    setIsDraw(false);
  }, []);

  return (
    <div className="min-h-screen bg-gradient-to-br from-gray-900 to-gray-800 flex flex-col items-center justify-center p-4">
      <h1 className="text-5xl font-bold text-white mb-8">Connect 4</h1>
      
      <div className="mb-6 text-center">
        {winner ? (
          <p className="text-2xl font-semibold text-white">
            Player {winner} ({winner === PLAYER1 ? "Red" : "Yellow"}) wins!
          </p>
        ) : isDraw ? (
          <p className="text-2xl font-semibold text-white">It&apos;s a draw!</p>
        ) : (
          <p className="text-xl text-white">
            Current player: {currentPlayer === PLAYER1 ? "Red" : "Yellow"}
          </p>
        )}
      </div>

      <div className="bg-blue-600 p-4 rounded-xl shadow-2xl">
        <div className="grid grid-cols-7 gap-2">
          {board.map((row, rowIndex) => 
            row.map((cell, colIndex) => (
              <button
                key={`${rowIndex}-${colIndex}`}
                onClick={() => dropPiece(colIndex)}
                className="w-16 h-16 bg-blue-800 rounded-full relative overflow-hidden hover:bg-blue-700 transition-colors"
                disabled={winner !== null || isDraw}
              >
                <div
                  className={`absolute inset-2 rounded-full transition-all duration-300 ${
                    cell === PLAYER1
                      ? "bg-red-500 shadow-inner"
                      : cell === PLAYER2
                      ? "bg-yellow-400 shadow-inner"
                      : "bg-blue-900"
                  }`}
                />
              </button>
            ))
          )}
        </div>
      </div>

      <button
        onClick={resetGame}
        className="mt-8 px-6 py-3 bg-green-600 text-white font-semibold rounded-lg hover:bg-green-700 transition-colors shadow-lg"
      >
        New Game
      </button>

      <a
        href="/"
        className="mt-4 text-blue-300 hover:text-blue-400 transition-colors"
      >
        ← Back to Lovable UI
      </a>
    </div>
  );
}
```


## `lovable-ui/app/hello-world/page.tsx`

```tsx
export default function HelloWorld() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-purple-400 via-pink-500 to-red-500">
      <h1 className="text-6xl font-bold text-white drop-shadow-lg">
        Hello World
      </h1>
    </div>
  );
}
```


## `lovable-ui/app/api/generate/route.ts`

```ts
import { NextRequest } from "next/server";
import { query } from "@anthropic-ai/claude-code";

export async function POST(req: NextRequest) {
  try {
    const { prompt } = await req.json();
    
    if (!prompt) {
      return new Response(
        JSON.stringify({ error: "Prompt is required" }),
        { status: 400, headers: { "Content-Type": "application/json" } }
      );
    }
    
    console.log("[API] Starting code generation for prompt:", prompt);
    
    // Create a streaming response
    const encoder = new TextEncoder();
    const stream = new TransformStream();
    const writer = stream.writable.getWriter();
    
    // Start the async generation
    (async () => {
      try {
        const abortController = new AbortController();
        let messageCount = 0;
        
        for await (const message of query({
          prompt: prompt,
          abortController: abortController,
          options: {
            maxTurns: 10,
            allowedTools: [
              "Read",
              "Write",
              "Edit",
              "MultiEdit",
              "Bash",
              "LS",
              "Glob",
              "Grep",
              "WebSearch",
              "WebFetch"
            ]
          }
        })) {
          messageCount++;
          console.log(`[API] Message ${messageCount} - Type: ${message.type}`);
          
          // Log specific details based on message type
          if (message.type === 'tool_use') {
            console.log(`[API] Tool use: ${(message as any).name}`);
          } else if (message.type === 'result') {
            console.log(`[API] Result: ${(message as any).subtype}`);
          }
          
          // Send the message to the client
          await writer.write(
            encoder.encode(`data: ${JSON.stringify(message)}\n\n`)
          );
        }
        
        console.log(`[API] Generation complete. Total messages: ${messageCount}`);
        
        // Send completion signal
        await writer.write(encoder.encode("data: [DONE]\n\n"));
      } catch (error: any) {
        console.error("[API] Error during generation:", error);
        await writer.write(
          encoder.encode(`data: ${JSON.stringify({ error: error.message })}\n\n`)
        );
      } finally {
        await writer.close();
      }
    })();
    
    return new Response(stream.readable, {
      headers: {
        "Content-Type": "text/event-stream",
        "Cache-Control": "no-cache",
        "Connection": "keep-alive",
      },
    });
    
  } catch (error: any) {
    console.error("[API] Error:", error);
    return new Response(
      JSON.stringify({ error: error.message || "Internal server error" }),
      { status: 500, headers: { "Content-Type": "application/json" } }
    );
  }
}
```


## `lovable-ui/app/api/generate-daytona/route.ts`

```ts
import { NextRequest } from "next/server";
import { spawn } from "child_process";
import path from "path";

export async function POST(req: NextRequest) {
  try {
    const { prompt } = await req.json();
    
    if (!prompt) {
      return new Response(
        JSON.stringify({ error: "Prompt is required" }),
        { status: 400, headers: { "Content-Type": "application/json" } }
      );
    }
    
    if (!process.env.DAYTONA_API_KEY || !process.env.ANTHROPIC_API_KEY) {
      return new Response(
        JSON.stringify({ error: "Missing API keys" }),
        { status: 500, headers: { "Content-Type": "application/json" } }
      );
    }
    
    console.log("[API] Starting Daytona generation for prompt:", prompt);
    
    // Create a streaming response
    const encoder = new TextEncoder();
    const stream = new TransformStream();
    const writer = stream.writable.getWriter();
    
    // Start the async generation
    (async () => {
      try {
        // Use the generate-in-daytona.ts script
        const scriptPath = path.join(process.cwd(), "scripts", "generate-in-daytona.ts");
        const child = spawn("npx", ["tsx", scriptPath, prompt], {
          env: {
            ...process.env,
            DAYTONA_API_KEY: process.env.DAYTONA_API_KEY,
            ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
          },
        });
        
        let sandboxId = "";
        let previewUrl = "";
        let buffer = "";
        
        // Capture stdout
        child.stdout.on("data", async (data) => {
          buffer += data.toString();
          const lines = buffer.split('\n');
          buffer = lines.pop() || ""; // Keep incomplete line in buffer
          
          for (const line of lines) {
            if (!line.trim()) continue;
            
            // Parse Claude messages
            if (line.includes('__CLAUDE_MESSAGE__')) {
              const jsonStart = line.indexOf('__CLAUDE_MESSAGE__') + '__CLAUDE_MESSAGE__'.length;
              try {
                const message = JSON.parse(line.substring(jsonStart).trim());
                await writer.write(
                  encoder.encode(`data: ${JSON.stringify({ 
                    type: "claude_message", 
                    content: message.content 
                  })}\n\n`)
                );
              } catch (e) {
                // Ignore parse errors
              }
            }
            // Parse tool uses
            else if (line.includes('__TOOL_USE__')) {
              const jsonStart = line.indexOf('__TOOL_USE__') + '__TOOL_USE__'.length;
              try {
                const toolUse = JSON.parse(line.substring(jsonStart).trim());
                await writer.write(
                  encoder.encode(`data: ${JSON.stringify({ 
                    type: "tool_use", 
                    name: toolUse.name,
                    input: toolUse.input 
                  })}\n\n`)
                );
              } catch (e) {
                // Ignore parse errors
              }
            }
            // Parse tool results
            else if (line.includes('__TOOL_RESULT__')) {
              // Skip tool results for now to reduce noise
              continue;
            }
            // Regular progress messages
            else {
              const output = line.trim();
              
              // Filter out internal logs
              if (output && 
                  !output.includes('[Claude]:') && 
                  !output.includes('[Tool]:') &&
                  !output.includes('__')) {
                
                // Send as progress
                await writer.write(
                  encoder.encode(`data: ${JSON.stringify({ 
                    type: "progress", 
                    message: output 
                  })}\n\n`)
                );
                
                // Extract sandbox ID
                const sandboxMatch = output.match(/Sandbox created: ([a-f0-9-]+)/);
                if (sandboxMatch) {
                  sandboxId = sandboxMatch[1];
                }
                
                // Extract preview URL
                const previewMatch = output.match(/Preview URL: (https:\/\/[^\s]+)/);
                if (previewMatch) {
                  previewUrl = previewMatch[1];
                }
              }
            }
          }
        });
        
        // Capture stderr
        child.stderr.on("data", async (data) => {
          const error = data.toString();
          console.error("[Daytona Error]:", error);
          
          // Only send actual errors, not debug info
          if (error.includes("Error") || error.includes("Failed")) {
            await writer.write(
              encoder.encode(`data: ${JSON.stringify({ 
                type: "error", 
                message: error.trim() 
              })}\n\n`)
            );
          }
        });
        
        // Wait for process to complete
        await new Promise((resolve, reject) => {
          child.on("exit", (code) => {
            if (code === 0) {
              resolve(code);
            } else {
              reject(new Error(`Process exited with code ${code}`));
            }
          });
          
          child.on("error", reject);
        });
        
        // Send completion with preview URL
        if (previewUrl) {
          await writer.write(
            encoder.encode(`data: ${JSON.stringify({ 
              type: "complete", 
              sandboxId,
              previewUrl 
            })}\n\n`)
          );
          console.log(`[API] Generation complete. Preview URL: ${previewUrl}`);
        } else {
          throw new Error("Failed to get preview URL");
        }
        
        // Send done signal
        await writer.write(encoder.encode("data: [DONE]\n\n"));
      } catch (error: any) {
        console.error("[API] Error during generation:", error);
        await writer.write(
          encoder.encode(`data: ${JSON.stringify({ 
            type: "error", 
            message: error.message 
          })}\n\n`)
        );
        await writer.write(encoder.encode("data: [DONE]\n\n"));
      } finally {
        await writer.close();
      }
    })();
    
    return new Response(stream.readable, {
      headers: {
        "Content-Type": "text/event-stream",
        "Cache-Control": "no-cache",
        "Connection": "keep-alive",
      },
    });
    
  } catch (error: any) {
    console.error("[API] Error:", error);
    return new Response(
      JSON.stringify({ error: error.message || "Internal server error" }),
      { status: 500, headers: { "Content-Type": "application/json" } }
    );
  }
}
```


## `lovable-ui/components/Navbar.tsx`

```tsx
import React from "react";

export default function Navbar() {
  return (
    <nav className="absolute top-0 left-0 right-0 z-20 flex items-center justify-between px-6 py-4">
      {/* Logo & main navigation */}
      <div className="flex items-center gap-10">
        <a
          href="/"
          className="flex items-center gap-2 text-2xl font-semibold text-white hover:opacity-90 transition-opacity"
        >
          {/* Simple gradient square to mimic Lovable logo */}
          <span className="inline-block w-6 h-6 rounded-sm bg-gradient-to-br from-orange-400 via-pink-500 to-blue-500" />
          Lovable
        </a>

        <div className="hidden md:flex items-center gap-8 text-sm text-gray-300">
          <a href="#" className="hover:text-white transition-colors">
            Community
          </a>
          <a href="#" className="hover:text-white transition-colors">
            Enterprise
          </a>
          <a href="#" className="hover:text-white transition-colors">
            Learn
          </a>
          <a href="#" className="hover:text-white transition-colors">
            Shipped
          </a>
        </div>
      </div>

      {/* Auth buttons */}
      <div className="flex items-center gap-4 text-sm">
        <a
          href="#"
          className="text-gray-300 hover:text-white transition-colors"
        >
          Log in
        </a>
        <a
          href="#"
          className="px-4 py-2 bg-white text-black rounded-lg font-semibold hover:bg-gray-100 transition-colors"
        >
          Get started
        </a>
      </div>
    </nav>
  );
}
```


## `lovable-ui/components/MessageDisplay.tsx`

```tsx
import { type SDKMessage } from "@anthropic-ai/claude-code";
import { useState, useEffect } from "react";

interface MessageDisplayProps {
  messages: SDKMessage[];
}

export default function MessageDisplay({ messages }: MessageDisplayProps) {
  const [generatedPages, setGeneratedPages] = useState<string[]>([]);
  
  useEffect(() => {
    // Look for generated pages
    const pages = messages
      .filter((m: any) => 
        m.type === 'tool_use' && 
        m.name === 'Write' && 
        m.input?.file_path?.includes('/app/') &&
        (m.input?.file_path?.endsWith('.tsx') || m.input?.file_path?.endsWith('/page.tsx'))
      )
      .map((m: any) => {
        const path = m.input.file_path;
        const match = path.match(/\/app\/([^\/]+)\//);
        return match ? `/${match[1]}` : null;
      })
      .filter(Boolean);
    
    setGeneratedPages([...new Set(pages)]);
  }, [messages]);
  
  if (messages.length === 0) return null;
  
  // Filter to show only assistant messages and tool uses
  const displayMessages = messages.filter(m => 
    m.type === 'assistant' || m.type === 'tool_use' || m.type === 'result'
  );
  
  return (
    <div className="mt-8 max-w-4xl mx-auto px-4">
      <div className="bg-gray-900/50 backdrop-blur-sm rounded-2xl border border-gray-800 p-6 max-h-[600px] overflow-y-auto">
        <div className="flex items-center justify-between mb-4">
          <h3 className="text-lg font-semibold text-white">AI Assistant</h3>
          {generatedPages.length > 0 && (
            <div className="flex gap-2">
              {generatedPages.map((page, idx) => (
                <a
                  key={idx}
                  href={page}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="px-3 py-1 bg-green-600 hover:bg-green-700 text-white text-sm rounded-lg transition-colors"
                >
                  Open {page} →
                </a>
              ))}
            </div>
          )}
        </div>
        
        <div className="space-y-3">
          {displayMessages.map((message, index) => {
            // Assistant messages
            if (message.type === 'assistant' && (message as any).message?.content) {
              const content = (message as any).message.content;
              const textContent = Array.isArray(content) 
                ? content.find((c: any) => c.type === 'text')?.text 
                : content;
                
              if (!textContent) return null;
              
              return (
                <div key={index} className="animate-fadeIn">
                  <div className="text-gray-300 leading-relaxed">
                    {textContent}
                  </div>
                </div>
              );
            }
            
            // Tool uses - show as compact status
            if (message.type === 'tool_use') {
              const toolName = (message as any).name;
              const input = (message as any).input;
              
              return (
                <div key={index} className="animate-fadeIn">
                  <div className="flex items-center gap-2 text-sm text-gray-500">
                    <div className="w-2 h-2 bg-blue-500 rounded-full animate-pulse" />
                    <span className="font-mono">
                      {toolName === 'Write' && input?.file_path && 
                        `Creating ${input.file_path.split('/').pop()}`}
                      {toolName === 'Edit' && input?.file_path && 
                        `Editing ${input.file_path.split('/').pop()}`}
                      {toolName === 'Read' && input?.file_path && 
                        `Reading ${input.file_path.split('/').pop()}`}
                      {toolName === 'Bash' && input?.command && 
                        `Running: ${input.command.substring(0, 50)}...`}
                      {!['Write', 'Edit', 'Read', 'Bash'].includes(toolName) && 
                        `Using ${toolName}`}
                    </span>
                  </div>
                </div>
              );
            }
            
            // Final result
            if (message.type === 'result' && (message as any).subtype === 'success') {
              return (
                <div key={index} className="animate-fadeIn mt-4">
                  <div className="bg-green-900/20 border border-green-700 rounded-lg p-4">
                    <div className="text-green-400 font-semibold mb-2">✅ Generation Complete</div>
                    <div className="text-gray-300 text-sm">
                      {(message as any).result}
                    </div>
                    {(message as any).total_cost_usd && (
                      <div className="text-xs text-gray-500 mt-2">
                        Cost: ${(message as any).total_cost_usd.toFixed(4)}
                      </div>
                    )}
                  </div>
                </div>
              );
            }
            
            return null;
          })}
          
          {/* Show typing indicator if still generating */}
          {messages.length > 0 && !messages.some((m: any) => m.type === 'result') && (
            <div className="flex items-center gap-2 text-gray-500">
              <div className="flex gap-1">
                <div className="w-2 h-2 bg-gray-500 rounded-full animate-bounce" style={{animationDelay: '0ms'}} />
                <div className="w-2 h-2 bg-gray-500 rounded-full animate-bounce" style={{animationDelay: '150ms'}} />
                <div className="w-2 h-2 bg-gray-500 rounded-full animate-bounce" style={{animationDelay: '300ms'}} />
              </div>
              <span className="text-sm">AI is working...</span>
            </div>
          )}
        </div>
      </div>
      
      <style jsx>{`
        @keyframes fadeIn {
          from {
            opacity: 0;
            transform: translateY(10px);
          }
          to {
            opacity: 1;
            transform: translateY(0);
          }
        }
        
        .animate-fadeIn {
          animation: fadeIn 0.3s ease-out;
        }
      `}</style>
    </div>
  );
}
```


## `lovable-ui/lib/claude-code.ts`

```ts
import { query, type SDKMessage } from "@anthropic-ai/claude-code";

export interface CodeGenerationResult {
  success: boolean;
  messages: SDKMessage[];
  error?: string;
}

export async function generateCodeWithClaude(prompt: string): Promise<CodeGenerationResult> {
  try {
    const messages: SDKMessage[] = [];
    const abortController = new AbortController();
    
    // Execute the query and collect all messages
    for await (const message of query({
      prompt: prompt,
      abortController: abortController,
      options: {
        maxTurns: 10, // Allow multiple turns for complex builds
        // Grant all necessary permissions for code generation
        allowedTools: [
          "Read",
          "Write",
          "Edit",
          "MultiEdit",
          "Bash",
          "LS",
          "Glob",
          "Grep",
          "WebSearch",
          "WebFetch"
        ]
      }
    })) {
      messages.push(message);
    }
    
    return {
      success: true,
      messages: messages
    };
    
  } catch (error: any) {
    console.error("Error generating code:", error);
    return {
      success: false,
      messages: [],
      error: error.message
    };
  }
}
```


## `lovable-ui/scripts/generate-in-daytona.ts`

```ts
import { Daytona } from "@daytonaio/sdk";
import * as dotenv from "dotenv";
import * as path from "path";

// Load environment variables
dotenv.config({ path: path.join(__dirname, "../../.env") });

async function generateWebsiteInDaytona(
  sandboxIdArg?: string,
  prompt?: string
) {
  console.log("🚀 Starting website generation in Daytona sandbox...\n");

  if (!process.env.DAYTONA_API_KEY || !process.env.ANTHROPIC_API_KEY) {
    console.error("ERROR: DAYTONA_API_KEY and ANTHROPIC_API_KEY must be set");
    process.exit(1);
  }

  const daytona = new Daytona({
    apiKey: process.env.DAYTONA_API_KEY,
  });

  let sandbox;
  let sandboxId = sandboxIdArg;

  try {
    // Step 1: Create or get sandbox
    if (sandboxId) {
      console.log(`1. Using existing sandbox: ${sandboxId}`);
      // Get existing sandbox
      const sandboxes = await daytona.list();
      sandbox = sandboxes.find((s: any) => s.id === sandboxId);
      if (!sandbox) {
        throw new Error(`Sandbox ${sandboxId} not found`);
      }
      console.log(`✓ Connected to sandbox: ${sandbox.id}`);
    } else {
      console.log("1. Creating new Daytona sandbox...");
      sandbox = await daytona.create({
        public: true,
        image: "node:20",
      });
      sandboxId = sandbox.id;
      console.log(`✓ Sandbox created: ${sandboxId}`);
    }

    // Get the root directory
    const rootDir = await sandbox.getUserRootDir();
    console.log(`✓ Working directory: ${rootDir}`);

    // Step 2: Create project directory
    console.log("\n2. Setting up project directory...");
    const projectDir = `${rootDir}/website-project`;
    await sandbox.process.executeCommand(`mkdir -p ${projectDir}`, rootDir);
    console.log(`✓ Created project directory: ${projectDir}`);

    // Step 3: Initialize npm project
    console.log("\n3. Initializing npm project...");
    await sandbox.process.executeCommand("npm init -y", projectDir);
    console.log("✓ Package.json created");

    // Step 4: Install Claude Code SDK locally in project
    console.log("\n4. Installing Claude Code SDK locally...");
    const installResult = await sandbox.process.executeCommand(
      "npm install @anthropic-ai/claude-code@latest",
      projectDir,
      undefined,
      180000 // 3 minute timeout
    );

    if (installResult.exitCode !== 0) {
      console.error("Installation failed:", installResult.result);
      throw new Error("Failed to install Claude Code SDK");
    }
    console.log("✓ Claude Code SDK installed");

    // Verify installation
    console.log("\n5. Verifying installation...");
    const checkInstall = await sandbox.process.executeCommand(
      "ls -la node_modules/@anthropic-ai/claude-code",
      projectDir
    );
    console.log("Installation check:", checkInstall.result);

    // Step 6: Create the generation script file
    console.log("\n6. Creating generation script file...");

    const generationScript = `const { query } = require('@anthropic-ai/claude-code');
const fs = require('fs');

async function generateWebsite() {
  const prompt = \`${
    prompt ||
    "Create a modern blog website with markdown support and a dark theme"
  }
  
  Important requirements:
  - Create a NextJS app with TypeScript and Tailwind CSS
  - Use the app directory structure
  - Create all files in the current directory
  - Include a package.json with all necessary dependencies
  - Make the design modern and responsive
  - Add at least a home page and one other page
  - Include proper navigation between pages
  \`;

  console.log('Starting website generation with Claude Code...');
  console.log('Working directory:', process.cwd());
  
  const messages = [];
  const abortController = new AbortController();
  
  try {
    for await (const message of query({
      prompt: prompt,
      abortController: abortController,
      options: {
        maxTurns: 20,
        allowedTools: [
          'Read',
          'Write',
          'Edit',
          'MultiEdit',
          'Bash',
          'LS',
          'Glob',
          'Grep'
        ]
      }
    })) {
      messages.push(message);
      
      // Log progress
      if (message.type === 'text') {
        console.log('[Claude]:', (message.text || '').substring(0, 80) + '...');
        console.log('__CLAUDE_MESSAGE__', JSON.stringify({ type: 'assistant', content: message.text }));
      } else if (message.type === 'tool_use') {
        console.log('[Tool]:', message.name, message.input?.file_path || '');
        console.log('__TOOL_USE__', JSON.stringify({ 
          type: 'tool_use', 
          name: message.name, 
          input: message.input 
        }));
      } else if (message.type === 'result') {
        console.log('__TOOL_RESULT__', JSON.stringify({ 
          type: 'tool_result', 
          result: message.result 
        }));
      }
    }
    
    console.log('\\nGeneration complete!');
    console.log('Total messages:', messages.length);
    
    // Save generation log
    fs.writeFileSync('generation-log.json', JSON.stringify(messages, null, 2));
    
    // List generated files
    const files = fs.readdirSync('.').filter(f => !f.startsWith('.'));
    console.log('\\nGenerated files:', files.join(', '));
    
  } catch (error) {
    console.error('Generation error:', error);
    console.error('Stack:', error.stack);
    process.exit(1);
  }
}

generateWebsite().catch(console.error);`;

    // Write the script to a file
    await sandbox.process.executeCommand(
      `cat > generate.js << 'SCRIPT_EOF'
${generationScript}
SCRIPT_EOF`,
      projectDir
    );
    console.log("✓ Generation script written to generate.js");

    // Verify the script was created
    const checkScript = await sandbox.process.executeCommand(
      "ls -la generate.js && head -5 generate.js",
      projectDir
    );
    console.log("Script verification:", checkScript.result);

    // Step 7: Run the generation script
    console.log("\n7. Running Claude Code generation...");
    console.log(`Prompt: "${prompt || "Create a modern blog website"}"`);
    console.log("\nThis may take several minutes...\n");

    const genResult = await sandbox.process.executeCommand(
      "node generate.js",
      projectDir,
      {
        ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
        NODE_PATH: `${projectDir}/node_modules`,
      },
      600000 // 10 minute timeout
    );

    console.log("\nGeneration output:");
    console.log(genResult.result);

    if (genResult.exitCode !== 0) {
      throw new Error("Generation failed");
    }

    // Step 8: Check generated files
    console.log("\n8. Checking generated files...");
    const filesResult = await sandbox.process.executeCommand(
      "ls -la",
      projectDir
    );
    console.log(filesResult.result);

    // Step 9: Install dependencies if package.json was updated
    const hasNextJS = await sandbox.process.executeCommand(
      "test -f package.json && grep -q next package.json && echo yes || echo no",
      projectDir
    );

    if (hasNextJS.result?.trim() === "yes") {
      console.log("\n9. Installing project dependencies...");
      const npmInstall = await sandbox.process.executeCommand(
        "npm install",
        projectDir,
        undefined,
        300000 // 5 minute timeout
      );

      if (npmInstall.exitCode !== 0) {
        console.log("Warning: npm install had issues:", npmInstall.result);
      } else {
        console.log("✓ Dependencies installed");
      }

      // Step 10: Start dev server in background
      console.log("\n10. Starting development server in background...");

      // Start the server in background using nohup
      await sandbox.process.executeCommand(
        `nohup npm run dev > dev-server.log 2>&1 &`,
        projectDir,
        { PORT: "3000" }
      );

      console.log("✓ Server started in background");

      // Wait a bit for server to initialize
      console.log("Waiting for server to start...");
      await new Promise((resolve) => setTimeout(resolve, 8000));

      // Check if server is running
      const checkServer = await sandbox.process.executeCommand(
        "curl -s -o /dev/null -w '%{http_code}' http://localhost:3000 || echo 'failed'",
        projectDir
      );

      if (checkServer.result?.trim() === '200') {
        console.log("✓ Server is running!");
      } else {
        console.log("⚠️  Server might still be starting...");
        console.log("You can check logs with: cat dev-server.log");
      }
    }

    // Step 11: Get preview URL
    console.log("\n11. Getting preview URL...");
    const preview = await sandbox.getPreviewLink(3000);

    console.log("\n✨ SUCCESS! Website generated!");
    console.log("\n📊 SUMMARY:");
    console.log("===========");
    console.log(`Sandbox ID: ${sandboxId}`);
    console.log(`Project Directory: ${projectDir}`);
    console.log(`Preview URL: ${preview.url}`);
    if (preview.token) {
      console.log(`Access Token: ${preview.token}`);
    }

    console.log("\n🌐 VISIT YOUR WEBSITE:");
    console.log(preview.url);

    console.log("\n💡 TIPS:");
    console.log("- The sandbox will stay active for debugging");
    console.log("- Server logs: SSH in and run 'cat website-project/dev-server.log'");
    console.log(
      `- To get preview URL again: npx tsx scripts/get-preview-url.ts ${sandboxId}`
    );
    console.log(
      `- To reuse this sandbox: npx tsx scripts/generate-in-daytona.ts ${sandboxId}`
    );
    console.log(`- To remove: npx tsx scripts/remove-sandbox.ts ${sandboxId}`);

    return {
      success: true,
      sandboxId: sandboxId,
      projectDir: projectDir,
      previewUrl: preview.url,
    };
  } catch (error: any) {
    console.error("\n❌ ERROR:", error.message);

    if (sandbox) {
      console.log(`\nSandbox ID: ${sandboxId}`);
      console.log("The sandbox is still running for debugging.");

      // Try to get debug info
      try {
        const debugInfo = await sandbox.process.executeCommand(
          "pwd && echo '---' && ls -la && echo '---' && test -f generate.js && cat generate.js | head -20 || echo 'No script'",
          `${await sandbox.getUserRootDir()}/website-project`
        );
        console.log("\nDebug info:");
        console.log(debugInfo.result);
      } catch (e) {
        // Ignore
      }
    }

    throw error;
  }
}

// Main execution
async function main() {
  const args = process.argv.slice(2);
  let sandboxId: string | undefined;
  let prompt: string | undefined;

  // Parse arguments
  if (args.length > 0) {
    // Check if first arg is a sandbox ID (UUID format)
    const uuidRegex =
      /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
    if (uuidRegex.test(args[0])) {
      sandboxId = args[0];
      prompt = args.slice(1).join(" ");
    } else {
      prompt = args.join(" ");
    }
  }

  if (!prompt) {
    prompt =
      "Create a modern blog website with markdown support and a dark theme. Include a home page, blog listing page, and individual blog post pages.";
  }

  console.log("📝 Configuration:");
  console.log(
    `- Sandbox: ${sandboxId ? `Using existing ${sandboxId}` : "Creating new"}`
  );
  console.log(`- Prompt: ${prompt}`);
  console.log();

  try {
    await generateWebsiteInDaytona(sandboxId, prompt);
  } catch (error) {
    console.error("Failed to generate website:", error);
    process.exit(1);
  }
}

// Handle graceful shutdown
process.on("SIGINT", () => {
  console.log("\n\n👋 Exiting... The sandbox will continue running.");
  process.exit(0);
});

main();
```


## `lovable-ui/scripts/get-preview-url.ts`

```ts
import { Daytona } from "@daytonaio/sdk";
import * as dotenv from "dotenv";
import * as path from "path";

// Load environment variables
dotenv.config({ path: path.join(__dirname, "../../.env") });

async function getPreviewUrl(sandboxId: string, port: number = 3000) {
  if (!process.env.DAYTONA_API_KEY) {
    console.error("ERROR: DAYTONA_API_KEY must be set");
    process.exit(1);
  }

  const daytona = new Daytona({
    apiKey: process.env.DAYTONA_API_KEY,
  });

  try {
    // Get sandbox
    const sandboxes = await daytona.list();
    const sandbox = sandboxes.find((s: any) => s.id === sandboxId);
    
    if (!sandbox) {
      throw new Error(`Sandbox ${sandboxId} not found`);
    }

    console.log(`✓ Found sandbox: ${sandboxId}`);

    // Get preview URL
    const preview = await sandbox.getPreviewLink(port);
    
    console.log("\n🌐 Preview URL:");
    console.log(preview.url);
    
    if (preview.token) {
      console.log(`\n🔑 Access Token: ${preview.token}`);
    }
    
    return preview.url;
  } catch (error: any) {
    console.error("Failed to get preview URL:", error.message);
    process.exit(1);
  }
}

// Main execution
async function main() {
  const sandboxId = process.argv[2];
  const port = process.argv[3] ? parseInt(process.argv[3]) : 3000;
  
  if (!sandboxId) {
    console.error("Usage: npx tsx scripts/get-preview-url.ts <sandbox-id> [port]");
    console.error("Example: npx tsx scripts/get-preview-url.ts 7a517a82-942c-486b-8a62-6357773eb3ea 3000");
    process.exit(1);
  }

  await getPreviewUrl(sandboxId, port);
}

main();
```


## `lovable-ui/scripts/start-dev-server.ts`

```ts
import { Daytona } from "@daytonaio/sdk";
import * as dotenv from "dotenv";
import * as path from "path";

// Load environment variables
dotenv.config({ path: path.join(__dirname, "../../.env") });

async function startDevServer(sandboxId: string, projectPath: string = "website-project") {
  if (!process.env.DAYTONA_API_KEY) {
    console.error("ERROR: DAYTONA_API_KEY must be set");
    process.exit(1);
  }

  const daytona = new Daytona({
    apiKey: process.env.DAYTONA_API_KEY,
  });

  try {
    // Get sandbox
    const sandboxes = await daytona.list();
    const sandbox = sandboxes.find((s: any) => s.id === sandboxId);
    
    if (!sandbox) {
      throw new Error(`Sandbox ${sandboxId} not found`);
    }

    console.log(`✓ Found sandbox: ${sandboxId}`);
    
    const rootDir = await sandbox.getUserRootDir();
    const projectDir = `${rootDir}/${projectPath}`;
    
    // Check if project exists
    const checkProject = await sandbox.process.executeCommand(
      `test -d ${projectPath} && echo "exists" || echo "not found"`,
      rootDir
    );
    
    if (checkProject.result?.trim() !== "exists") {
      throw new Error(`Project directory ${projectPath} not found in sandbox`);
    }
    
    // Kill any existing dev server
    console.log("Stopping any existing dev server...");
    await sandbox.process.executeCommand(
      "pkill -f 'npm run dev' || true",
      projectDir
    );
    
    // Start dev server in background
    console.log("Starting development server...");
    await sandbox.process.executeCommand(
      `nohup npm run dev > dev-server.log 2>&1 &`,
      projectDir,
      { PORT: "3000" }
    );
    
    console.log("✓ Server started in background");
    
    // Wait for server to start
    console.log("Waiting for server to initialize...");
    await new Promise((resolve) => setTimeout(resolve, 8000));
    
    // Check if server is running
    const checkServer = await sandbox.process.executeCommand(
      "curl -s -o /dev/null -w '%{http_code}' http://localhost:3000 || echo 'failed'",
      projectDir
    );
    
    if (checkServer.result?.trim() === '200') {
      console.log("✓ Server is running!");
      
      // Get preview URL
      const preview = await sandbox.getPreviewLink(3000);
      console.log("\n🌐 Preview URL:");
      console.log(preview.url);
      
      if (preview.token) {
        console.log(`\n🔑 Access Token: ${preview.token}`);
      }
    } else {
      console.log("⚠️  Server might still be starting...");
      console.log("Check logs by SSHing into the sandbox and running:");
      console.log(`cat ${projectPath}/dev-server.log`);
    }
    
  } catch (error: any) {
    console.error("Failed to start dev server:", error.message);
    process.exit(1);
  }
}

// Main execution
async function main() {
  const sandboxId = process.argv[2];
  const projectPath = process.argv[3] || "website-project";
  
  if (!sandboxId) {
    console.error("Usage: npx tsx scripts/start-dev-server.ts <sandbox-id> [project-path]");
    console.error("Example: npx tsx scripts/start-dev-server.ts 7a517a82-942c-486b-8a62-6357773eb3ea");
    process.exit(1);
  }

  await startDevServer(sandboxId, projectPath);
}

main();
```


## `lovable-ui/scripts/test-preview-url.ts`

```ts
import { Daytona } from "@daytonaio/sdk";
import * as dotenv from "dotenv";
import * as path from "path";

// Load environment variables
dotenv.config({ path: path.join(__dirname, "../../.env") });

async function testPreviewUrl() {
  console.log("Testing Daytona Preview URL functionality...\n");

  if (!process.env.DAYTONA_API_KEY) {
    console.error("ERROR: DAYTONA_API_KEY is not set");
    process.exit(1);
  }

  const daytona = new Daytona({
    apiKey: process.env.DAYTONA_API_KEY,
  });

  try {
    console.log("1. Creating sandbox...");
    const sandbox = await daytona.create({
      public: true, // Make it publicly accessible
    });
    console.log(`✓ Sandbox created: ${sandbox.id}`);

    console.log("\n2. Creating NextJS app...");
    await sandbox.process.executeCommand(
      "npx create-next-app@latest my-app --typescript --tailwind --app --no-git --yes"
    );
    console.log("✓ NextJS app created");

    console.log("\n3. Creating custom page...");
    const customPage = `export default function PreviewTest() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24 bg-gradient-to-b from-purple-900 to-black">
      <h1 className="text-6xl font-bold text-white mb-4">🎉 Preview Works!</h1>
      <p className="text-2xl text-gray-300">This page is served from Daytona sandbox.</p>
      <div className="mt-8 p-4 bg-white/10 rounded-lg">
        <p className="text-lg text-gray-200">Sandbox ID:</p>
        <code className="text-sm text-green-400">${sandbox.id}</code>
      </div>
    </main>
  );
}`;

    await sandbox.process.executeCommand(`
      cd my-app && 
      mkdir -p app/preview && 
      echo '${customPage.replace(/'/g, "'\"'\"'")}' > app/preview/page.tsx
    `);
    console.log("✓ Custom page created at /preview");

    console.log("\n4. Installing dependencies...");
    await sandbox.process.executeCommand("cd my-app && npm install");
    console.log("✓ Dependencies installed");

    console.log("\n5. Starting dev server...");
    // Start server in background
    await sandbox.process.executeCommand(
      "cd my-app && nohup npm run dev > /tmp/server.log 2>&1 &"
    );
    
    // Give server time to start
    console.log("⏳ Waiting for server to start...");
    await new Promise(resolve => setTimeout(resolve, 10000));

    // Verify server is running
    const checkServer = await sandbox.process.executeCommand("curl -s -o /dev/null -w '%{http_code}' http://localhost:3000");
    console.log(`✓ Server status: ${checkServer.result}`);

    console.log("\n6. Getting preview URL using getPreviewLink()...");
    
    // THIS IS THE KEY PART - Using getPreviewLink()
    const previewInfo = await sandbox.getPreviewLink(3000);
    
    console.log("\n🎯 PREVIEW INFORMATION:");
    console.log("=======================");
    console.log(`Preview URL: ${previewInfo.url}`);
    console.log(`Access Token: ${previewInfo.token || 'No token (public sandbox)'}`);
    
    console.log("\n📍 PAGES TO VISIT:");
    console.log(`- ${previewInfo.url} (default Next.js page)`);
    console.log(`- ${previewInfo.url}/preview (custom page we created)`);
    
    if (previewInfo.token) {
      console.log("\n🔐 For programmatic access with curl:");
      console.log(`curl -H "x-daytona-preview-token: ${previewInfo.token}" ${previewInfo.url}`);
    }
    
    console.log("\n✅ SUCCESS! The preview URL should now work.");
    console.log(`\nSandbox ID: ${sandbox.id}`);
    console.log("This sandbox will stay alive for testing.");
    console.log(`\nTo remove it later, run:`);
    console.log(`npx tsx scripts/remove-sandbox.ts ${sandbox.id}`);
    
    // Keep checking server status
    console.log("\n📊 Monitoring server (press Ctrl+C to exit)...");
    setInterval(async () => {
      try {
        const status = await sandbox.process.executeCommand("ps aux | grep 'next dev' | grep -v grep | wc -l");
        const timestamp = new Date().toLocaleTimeString();
        process.stdout.write(`\r[${timestamp}] Server processes: ${status.result?.trim()}`);
      } catch (e) {
        // Ignore errors
      }
    }, 5000);

  } catch (error: any) {
    console.error("\n❌ ERROR:", error.message);
    console.error("Full error:", error);
    process.exit(1);
  }
}

// Handle graceful shutdown
process.on('SIGINT', () => {
  console.log('\n\n👋 Exiting... The sandbox will continue running.');
  process.exit(0);
});

testPreviewUrl().catch(console.error);
```


## `lovable-ui/scripts/remove-sandbox.ts`

```ts
import { Daytona } from "@daytonaio/sdk";
import * as dotenv from "dotenv";
import * as path from "path";

// Load environment variables
dotenv.config({ path: path.join(__dirname, "../../.env") });

async function removeSandbox(sandboxId: string) {
  if (!process.env.DAYTONA_API_KEY) {
    console.error("ERROR: DAYTONA_API_KEY must be set");
    process.exit(1);
  }

  const daytona = new Daytona({
    apiKey: process.env.DAYTONA_API_KEY,
  });

  try {
    console.log(`Removing sandbox: ${sandboxId}...`);
    await daytona.remove(sandboxId);
    console.log("✓ Sandbox removed successfully");
  } catch (error: any) {
    console.error("Failed to remove sandbox:", error.message);
    process.exit(1);
  }
}

// Main execution
async function main() {
  const sandboxId = process.argv[2];
  
  if (!sandboxId) {
    console.error("Usage: npx tsx scripts/remove-sandbox.ts <sandbox-id>");
    process.exit(1);
  }

  await removeSandbox(sandboxId);
}

main();
```

