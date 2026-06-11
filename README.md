# n8n-telegram-ai-hydration-reminder 💧
Automated hydration reminder workflow for n8n. Stay hydrated with a little help from AI!

## Requirements

*   n8n Version 2.23.2 or later
*   Ollama API (running and accessible)
*   Telegram API (running, accessible and paired Chat!) Bot Token & Chat ID (see setup below)

## Setup

This workflow uses a `.env` file to store sensitive information like API keys and tokens. This keeps your workflow clean and secure!

1.  **Configure Environmental Constants:** Create a `.env` file in the same directory as your `docker-compose.yml` file. Here's an example:

    ```
    OLLAMA_API_URI=http://your_hostname:11434
    TELEGRAM_BOT_TOKEN="your_bot_token"
    TELEGRAM_CHAT_ID="your_chat_id"
    # Add other constants here as needed
    ```

2.  **Docker Compose:** Modify your `docker-compose.yml` file to load the `.env` file:

    ```yaml
    environment:
      N8N_BLOCK_ENV_ACCESS_IN_NODE=false
      env_file:
        - .env
    ```

3.  **Download Workflows & Import in n8n:** 

    files: Telegram_AI_Hydration_Reminder_CONFIG.json Telegram_AI_Hydration_Reminder.json
	
5.  **Configure Constants:** Modify the "Set: Constants" node in the "Telegram AI Hydration Reminder CONFIG" sub-workflow:

    ```
    language: 'en', // telegram chat language, currently supported: 'en', 'de'
    ollamaModel: "gemma3:27b",     // your model ID
    weatherLocation: "Dortmund",     // your location
    temperatureHot: 26,   // temperature limit until triggering a warning (ai generated)
    dailyGoal: 3000,	// your daily goal in ml
    ```
6. **Start:**

   Publish "Telegram AI Hydration Reminder CONFIG" then "Telegram AI Hydration Reminder"
   
## Workflow Overview

This n8n workflow consists of four main parts:

*   **Startup Workflow:** Initializes the workflow, loading configuration and resetting variables.
*   **Reminder Workflow:** Generates and sends hydration reminders via Telegram.
*   **Telegram Input Handler Workflow:** Processes user input (button clicks & text) from Telegram and updates water intake.
*   **Aggressive Workflow:** Handles missed reminders and encourages hydration with increasingly insistent messages.

## Startup Workflow - Getting Things Ready

This workflow kicks everything off. Key features:

*   **Loads Configuration:** Reads constants from the `.env` file.
*   **"DrinkWater CONFIG" Sub-Workflow:** Centralizes all main configuration, timings, and flags. This makes updating settings much easier!
*   **Resets Variables:** Initializes workflow variables to zero.

**Note:** n8n can be a bit quirky with stateful data. The 'Set: Init Workflow Variables' node helps manage n8n's state and ensures reliable operation, especially during development and when switching environments.

## Reminder Workflow - Sending Hydration Alerts

This is where the magic happens!

*   **AI-Powered Messages:** Uses the Ollama API to generate personalized "Hydration & Why" reminders. It includes the current weather according to the temperature.
*   **Telegram Delivery:** Sends reminders via Telegram using your configured bot and chat ID.
*   **Interactive Buttons:** Includes buttons in the message, allowing you to easily log your water intake.

### Schedule Configuration

*   **Production:** Hourly, between 8:00 and 22:00 (Cron expression: `0 8-22 * * *`)
*   **Testing:** For quick testing in production, set the trigger interval to 1 minute.

## Telegram Input Handler Workflow - Processing Your Input

This sub-workflow runs every 15 seconds to:

*   **Handle Button Clicks:** Processes responses from the buttons in the reminder messages.
*   **Calculate Intake:** Totals your daily water intake.
*   **Analyze Text Inputs:** Processes any text you send to the bot (for future features).

## Aggressive Workflow - Gentle (and not so gentle) Nudges

This workflow is designed to *gently* encourage hydration if you miss responding to reminders.

*   **Missed Callback Detection:** Tracks how often you ignore the reminder callbacks.
*   **Escalating Messages:** If you miss 4 callbacks, the workflow starts sending increasingly insistent (and slightly humorous) messages. Each message's prompt incorporates previous messages to create a conversational history.
*   **Reset Mechanism:** Any response to a callback or a valid water intake amount will reset the aggressive mode.

**Important Considerations:**

*   **n8n Quirks:** We've all been there! n8n can sometimes have issues with data persistence and caching. This workflow includes some workarounds to help minimize those problems.
*   **Sub-Workflows:** Using sub-workflows makes the overall workflow more organized and easier to maintain.
*   **Configuration:** Keep your configuration centralized in the `"DrinkWater CONFIG"` sub-workflow.

Happy hydrating! 💧

PS:  This was my first n8n project, and what started as a simple problem turned into a much bigger learning experience than I expected. 
