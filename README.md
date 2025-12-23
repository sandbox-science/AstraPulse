<p align="center">
  <img src="./src-tauri/icons/128x128@2x.png" alt="AstraPulse Logo">
</p>

<h1 align="center">AstraPulse</h1>

A minimalist timer app designed for simplicity and ease of use. You can start a timer and forget about the app, but it won’t forget you. When the timer reaches zero, the app will emit a background sound to alert you that the time is up. You can focus on your work instead of constantly checking when your next break is.

> [!WARNING]
> 
> AstraPulse has been tested on macOS only.

### Run for development

```bash
# Build the frontend
cd frontend
npm run dev
# Open a new terminal window in root directory and run:
cargo tauri dev
```


### Build for production

```bash
# Build the frontend
cd frontend
npm install
# Open a new terminal window in root directory and run:
cargo tauri build
```

## Notes

- The frontend is implemented with Svelte and lives in `frontend/`.
- The backend is implemented with Rust and Tauri, and lives in `src-tauri/`.