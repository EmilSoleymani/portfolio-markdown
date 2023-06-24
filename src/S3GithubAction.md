# S3 Uploads Using GitHub Actions

- Author: Emil Soleymani
- Date: 2023-06-17

The following article outlines the process of uploading files to an S3 Bucket using GitHub actions.

## Marketplace Actions

Luckily `keithweaver/aws-s3-github-action@v1.0.0` provides a simple way to upload files to S3 Buckets via GitHub Actions. This section will outline building a `React` app using `npm`, and uploading the `build` folder to a public S3 bucket to be hosted as a static website.

First we create a new action in `.github/workflows` directory, called `Upload.yaml`. Add the following line:

<Code language='yaml'>
name: Upload S3
</Code>

Next, we setup triggers for our GitHub Actions, and optionally environment variables for the bucket name and region:

<Code language='yaml'>
on:
  push:
    branches: main
    paths: ['package.json', 'public/**', 'src/**', '.github/workflows/Publish.yaml']

env:
  S3_BUCKET_URL: s3://\<your-bucket-name>/
  S3_BUCKET_REGION: \<region-name>
</Code>

This will ensure that this action runs on pushes to `main` branch (Note: You should always set branch protection rules to protect main branch from having pushes, as I do for my repo, meaning this **ONLY** runs on merged pull requests). Furthermore, we wouldn't want to upload to S3 on every change - e.g. test cases or changes to docs or `README` - which is what the `paths` parameter can help specify.

Next, we specify the `jobs`. We will have two jobs in this workflow, `build` and `upload`. Note that all jobs run in parallel unless dependencies are specified. In our case, we will need the `build` to complete prior to the `upload` step, so we will specify the dependency using the `needs` parameter. 

<Code language='yaml'>
jobs:
  build:
    runs-on: [ubuntu-latest]
    steps:
      - # Step 1 ...
  upload:
    runs-on: [ubuntu-latest]
    needs: build  # Make sure upload runs after build
    steps:
      - # Step 1 ...
</Code>

### Build Job

The job can be built using `npm` by following these steps:

1. Checkout the code from the `GitHub` repository using the `actions/checkout@v3`.

2. Install and setup `Node` and `npm` using `actions/setup-node@v3` (you can also specify the Node version you want here).

3. Install dependencies

4. Build project

Putting these together produces:

<Code language='yaml'>
steps:
  - name: Checkout
    uses: actions/checkout@v3

  - name: Setup Node
    uses: actions/setup-node@v3
    with:
      node-version: \<your-node-version>

  - name: Install dependencies
    run: npm install

  - name: Build
    run: npm run build
</Code>

This will produce a static version of your project in the `build/` directory. However, this is considered an artifact of the job, and will not be passed onto the `upload` step! We will have to use `actions/upload-artifact@v2` and `actions/download-artifact@v2` to acheive this. Add the following step to the `build` job:

<Code language='yaml'>
- name: Temporarily save build path
  uses: actions/upload-artifact@v2
  with:
    name: \<artifact-name>
    path: ./build
    retention-days: 3
</Code>

### Upload Job

Following from the last step in the `build` job, we will first have to download the saved artifact using `actions/download-artifact@v2`.

## Manual