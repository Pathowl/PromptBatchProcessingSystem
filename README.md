# 🚀 Prompt Processing System

An asynchronous application designed for scalable user prompt processing.

## 🛠️ Tech Stack

- **Frontend:** React, Vite, Axios
- **Backend:** C#, .NET 10.0.8, Entity Framework Core
- **Messaging:** RabbitMQ, MassTransit
- **Testing:** xUnit, Moq, In-Memory DB

## 🧠 The Architecture

1. **Frontend Submission:** The React interface batches user prompts and sends them simultaneously to the API.
2. **API & Database:** A .NET controller receives the DTO, saves it to the database with a Pending status, and utilizes MassTransit to publish a PromptCreated event to the message queue. The API immediately returns a 200 OK status to the frontend.
3. **RabbitMQ:** The prompt event is queued in RabbitMQ, waiting for a worker to pick it up. The worker is specifically configured to process one task at a time to ensure stability.
4. **Background Worker:** PromptWorker retrieves the task from RabbitMQ, updates the database status to Processing, and simulates a 5-second delay (to mimic AI generation time) before executing the AI integration logic.
5. **Wyniki i bledy:** Once the AI service (or mock) generates a response, the worker saves the result to the database and updates the status to Completed (or Failed if an error occurred).
6. **Frontend Polling:** The React application polls the API (GET /prompts) every second to fetch the latest data. This guarantees a fluid, real-time UI experience where statuses update dynamically (Pending ➔ Processing ➔ Completed / Failed).

## ✨ Key Features

- **Processing:** The frontend remains fully responsive, even when background processing takes several seconds.
- **Batching:** Users can submit multiple prompts at once (handled seamlessly on the frontend using Promise.allSettled).
- **Error handling:** Sending the exact phrase `error` will intentionally trigger a backend exception, demonstrating how the system gracefully handles and displays the Failed status.
- **Real-time UX:** Clear and dynamic status tracking implemented via a custom `StatusBadge` component.

## AI Integration

Initially, this project was built to integrate directly with the Google Gemini API. However, due to recent restrictions on free-tier access, live requests became inconsistent. I considered hosting a local LLM via Ollama, but opted against it so that anyone reviewing this repository wouldn't have to download a 3.5GB+ model just to run the application locally.

To keep the setup lightweight, the application currently defaults to a Mock Response Service. However, the fully functional GeminiService implementation is still in the codebase!

If you have a valid Gemini API key, you can easily switch to the live model by updating the cleanApiKey property in Backend/appsettings.Development.json:

```bash
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "cleanApiKey": "Your_Gemini_API_Key_Here"
}
```

## 🚀 How to Run Locally

The project utilizes Docker to streamline the setup process, eliminating the need to manually install and configure RabbitMQ or PostgreSQL.

### Wymagania

- [.NET 10.0 SDK] (or newer)
- [Node.js]
- [Docker Desktop] (required for the database and message queue)

### 1. Start the Infrastructure (RabbitMQ & PostgreSQL)

Using Docker Compose, you can spin up the required services in the background. In the root directory (where the `docker-compose.yml` file is located), run:

```bash
docker compose up -d
```

### Haslo do DB

Dodaj plik .env w Backend/ i wpisz w nim hasło do bazy, w tym przypadku na potrzeby projektu:

```bash
DB_PASSWORD=SecretPassword123!
```

### 2. Configure the Database

Create a `.env` file inside the `Backend/` directory and add the database password required for this local environment:

```bash
cd Backend
dotnet run
```

### 3. Run the Backend

Navigate to the backend directory and start the application. Entity Framework Core migrations will automatically run on startup to create and seed the database schema:

```bash
cd Backend
npm run dev
```

### 4. Run the Frontend

```bash
cd Frontend
npm install
npm run dev
```

### 5. Access the Application

Open your browser and navigate to: http://localhost:5173/

### 6. Running Tests

The project includes basic automated tests. To run them, navigate to the Tests/ directory and execute:
`dotnet test`

## 🔮 Future Improvements

- **Live AI Integration:** Implement a permanent, reliable connection to an active LLM API.
- **Unit Tests Expansion:** Add more comprehensive Unit and Integration tests.
- **Logging:** Integrate external logging telemetry (e.g., Application Insights or Serilog).
- **Containerization:** Create dedicated `Dockerfile` configurations for the .NET API and React frontend so the entire system can be spun up purely through Docker

## Development Log

Interested in seeing how this system was built from the ground up? Check out the DEVELOPMENT_LOG.md file, where I documented my step-by-step progress, challenges, and architectural decisions during the development process!

(Note: Throughout the codebase and the development log, you might occasionally find comments written in both English and Polish, reflecting my raw thought process as the project evolved, but for the sake of keeping the thinking process raw, i decided to keep it polish :))
