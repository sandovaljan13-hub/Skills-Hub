# Master Orchestrator Skill

## Description
Coordinates multi-step workflows using Vercel, Hugging Face, and Anthropic skills to build full-stack AI applications.

## Instructions
When a user asks to build or improve an AI feature, follow this coordination logic:

1. **Frontend & UI:** Refer to the `vercel-skills` directory. Prioritize the "Agent Skills" for design system consistency and Vercel AI SDK implementation.
2. **AI Logic & Models:** - Use `anthropic-skills` for prompt engineering and complex reasoning tasks.
   - Use `hf-skills` when the user needs specific open-source models or specialized "transformers" logic.
3. **Integration:** - Use Vercel's `useChat` or `useCompletion` hooks to connect the UI to the Anthropic/HF backends.
   - Ensure all API keys are handled via environment variables (e.g., `.env.local`).

## Best Practices
- Always suggest the "Vercel way" for deployment and edge functions.
- Always suggest the "Anthropic way" for structured JSON outputs.