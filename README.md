# J.A.R.V.I.S. MK IV Assistant

A full-stack J.A.R.V.I.S. assistant with an Iron Man inspired holographic interface.

## Project Structure

```
├── backend/          # Django REST API backend
│   ├── jarvis_backend/
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── manage.py
│   └── requirements.txt
├── frontend/         # Vue 3 + Vite frontend
│   ├── src/
│   │   ├── App.vue
│   │   └── main.js
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
└── README.md
```

## Backend Setup (Django)

1. Navigate to the backend directory:
   ```bash
   cd backend
   ```

2. Create a virtual environment (optional but recommended):
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Run the development server:
   ```bash
   python manage.py runserver
   ```

The backend will be available at `http://localhost:8000`

## Frontend Setup (Vue 3 + Vite)

1. Navigate to the frontend directory:
   ```bash
   cd frontend
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Run the development server:
   ```bash
   npm run dev
   ```

The frontend will be available at `http://localhost:5173`

## Features

- **Holographic Iron Man UI**: Three-column layout with rotating arc reactor animation
- **Voice Synthesis**: J.A.R.V.I.S. speaks responses using Web Speech API
- **Speech Recognition**: Voice command input support
- **Activity Logging**: Real-time activity log panel
- **Responsive Design**: Adapts to different screen sizes

## Speech Fix Notes

The application implements several fixes for browser speech synthesis:
1. `speechSynthesis.resume()` is called on user click to unlock the audio context
2. A silent utterance is played to fully unlock audio on first interaction
3. Proper handling of `onvoiceschanged` event ensures voices are loaded
4. Fallback to any available English voice if preferred voice is not found
