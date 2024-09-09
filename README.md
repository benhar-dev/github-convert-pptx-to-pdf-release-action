# GitHub Action - .pptx to released zipped pdf

## Disclaimer

This is a personal guide not a peer reviewed journal or a sponsored publication. We make
no representations as to accuracy, completeness, correctness, suitability, or validity of any
information and will not be liable for any errors, omissions, or delays in this information or any
losses injuries, or damages arising from its display or use. All information is provided on an as
is basis. It is the reader’s responsibility to verify their own facts.

The views and opinions expressed in this guide are those of the authors and do not
necessarily reflect the official policy or position of any other agency, organization, employer or
company. Assumptions made in the analysis are not reflective of the position of any entity
other than the author(s) and, since we are critically thinking human beings, these views are
always subject to change, revision, and rethinking at any time. Please do not hold us to them
in perpetuity.

## Overview

This action was created to convert pptx files to pdf, then to zip and release them as an asset.

A demo of this in action can be found [here](https://github.com/benhar-dev/github-convert-pptx-to-pdf-release-action-demo).

## Getting Started

1. Add the convert-pptx-to-pdf to the .github\workflows folder in your repo.
2. Commit and push to github
3. Add a pptx file to the root of your repo
4. Commit and push to github

You will see a new release is made with a zip file containing the pptx files as pdf.

## Error: Resource not accessible by integration

Depending on your GitHub Settings you may need to enable action privileges.

1. Go to Settings > Actions > General
2. Scroll down to Workflow permissions
3. Select "Read and write permissions"
4. Click Save

## Code Snippet

```yml
name: Convert PPTX to PDF and Create Release with PDF Zip

on:
  push:
    paths:
      - "**/*.pptx"

  workflow_dispatch:

jobs:
  convert-and-release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install LibreOffice
        run: sudo apt-get install -y libreoffice

      - name: Clear old PDFs in handouts folder
        run: |
          mkdir -p handouts
          rm -f handouts/*.pdf

      - name: Convert PPTX to PDF and move to handouts folder
        run: |
          find . -name "*.pptx" | while IFS= read -r file; do
            echo "Found file: $file"
            abs_path=$(realpath "$file")
            echo "Absolute path: $abs_path"
            echo "Converting file to PDF..."
            soffice --headless --convert-to pdf "$abs_path" --outdir handouts
            if [ $? -eq 0 ]; then
              echo "Conversion successful for $abs_path"
            else
              echo "Error in conversion for $abs_path"
            fi
          done

      - name: Create zip of PDFs
        run: |
          zip -r handouts.zip handouts/

      - name: Create GitHub Release
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: "pdf-release-${{ github.sha }}"
          release_name: "PDF Release ${{ github.sha }}"
          draft: false
          prerelease: false

      - name: Upload PDF zip to Release
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./handouts.zip
          asset_name: handouts.zip
          asset_content_type: application/zip
```
