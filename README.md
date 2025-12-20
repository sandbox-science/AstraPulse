# AstraPulse

A minimalist timer app with a focus on simplicity and ease of use.

### Run dev

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
npm run dev
# Open a new terminal window in root directory and run:
cargo tauri build
```

## Notes

- The frontend is implemented with Leptos and lives in `frontend/`.
- The backend is implemented with Rust and Tauri, and lives in `src-tauri/`.