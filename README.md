# Navien Public Documents

A static website built with Astro, deployed on GitHub Pages.

## 🚀 Getting Started

1. Install dependencies:
   ```bash
   npm install
   ```

2. Start the development server:
   ```bash
   npm run dev
   ```

3. Build for production:
   ```bash
   npm run build
   ```

## 📦 Deploying to GitHub Pages

1. Update `astro.config.mjs` with your GitHub username:
   ```js
   site: 'https://yourusername.github.io',
   ```

2. Push to GitHub:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/yourusername/navien-public-documents.git
   git push -u origin main
   ```

3. Enable GitHub Pages in repository settings:
   - Go to Settings → Pages
   - Under "Build and deployment", select "GitHub Actions" as the source
   - The workflow will automatically deploy your site

## 📚 Learn More

- [Astro Documentation](https://docs.astro.build)
- [Astro GitHub Pages Guide](https://docs.astro.build/en/guides/deploy/github/)
