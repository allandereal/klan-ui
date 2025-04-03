# Klan Violations UI

This simple interface displays events that are considered a violation
in the Klan Logistics Limited fleet management processes in a table with functionality
to take actions on each violation.
It is built using NuxtJs 3.

## Setup

Clone the repository from github

```bash
#git
git clone https://github.com/allandereal/klan-ui
```

Make sure to install the dependencies:

```bash
# npm
npm install
```

## Development Server

Make sure to first run the API from [https://github.com/allandereal/klan-api](https://github.com/allandereal/klan-api):

Then set these env variable in the .env file in the project directory:

```bash
# bash
touch .env

#add these values
NUXT_CMS_API_PASSCODE=
NUXT_PUBLIC_APP_API_BASE=http://127.0.0.1:8000/api
NUXT_PUBLIC_CMS_API_BASE=https://cms.klanlogistics.com:8443/api

```

Start the development server on `http://localhost:3000`:

```bash
# npm
npm run dev
```
