+++
date = '2025-11-14T10:32:14-05:00'
draft = false 
title = 'Building My Perfect Digital Library'
+++

## Intro
Tired of a scattered digital library? I know I am. Trying to organize all my digital media--books, manga, textbooks, homework--between all of my various devices is a nightmare. Enter MyBookShelfApp (WIP Name). MyBookshelfApp is a self-hosted platform designed to centralize your personal collection. While the current prototype focuses on PDFs, the vision is a universal digital library accessible anywhere with an internet connection.

## The Tech Stack

For this project, I decided to opt for two frameworks I've never worked with: React and FastAPI.

- **Backend (FastAPI and SQLite):** - FastAPI was a joy to work with. Coming from Flask, defining REST endpoints felt intuitive, and I especially appreciated how it leverages Pydantic for built-in request/response validation, simplifying data handling for book uploads and edits. SQLite was also wonderful, allowing me to rapidly prototype and iterate without having to go through the hassle of setting up a Postgres server.
- **Frontend (React, TypeScript & Vite):** React + TypeScript definitely introduces complexity over Vanilla JS. But the payoff in building modular, expressive UI components was massive, especially when designing the PDF viewer and dynamic book lists. Furthermore, TypeScript provides type safety, making it significantly easier to build a stable front-end.

## The Prototype

After days of juggling my time between swim meets and school work, I was finally able to pull together a working prototype. The features that work include:
- **Secure Multi-User System:** Implemented robust user authentication using JWTs with dedicated access and refresh tokens.
- **Book Management:** Users can securely upload, edit metadata (title/author), and delete books.
- **Preview Generation:** A working book thumbnail generator provides visual previews for a better browsing experience.
- **Integrated Viewer:** A functional PDF viewer allows for in-app reading.

## What's Next?

With the core functionality in place, the next phase focuses on stability, maintainability, and feature expansion:

1. **SQLAlchemy Integration:** Moving from direct SQLite calls to an Object Relational Manager will abstract the database layer, making future migrations to scalable systems like Postgres seamless.
2. **Major Backend Refactoring:** This is crucial for simplifying maintenance, improving the developer experience, and welcoming outside contributors (like my friend who is already hopping in!).
3. **Testing Implementations:** Introducing Unit and Integration Tests to ensure stability and prevent regressions as the codebase grows.
4. **Metadata Extraction:** Implementing logic to automatically pull rich information (author, publication date, etc.) from uploaded file metadata. I've heard that `pdfminer` may be just what i need.

I’m really excited about the direction this project is taking, and can't wait to expand on the prototype.

Check out the code on GitHub right now! The repo is admittedly a bit of a mess as I begin the refactor, but I’d love to hear your outside feedback on the current state or future features!

[Link to the Github](https://github.com/CaldwellDN/bookshelfapp)
