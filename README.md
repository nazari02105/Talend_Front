# Talend Front End

The front end of a small **ETL tool**, a simple Talend-like web app. It is written with **Angular 12** and **Angular Material**. The interface is in **Persian** (right-to-left).

With this app you upload datasets (CSV files or SQL Server tables) and build **pipelines** on a visual board: you pick a source dataset, add steps (filter, join, aggregate), pick a destination dataset, and run the pipeline.

This app needs its back end, which is a separate project: **[Talend_Back](https://github.com/nazari02105/Talend_Back)** (ASP.NET Core API). The front end always sends its requests to `https://localhost:5001`.

## Features

- **Landing page** with links to sign up and log in
- **Sign up** (email, password, username, first name, last name) and **log in**
- **Datasets page**
  - Upload a CSV file
  - Add a SQL Server table as a dataset: enter the database name, username, password and address, then pick a table from the list
  - Open a dataset to see its data in a table; delete a dataset
- **Pipelines page:** list, create and delete pipelines. New pipelines get the names `pipeline 1`, `pipeline 2`, …
- **Pipeline board** (graph editor)
  - Choose the source and destination datasets
  - Add steps: **filter** (conditions joined with and / or), **join** (with a second dataset, on a left key and a right key) and **aggregate** (sum, count, min, max, average, with group-by columns)
  - Set each step in a side panel, and see a preview table of the data
  - **Run** the pipeline, see its status, **cancel** it, and **download** it as a YAML file
- Pages after login are protected: if you are not logged in, you are sent to the login page

## Requirements

- **Node.js 16** (recommended). Angular 12 officially supports Node 12, 14 and 16.
  Newer Node versions (17 and above) also work, but only with an extra setting, see [Newer Node versions](#newer-node-versions).
- **npm** (comes with Node.js)
- The **[Talend_Back](https://github.com/nazari02105/Talend_Back)** API running on `https://localhost:5001`

You don't need to install Angular CLI globally: `npm install` installs the right version (12.2) inside the project.

## How to run

1. **Start the back end first** (see the [Talend_Back README](https://github.com/nazari02105/Talend_Back)). Make sure <https://localhost:5001/swagger> opens in your browser **without a certificate warning**. If the browser doesn't trust the back end's certificate, every request from the front end fails. Run `dotnet dev-certs https --trust` on the back-end computer to fix this.

2. **Install the packages** from the project folder:

   ```bash
   npm install
   ```

3. **Start the front end:**

   ```bash
   npm start
   ```

   (This runs `ng serve`.) Wait for `Compiled successfully`, then open **<http://localhost:4200>**.

   Use port **4200**. The back end only accepts browser requests (CORS) from `http://localhost:4200`.

4. **Sign up**, then **log in with your username**. The back end also accepts your email at login, but the front end then uses the email as your username, and your datasets and pipelines lists don't load. So use the username.

5. After login you go to the **datasets** page. Upload a CSV file or add a SQL Server table, then go to the **pipelines** page and create a pipeline.

### Newer Node versions

On Node 17 or newer, `npm start` and `npm run build` stop with this error:

```
error:0308010C:digital envelope routines::unsupported   (ERR_OSSL_EVP_UNSUPPORTED)
```

Angular 12 uses an old hashing method that newer Node versions block. Use Node 16, or turn on the old method before you start the app:

- **macOS / Linux:**

  ```bash
  export NODE_OPTIONS=--openssl-legacy-provider
  npm start
  ```

- **Windows (PowerShell):**

  ```powershell
  $env:NODE_OPTIONS="--openssl-legacy-provider"
  npm start
  ```

- **Windows (Command Prompt):**

  ```bat
  set NODE_OPTIONS=--openssl-legacy-provider
  npm start
  ```

I tested this way on Node 22: `npm install` and `npm start` work, and the landing, login and sign-up pages open.

## Building for production

`npm run build` (production build) **fails** as the project is now:

```
Error: initial exceeded maximum budget. Budget 1.00 MB was not met by 1.33 MB with a total of 2.33 MB.
```

The app is 2.33 MB, but `angular.json` allows at most 1 MB. To build anyway, either:

- make a development build: `npx ng build --configuration development`, or
- raise the `"initial"` budget in `angular.json` (under `configurations` → `production` → `budgets`), for example `"maximumError": "3mb"`, then run `npm run build`.

The built files go to `dist/team3-project/`.

## Notes

- **CSV upload:** the app always sends the file as **comma-separated, with a header row**. Use CSV files in this format.
- **The back-end address is fixed** to `https://localhost:5001` in the code (in `src/app/services/`, `src/app/identify/gaurd/auth.service.ts` and `src/app/dataset/banner/banner.component.ts`), not in `environment.ts`. To use another address, change it in these files.
- **Login is saved in the browser** (`localStorage`). The back end keeps tokens in memory, so after the back end restarts your saved login stops working. If pages stop loading data, log out and log in again.
- **Fonts:** the Roboto font and Material icons load from Google Fonts, so they need an internet connection. The Persian font (IRANSans) is included in the project.
- **Pipeline names:** new names are made from the **last digit** of the last pipeline's name. After `pipeline 10`, the next new pipeline is named `pipeline 1` again.
- **Graph library license:** the pipeline board uses `src/assets/ogma.min.js`, which is **Ogma 2.9.3** by Linkurious. Ogma is a **commercial library** ("© Linkurious 2020. All rights reserved."), not open source. Check its license before you share or reuse this project.
- `npm test` runs the default Angular unit tests with Karma and needs Chrome. These tests were made by Angular CLI and were not updated for the app.

## Project structure

| Folder (in `src/app/`) | What it is |
|---|---|
| `landing/` | Landing page with its header and footer |
| `identify/` | Sign-up and login pages, and the login guard (`gaurd/auth.service.ts`) |
| `layouts/` | Page frame after login: header, side menu, footer |
| `dataset/` | Datasets page: CSV upload, SQL Server form, dataset list and data table |
| `pipelines-list/` | Pipelines page (list, create, delete) |
| `pipeline/` | Pipeline board: graph, header (run, cancel, download), filter, join and aggregate panels, preview table |
| `services/` | Requests to the back end (users, datasets, pipelines) |
| `material/` | Angular Material modules |
| `common/` | Pop-up messages (Material snack bar) |

`src/assets/` has the images, icons, fonts and the Ogma graph library.

## Tech

- Angular 12.2, TypeScript 4.3, RxJS 6.6
- Angular Material and CDK 12.2
- Ogma 2.9.3 (graph drawing)
