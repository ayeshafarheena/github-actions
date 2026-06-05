# 🎬 CineStream — Movies App

Full-stack movies application with React frontend, Python FastAPI backend, and AWS infrastructure.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        GitHub Actions                        │
│  push to main → provision infra → deploy BE → deploy FE     │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┼──────────────┐
        ▼             ▼              ▼
  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐
  │ S3 Bucket│  │ S3 Bucket│  │   EC2 (t2.medium)    │
  │ (Upload) │  │ (Frontend│  │  Python FastAPI BE   │
  │  videos/ │  │  static) │  │  port 8000           │
  │thumbnails│  │  React   │  └──────────┬───────────┘
  └────┬─────┘  └──────────┘             │
       │                                 │
       ▼                                 │
  ┌──────────────────┐                   │
  │  AWS CloudFront  │◄──────────────────┘
  │  CDN Distribution│   serves video URLs
  │  (HTTPS static   │
  │   video URLs)    │
  └────────┬─────────┘
           │
           ▼
      ┌─────────┐
      │ Browser │  streams video via CloudFront
      └─────────┘
```

## S3 Buckets

| Bucket | Purpose | Access |
|--------|---------|--------|
| `movies-app-upload-raw-videos` | Store uploaded videos + thumbnails | Private — accessed via CloudFront OAC |
| `movies-app-static-frontend` | Host built React app | Private — served via CloudFront |

## Features

- 🎬 Upload movies (video + thumbnail) directly to S3
- 🌐 Stream videos via CloudFront CDN (static URLs)
- 🔍 Search movies by title/description
- 🏷️ Filter by category (Action, Drama, Comedy…)
- ⭐ Rate movies (1–5 stars)
- 👁 View count tracking
- 🗑 Delete movies

## GitHub Secrets Required

| Secret | Description |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS IAM access key |
| `AWS_SECRET_ACCESS_KEY` | AWS IAM secret key |

## IAM Permissions Needed

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:*",
    "ec2:*",
    "cloudfront:*",
    "ssm:GetParameter",
    "ssm:SendCommand"
  ],
  "Resource": "*"
}
```

## Project Structure

```
movies-app/
├── .github/workflows/
│   └── deploy.yml              # CI/CD — runs on push to main
├── backend/
│   ├── main.py                 # FastAPI app
│   ├── requirements.txt
│   └── movies-backend.service  # systemd service
└── frontend/
    ├── public/index.html
    ├── package.json
    └── src/
        ├── App.js
        ├── index.js / index.css
        ├── utils/api.js        # Axios API client
        ├── components/
        │   ├── Navbar.js
        │   └── MovieCard.js
        └── pages/
            ├── Home.js         # Search + categories + grid
            ├── MovieDetail.js  # Player + rating + CDN URL
            └── Upload.js       # Upload form + progress
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/movies` | List movies (search, category, pagination) |
| GET | `/movies/{id}` | Get single movie |
| POST | `/movies/upload` | Upload video to S3 |
| POST | `/movies/{id}/rate` | Rate a movie |
| POST | `/movies/{id}/view` | Increment view count |
| DELETE | `/movies/{id}` | Delete movie |
| GET | `/presigned-upload` | Get presigned S3 URL |
| GET | `/categories` | List categories |
