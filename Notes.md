
# Interactive Terminal Portfolio: Application Notes

This document provides an overview of how this Next.js and TypeScript application is built, detailing its architecture, key components, and core functionalities.

## 1. Tech Stack

- **Framework**: [Next.js](https://nextjs.org/) (using App Router)
- **Language**: [TypeScript](https://www.typescriptlang.org/)
- **UI Library**: [React](https://reactjs.org/)
- **Styling**: [Tailwind CSS](https://tailwindcss.com/) with [shadcn/ui](https://ui.shadcn.com/) components.
- **AI/Generative AI**: [Google's Genkit](https://firebase.google.com/docs/genkit) for the AI Assistant.
- **State Management**: React Hooks (`useState`, `useEffect`, `useContext`).

## 2. Project Structure

The project follows a standard Next.js App Router structure.

-   `src/app/`: Contains the main pages and layouts.
    -   `layout.tsx`: The root layout for the entire application. It includes the `<body>` and `<html>` tags, font imports, and the main `Toaster` component for notifications.
    -   `page.tsx`: The main entry point of the application. It manages the state for switching between the Terminal and Assistant views.
    -   `globals.css`: Global styles and Tailwind CSS layers. It also defines the variables for the `dark` and `light` themes.
    -   `api/chat/route.ts`: A Next.js API route that handles requests from the chat frontend, processes them with Genkit, and streams back the AI's response.
-   `src/components/`: Reusable React components.
    -   `ui/`: Contains the base UI components from `shadcn/ui` (e.g., `Button`, `Card`, `Textarea`).
    -   `terminal/`: Components specific to the terminal interface.
        -   `index.tsx`: The main `Terminal` component, which includes the header, history display, and input line.
        -   `command-handler.tsx`: A crucial file that contains the logic to parse user commands and return the appropriate React component as output.
        -   `outputs.tsx`: Defines the various React components that are rendered as output for different commands (e.g., `HelpMessage`, `ProjectsOutput`).
    -   `chat-panel.tsx`: The component for the AI Assistant interface, managing the chat history and user input.
-   `src/ai/`: All Genkit-related code for the AI assistant.
    -   `genkit.ts`: Initializes and configures the main Genkit `ai` object.
    -   `flows/`: Contains the Genkit flows.
        -   `chatbot.ts`: The main flow that orchestrates the chat logic. It receives user messages, consults the developer's profile data, and uses specific tools to answer questions about skills, projects, and hiring suitability.
        -   Other `chatbot-*.ts` files define specialized tools that the main `chatbot` flow can use.
-   `src/lib/`: Shared utilities, type definitions, and data.
    -   `data.ts`: A mock database. It stores all the portfolio content, such as the profile summary, skills, projects, and experience, as exported constants.
    -   `types.ts`: TypeScript interfaces for the data structures used throughout the app (e.g., `Profile`, `Project`).
    -   `utils.ts`: Utility functions, most notably the `cn` function for merging Tailwind CSS classes.
-   `src/hooks/`: Custom React hooks for shared logic.
    -   `use-command-history.ts`: Manages the terminal's command history, allowing navigation with arrow keys.
    -   `use-toast.ts`: Hook for triggering toast notifications.

## 3. Core Functionality Breakdown

### Terminal Interface

The terminal is the heart of the portfolio.

1.  **State Management**: The main state (`history`, `theme`, `showChat`) is managed in `src/app/page.tsx`.
2.  **Input Handling**: The `Terminal` component (`src/components/terminal/index.tsx`) captures user input. When the user presses `Enter`, it calls `processCommand`.
3.  **Command Processing**: `handleCommand` in `src/components/terminal/command-handler.tsx` uses a `switch` statement to match the input command. Based on the command, it returns a specific React component (from `outputs.tsx`) to be rendered.
4.  **History**: The `history` state is an array of objects, where each object contains the user's `input` and the corresponding `output` component. This array is mapped over to render the terminal's content.
5.  **Autofocus**: The `useEffect` hook in `src/components/terminal/index.tsx` is used to automatically focus the input field when the component mounts, improving interactivity. A random IP address is also generated on mount to give the prompt a realistic feel.

### AI Assistant (Chat Panel)

1.  **Frontend**: The `ChatPanel` component (`src/components/chat-panel.tsx`) manages the conversation's state (`messages`, `input`, `isLoading`).
2.  **API Request**: When a user sends a message, `handleSend` makes a `POST` request to `/api/chat/route.ts`. It sends the current message and the chat history.
3.  **Streaming Response**: The API route returns a `Response` object with a streaming body. The frontend uses the `ReadableStream` API (`res.body.getReader()`) to read the AI's response chunk-by-chunk. This creates the "typing" effect.
4.  **Backend (Genkit)**:
    -   The `/api/chat` route calls the `portfolioChat` flow in `src/ai/flows/chatbot.ts`.
    -   This flow is designed to be a streaming flow (`ai.generateStream`).
    -   It is augmented with context from `src/lib/data.ts` (profile info, skills, projects) and equipped with **Tools** (other Genkit flows like `chatbotWhyHire` and `getProjectExplanation`).
    -   Genkit's LLM decides whether to answer directly or use one of the provided tools based on the user's query. This allows for more structured and accurate answers to specific questions.

### Theming

The application supports both `dark` and `light` themes.

-   **CSS Variables**: The themes are defined in `src/app/globals.css` using HSL CSS variables under the `:root` (for dark mode) and `.light` selectors.
-   **State & DOM Manipulation**: The `theme` state is managed in `page.tsx`. A `useEffect` hook listens for changes to this state and updates the `<html>` element's class (`dark` or `light`), which causes the CSS variables to switch.

This setup provides a solid foundation for a modern, interactive, and easily maintainable portfolio application.
