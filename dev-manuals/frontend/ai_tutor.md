# AI Tutor Documentation

This documentation provides an overview for developers working with the AI Tutor widget, which consists of the components `TutorAvatar.tsx`, `TutorChat.tsx`, and `TutorWidget.tsx`. It describes component responsibilities, API usage, and key areas for customization.

---

## Table of Contents

- [Project Structure](#project-structure)
- [Component Overview](#component-overview)
  - [TutorAvatar](#tutoravatar)
  - [TutorChat](#tutorchatsx)
  - [TutorWidget](#tutorwidgettsx)
- [API Integration](#api-integration)
  - [1. Chat Backend (GraphQL)](#1-chat-backend-graphql)
  - [2. Recommendations Backend (GraphQL)](#2-recommendations-backend-graphql)
- [Customization & Extension Points](#customization--extension-points)
- [Styling](#styling)
- [Internationalization](#internationalization)
- [Known Issues / TODOs](#known-issues--todos)

---

## Project Structure

```
components/tutor/
  TutorAvatar.tsx      // Dino avatar, floating action button
  TutorChat.tsx        // Chat window component, handles chat logic
  TutorWidget.tsx      // Main widget, manages avatar, chat, recommendations, welcome
```

---

## Component Overview

### TutorAvatar.tsx

- **Purpose:** Displays the dino avatar used as the floating action button to open/close the chat window.
- **Usage:** Pure presentational component, no external dependencies.

### TutorChat.tsx

- **Purpose:** Handles chat UI, user input, and communication with the LLM backend.
- **Key Logic:**
  - Manages chat history and loading state.
  - Sends user queries to the backend via a GraphQL mutation.
  - Handles Enter/Shift+Enter for sending or newlines.
  - Displays chat messages and a placeholder welcome when empty.
- **API Integration:** 
  - Uses `llmRequest` GraphQL mutation (see [API Integration](#api-integration)).

### TutorWidget.tsx

- **Purpose:** Main container for the widget, handles:
  - Avatar display and drag behavior.
  - Showing/hiding the chat window.
  - Displaying and controlling the recommendation bubble and welcome message.
  - Fetching recommendations from the backend.
- **API Integration:**
  - Uses `recommendations` GraphQL query (see [API Integration](#api-integration)).

---

## API Integration

### 1. Chat Backend (GraphQL)

- **Location:** `TutorWdiget.tsx`
- **Endpoint:** tbd
- **Mutation Example:**

tbd
- **Expected Response:**
tbd


---

## Customization & Extension Points

- **API Endpoints:**  
  - Change GraphQL endpoints in `fetch` calls if needed.
  - Adapt queries/mutations to match your backend schema.

- **Welcome Message:**  
  - Edit the string in `TutorWidget.tsx` inside the welcome bubble for custom greetings.

- **Recommendation Bubble:**
  - Style/position in `TutorWidget.tsx`.
  - Bubble auto-hides after 8 seconds or immediately if the user clicks "×".

- **Chat Placeholder:**  
  - Change the placeholder string in `TutorChat.tsx` (`BOT_PLACEHOLDER`).

- **Chat UI:**  
  - Style the chat bubbles, button, and input in `TutorChat.tsx`.

- **Avatar:**  
  - Replace the Dino SVG or image in `TutorAvatar.tsx` for custom branding.

- **Authentication:**  
  - If APIs are protected, ensure authentication tokens/headers are set in fetch calls.

---

## Styling

- All styles are currently implemented as inline `style` objects in each component for ease of integration.
- For larger projects, consider extracting styles to CSS/SASS modules or using a CSS-in-JS solution.

---

## Internationalization

- All user-facing strings are hard-coded in German/English in the respective components.
- For multi-language support, extract strings and use a localization library (e.g., `react-i18next`).

---

## Known Issues / TODOs

- **API Error Handling:**  
  - Minimal error feedback is given to the user; consider improving error UI/UX.
- **Accessibility:**  
  - The widget and chat input are semi-accessible, but further ARIA improvements may be required.
- **Mobile Support:**  
  - Widget is designed for desktop; for touch/mobile, drag & drop and sizing may need adjustment.
- **Testing:**  
  - No unit or integration tests are included; add tests as needed for production environments.

---


---

For any issues or further integration questions, see the code comments in each file or contact the maintainers.
