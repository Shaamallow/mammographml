# README

This was done as a final project for the course 'DataVisualisation' at Ecole Polytechnique in 2023. The goal was to create a website to visualize a dataset (that we could chose). We chose to visualize the data from [UC Irvine Machine Learning Repository](https://archive.ics.uci.edu/dataset/17/breast+cancer+wisconsin+diagnostic), a dataset used as a baseline in this [Article](https://www.semanticscholar.org/paper/Nuclear-feature-extraction-for-breast-tumor-Street-Wolberg/53f0fbb425bc14468eb3bf96b2e1d41ba8087f36) about _Nuclear feature extraction for breast tumor diagnosis_

## TechStack

This was made using :

- [Svelte](https://svelte.dev/) a popular JS Framewok (and my personal favorite)
- [D3.js](https://d3js.org/) a popular JS library for data visualization
- [TailwindCSS](https://tailwindcss.com/) a CSS framework
- [DaisyUI](https://daisyui.com/) a CSS framework compatible with TailwindCSS
- [Python](https://www.python.org/) for data processing, computings some graphs and the differents models we trained

## Install and Run website

Clone the repository :

```bash
git clone https://github.com/Shaamallow/mammographml.git
```

Install dependencies :

```bash
npm install
```

Run in dev mode :

```bash
npm run dev
```

The website should also be available at [mammographml.shaamallow.com](https://mammographml.shaamallow.com/) (at least for project submission).
