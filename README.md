# blur_UI

A Vue 3 + Vite front-end for a face-blurring workflow. The app lets a user upload:

- a reference face image to keep clear
- a group image containing multiple people

It then submits the images to a local backend API, fetches the processed result, and allows the user to download the final image.

## What this project does

The interface is intentionally small and focused on a single workflow:

1. Select a reference image containing exactly one face.
2. Select a group image containing multiple faces.
3. Submit both files to the backend at `http://127.0.0.1:8000/process-images`.
4. Fetch the processed image from `http://127.0.0.1:8000/download`.
5. Download the generated result.

The actual image processing is not implemented in this repo; this project is the UI layer that calls the backend service.

## Project structure

```text
blur_UI/
├── public/
│   └── favicon.ico
├── src/
│   ├── App.vue
│   ├── main.ts
│   └── ...
├── .vscode/
│   ├── extensions.json
│   └── settings.json
├── env.d.ts
├── index.html
├── package.json
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
├── vite.config.ts
├── README.md
└── package-lock.json
```

## Key application logic

The UI logic in `src/App.vue` handles three main tasks:

- file validation for uploaded images
- form submission to the backend upload endpoint
- fetch/download of the generated processed image

### Upload flow

The form uses `FormData` and posts the following fields:

- `face_profile`
- `group_image`

The expected backend endpoint is:

```http
POST http://127.0.0.1:8000/process-images
```

The frontend expects a JSON response shaped like:

```json
{
  "face_profile": {
    "filename": "profile.jpg",
    "path": "/tmp/profile.jpg"
  },
  "group_image": {
    "filename": "group.jpg",
    "path": "/tmp/group.jpg"
  }
}
```

### Download flow

After processing, the app requests:

```http
GET http://127.0.0.1:8000/download
```

It expects a binary image response, usually served as a file blob with a `Content-Disposition` header that includes a filename.

## Requirements

- Node.js 22.18.0+ or 24.12.0+
- npm
- a backend service running locally on port `8000`

## Setup

Install dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

The app will typically run on the Vite local dev server URL printed in the terminal, usually `http://localhost:5173`.

## Production build

Create a production build:

```bash
npm run build
```

This runs TypeScript checking via `vue-tsc` and then builds the Vite bundle.

## Development notes

### Type checking

The project uses `vue-tsc` rather than raw `tsc` because Vue SFCs require template-aware type checking.

```bash
npm run type-check
```

### Build script

The build command is configured in `package.json` as:

```json
"build": "run-p type-check \"build-only {@}\" --"
```

This means the project checks types before generating the final bundle.

## Usage checklist

Before using the UI, make sure the backend service is running and available at `127.0.0.1:8000`.

If the app shows an error, confirm:

- both images are valid image files
- the backend is reachable
- the backend expects the same `FormData` field names
- the backend is returning an image for the download endpoint

## Troubleshooting

### "Please select both images before submitting."

This means one or both file inputs are empty. Select both files before clicking the upload button.

### "Please select an image file."

The browser only accepts files whose MIME type starts with `image/`.

### Network errors from the frontend

The app hard-codes the backend URL:

- `http://127.0.0.1:8000/process-images`
- `http://127.0.0.1:8000/download`

If the backend is not running or is using a different host/port, update the fetch URLs in `src/App.vue`.

## Notes

This repo is a front-end-only implementation. The actual face detection and blurring logic must exist in the backend service that `blur_UI` communicates with.
