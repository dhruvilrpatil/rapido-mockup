<div align="center">

# Rapido Mockup (GigFolio Integration)

This is a modern, responsive frontend application serving as a mockup for the **Rapido Partner Portal**. It demonstrates the integration of a unified, cross-platform reputation system (**GigFolio**) into a specific gig economy platform (Rapido), emphasizing strict data isolation, security, and a seamless user experience.

A separate platform rating system for Zomato delivery partners, integrated with the existing **[GigFolio](https://github.com/dhruvilrpatil/GigFolio)** architecture.

**[Live Demo →](https://dhruvilrpatil.github.io/zomato-mockup/)**

</div>

## 🚀 Features

### 1. Cross-Platform Privacy & Isolation
- **Platform-Specific Data:** While the application fetches the worker's global GigFolio score and tier, it **strictly isolates** reviews and feedback. Rapido can only view and display ratings submitted directly on the Rapido platform (`platform_name === 'Rapido'`).
- **Privacy by Design:** Comments, reviews, and ratings left on other platforms (e.g., Uber, Zomato) are completely inaccessible and invisible within this portal, fulfilling the strict cross-platform isolation requirement.

### 2. Streamlined Rating System
- **Frictionless Submission:** Customers can rate partners using a simple 1-5 star system.
- **Optional Details:** The reviewer's name is optional, and the mandatory "Write a short review" textarea has been completely removed to streamline the feedback process.

### 3. Optimistic UI Updates
- **Instant Feedback:** When a customer submits a rating, the UI updates instantly (optimistically) to reflect the new rating and calculate the new average, ensuring a snappy user experience before the server confirms the database insertion.
- **Data Synchronization:** The rating is then securely inserted into the Supabase backend under the Rapido platform scope.

### 4. Modern, Responsive UI
- **Clear Segmentation:** The interface is divided into two clear logical sections using established eyebrow headers:
  - `PROFILE (VISIBLE TO PARTNER)`: Displays worker identity, UUID, verified badge, and their global scores.
  - `RATING (VISIBLE TO CUSTOMER)`: Provides the streamlined interface for customers to rate the partner.
- **Mobile-First Design:** Fully responsive layout ensuring no horizontal scrolling (`overflow-x: hidden`, `max-width: 100%`). The UI scales elegantly on mobile devices (under 800px) with touch-friendly controls, responsive padding, and word-wrapping for long strings like UUIDs.
- **Rapido Branding:** Incorporates the signature Rapido yellow (`#f9c80e`) and dark accents (`#111827`) for an authentic look and feel.

### 5. Robust Architecture & Security
- **Secure Credentials:** Supabase credentials (`VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`) are abstracted away from the source code using `import.meta.env`.
- **Environment Management:** Utilizes `.env.local` for secure local development (ignored by Git) and provides `.env.example` as a template for new developers.

## 🛠️ Tech Stack
- **Framework:** React + Vite
- **Backend/Database:** Supabase (PostgreSQL)
- **Icons:** Lucide React
- **Styling:** Custom CSS (Responsive, CSS Variables)
- **Deployment:** GitHub Pages (via GitHub Actions)

---

## 💻 Local Development Setup

Follow these steps to run the mockup locally:

### 1. Clone & Install Dependencies
Navigate to the project directory and install the required NPM packages.
```bash
cd g:/mockups/mockups/rapido
npm install
```

*(Note: The first build or dev server start might take a moment to optimize dependencies, particularly the Lucide React icons.)*

### 2. Configure Environment Variables
Copy the example environment file to create your local configuration.
```bash
cp .env.example .env.local
```
Open `.env.local` and add your Supabase URL and Anon Key:
```env
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```
*Security Note: Never commit your `.env.local` file to version control. It is already included in `.gitignore`.*

### 3. Run the Development Server
Start Vite's local development server:
```bash
npm run dev
```
Open the provided `localhost` URL in your browser to view the application.

---

## 🌐 Deployment to GitHub Pages

This project is configured for automated deployment to **GitHub Pages** using GitHub Actions.

### Workflow Configuration
- **File:** `.github/workflows/deploy.yml`
- **Node Version:** Uses Node.js 22 (bypassing Node 20 deprecation warnings).
- **Actions:** Uses `actions/configure-pages@v5` (with `enablement: true`) and `actions/deploy-pages@v4`.
- **Base Path:** The `vite.config.js` is configured with `base: './'` to ensure assets load correctly on GitHub Pages relative paths.

### How to Deploy
1. Push your changes to the `main` branch.
2. The GitHub Action will automatically trigger, build the project (`npm run build`), and upload the artifact.
3. **Important:** Ensure that your GitHub Repository Settings are configured correctly:
   - Go to **Settings > Pages**.
   - Under **Build and deployment**, ensure the **Source** is set to **GitHub Actions**.

---

## 🗂️ Project Structure
```text
/
├── .github/workflows/   # GitHub Actions deployment pipelines
├── src/
│   ├── lib/
│   │   └── supabase.js  # Supabase client initialization
│   ├── main.jsx         # Main React application logic & UI components
│   └── styles.css       # Global styles, variables, and responsive layout
├── .env.example         # Template for environment variables
├── .gitignore           # Ignored files (including .env.local)
├── index.html           # HTML entry point
├── package.json         # Project metadata and dependencies
└── vite.config.js       # Vite build configuration (base path)
```
