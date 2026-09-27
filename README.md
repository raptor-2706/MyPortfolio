Project Core architecture (all free-tier eligible)

This is the standard beginner pattern for hosting a static portfolio site — no servers, no maintenance, scales automatically.

S3 bucket — holds your HTML/CSS/JS/images, configured for static website hosting.
CloudFront — sits in front of S3, caches your site globally (faster loads), and terminates HTTPS.
ACM (Certificate Manager) — issues a free SSL certificate for your custom domain, attached to CloudFront.
Route 53 — manages DNS for your domain, points it at CloudFront.

Build order: create the S3 bucket → upload your site → request an ACM cert for your domain (must be in us-east-1 for CloudFront) → create the CloudFront distribution pointing at the bucket, with the cert attached → in Route 53, add an A record (alias) pointing your domain at the CloudFront distribution.

-----------------------------------------------------------------------------------------------------------

What does CloudFront do and why use it with S3?

— this is the part that trips people up because S3 can serve a website on its own, so it's not obvious why you'd add CloudFront.

What CloudFront actually is: a CDN (content delivery network) — a global network of AWS edge locations that cache copies of your files close to wherever your visitors are. Someone in Tokyo hits an edge server in Tokyo, not your S3 bucket sitting in us-east-1.

Why you'd put it in front of S3 for a portfolio site specifically:

HTTPS on your own domain. S3 static website hosting doesn't support custom SSL certificates directly. If you want https://yourname.com with a valid padlock, CloudFront (paired with an ACM certificate) is the standard way to get it. Without it, you're stuck on the default http:// S3 website endpoint.
Speed. Once a file is cached at an edge location, repeat visitors (or anyone else near that edge) get it in milliseconds instead of a round trip to your bucket's region. For a portfolio, this mostly matters for good Lighthouse/PageSpeed scores — which recruiters or hiring managers might actually glance at.
Cost. Data transfer out from CloudFront is generally cheaper than data transfer directly out of S3, and CloudFront absorbs repeat requests via caching so your bucket gets hit less.
Security posture. With CloudFront in front, you can make the S3 bucket itself private (block all public access) and only let CloudFront read from it, using an Origin Access Control (OAC). That's a meaningfully better setup than a publicly-readable bucket, and it's the kind of detail that shows you understand least-privilege access if someone reviews your architecture.
The trade-off: it's one more moving part to configure (distribution settings, cache invalidation when you update files, OAC permissions), and cache invalidation means changes don't show up instantly unless you invalidate the cache or version your file names. For a first build, S3 + CloudFront + ACM + Route 53 is still the standard beginner-to-intermediate pattern — just know that skipping CloudFront (S3 website hosting alone) is a valid simpler starting point if you want fewer pieces to learn first.

-----------------------------------------------------------------------------------------------------------

Scaffold the React + Tailwind app locally
Run `npm create vite@latest my-portfolio -- --template react` (Vite is faster and simpler than Create React App for this). Then install Tailwind following its Vite guide. Get `npm run dev` showing a blank page in your browser before doing anything else — this is today's goal.

Push it to GitHub
Create a new repo on GitHub, then `git init`, `git remote add origin ...`, and push. Do this now, before you've written real content, so every change from here on is version-controlled and you have a deploy target ready.

Build out the portfolio content
Add your sections (about, projects, contact) as React components, style with Tailwind, test entirely with `npm run dev`. No AWS needed yet — this is pure frontend work and the fastest feedback loop you'll get.

Create an AWS account and an IAM user
Sign up for AWS if you haven't, then immediately create an IAM user with the permissions you need (S3, CloudFront, ACM, Route 53) instead of using your root account for daily work. This is a real security habit worth having from day one.

Run npm run build and create your S3 bucket
`npm run build` produces a `dist/` folder of static HTML/CSS/JS — that's what gets hosted. Create an S3 bucket, keep it private (block public access), and upload the `dist/` contents manually the first time just to see it work.

Add CloudFront, ACM, and Route 53
Request an ACM certificate for your domain (in us-east-1), create a CloudFront distribution pointing at the S3 bucket via Origin Access Control, attach the cert, then point Route 53 at the CloudFront distribution. This is the step we covered earlier.

Automate deploys with GitHub Actions
Write a workflow that runs `npm run build` and syncs `dist/` to S3 (plus a CloudFront cache invalidation) on every push to main. From here on, `git push` is your entire deploy process.

Add a backend piece once the site is live
For a contact form or any dynamic feature, add API Gateway + Lambda + DynamoDB (or SES for email). Do this last — it's isolated from the static site and easiest to reason about once the core site is already working.

-----------------------------------------------------------------------------------------------------------

1. Add Tailwind to your Vite project

npm create vite@latest MyPortfolio -- --template react
cd MyPortfolio
npm install tailwindcss @tailwindcss/vite

In vite.config.js, add the Tailwind plugin:

js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
})

In src/index.css (or wherever your global CSS is), replace the contents with:

css
@import "tailwindcss";

Run npm run dev

----------------------------------



