---
---

[[FastAPI]] doesn't come fully configured with logging out of the box.

It's OK to run [[uvicorn]] directly during development, but in production it's recommended to use a process manager such as [[gunicorn]]

[[uvicorn]] & [[gunicorn]] have separate logging configurations, which means we'll likely need different logging configs in dev vs production.

Here's a somewhat dated guide (from 2020): https://pawamoy.github.io/posts/unify-logging-for-a-gunicorn-uvicorn-app
