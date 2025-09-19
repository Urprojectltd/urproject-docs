# Sanctum / Auth Flow Cheat Sheet

## Endpoints
- `/sanctum/csrf-cookie` → get CSRF token
- `/login` → authenticate user
- `/logout` → end session
- `/user` → fetch current user

## Frontend Notes
- Always call `/sanctum/csrf-cookie` before `/login`
- Axios must send `withCredentials: true`

