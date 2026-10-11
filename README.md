# UVID Stationery Store

This package contains the customer shop, admin portal, product photos, and a
backend that saves products and uploaded images. The deployment setup uses
**Vercel for the website** and **Render for the API and persistent data**.

## Deploy the full website

The customer page can be hosted as static files, but the admin portal needs a
running API and a persistent database for product changes and uploaded photos.
This package connects the Vercel site to a Render service, so the customer page
and admin portal use the same live catalog. Render's persistent disk requires
a paid Render web service plan.

### 1. Put the extracted project in a GitHub repository

1. Extract the ZIP on your laptop.
2. Create a GitHub repository and upload the contents of the extracted
   `stationery` folder to the repository's top level. The files
   `render.yaml` and `vercel.json` should be at the top level.
3. Do not upload a `server/.env` file. Set production secrets in Render in the
   next step instead.

### 2. Create the backend and persistent storage on Render

1. In Render, choose **New → Blueprint** and connect the GitHub repository.
2. Render reads `render.yaml` and creates the Node service with a 1 GB persistent
   disk mounted at `/var/data`.
3. Set `ADMIN_EMAIL` to your admin Gmail address and `ADMIN_PASSWORD` to a
   strong admin password when Render asks. If it does not ask during Blueprint
   setup, add both in the service's **Environment** settings before its first
   successful start. Render generates the session-signing secret for you. The
   production launcher will stop with a clear message if a required secret is
   missing, instead of using a default password.
4. Wait for the service to finish deploying, then copy its address, such as
   `https://uvid-stationery-api.onrender.com`.

The first start restores the packaged catalog and product photos to the
persistent disk. New products and uploaded images are saved there as well, so
they remain available after a restart or redeploy. For privacy, this package's
initial catalog copy excludes local customer accounts, orders, sign-in codes,
audit history, and seminar registrations.

### 3. Connect Vercel to the Render service

1. Open the top-level `vercel.json` file.
2. In both destinations, replace only `YOUR-RENDER-SERVICE` with the part before
   `.onrender.com` in the Render address. For example, if Render gave you
   `https://uvid-stationery-api.onrender.com`, use `uvid-stationery-api`.
3. Save and commit that change to GitHub.
4. In Vercel, choose **Add New → Project**, import the same GitHub repository,
   and deploy it. Use the project root as the root directory, **Other** as the
   framework preset, and leave the build command empty.

Vercel forwards `/api/...` requests and uploaded-image URLs to Render. The
customer site and admin portal stay on your Vercel address, while both read and
update the same backend catalog.

### 4. Attach your domain

In Vercel, open the project's **Settings → Domains**, add your domain, and copy
the DNS records Vercel shows into your domain provider. Once DNS is verified,
the customer site and `/admin/` will both use your domain.

## Sign-in

- Customers enter a Gmail address and click **Sign in**. There is no customer
  password or one-time code. The site does not verify ownership of the email
  address.
- The admin portal is at `/admin/`. Admins sign in with the email and password
  entered for `ADMIN_EMAIL` and `ADMIN_PASSWORD` in Render.

## Run it on Windows first

1. Install Node.js LTS if it is not already installed: <https://nodejs.org>.
2. In `server`, copy `.env.example` to `.env` and set a local admin email and
   password. The `.env` file is for your laptop only; do not upload it.
3. Double-click `START-STORE.bat` in the `stationery` folder.
4. Open the customer site at <http://localhost:8787/> and the admin panel at
   <http://localhost:8787/admin/>.

Keep the black server window open while using the local site. Closing that
window stops the local server. The deployed website runs on Render and does not
depend on your laptop staying on.

## Where files and data are stored

- The logo and packaged product photos are in `assets/` and ship with the site.
- The product database and admin-uploaded images use the persistent disk on
  Render (`/var/data`). The laptop version stores these in `server/data/`.
- `vercel.json` sends API calls and `/uploads/` files to the Render service.
- `render.yaml` defines the backend service and its persistent disk.

If you deploy only to Vercel and skip Render, the website's static pages and
packaged images can load, but customer sign-in, admin changes, and saved
product uploads will not work.
