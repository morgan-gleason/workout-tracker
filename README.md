# Workout Tracker

A simple, single-file workout tracking app. No backend, no build step — just open `index.html` in a browser, or host it for free on GitHub Pages.

## Features

- **Log workouts**: exercise, machine/equipment, sets (weight × reps, optional RPE), and notes.
- **How I feel**: next-day soreness and energy check-ins, with optional muscle-group notes.
- **History**: browse and filter past sessions by exercise.
- **Trends**: charts for top weight over time, training volume, and recovery (soreness/energy) trends per exercise.
- **Backup**: export/import your data as JSON. Everything is stored in your browser's local storage — nothing leaves your device.

## Usage

Just open `index.html` in any modern browser. To host it publicly (e.g. via GitHub Pages), push this repo to GitHub and enable Pages in the repo settings, pointing at the `main` branch root.

## Tech

Vanilla HTML/CSS/JS, [Chart.js](https://www.chartjs.org/) via CDN for charts. No dependencies to install, no build process.
