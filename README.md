name: STR8BARZ CI

on:
  workflow_dispatch:
    inputs:
      prompt:
        description: "Generation prompt"
        required: true
        type: string
      tags:
        description: "Comma-separated tags"
        required: false
        default: ""
        type: string
      duration:
        description: "Duration seconds"
        required: false
        default: "30"
        type: string
      seed:
        description: "Seed"
        required: false
        default: ""
        type: string
      format:
        description: "wav or mp3"
        required: false
        default: "wav"
        type: string
      fail_on_warn:
        description: "Fail on warnings"
        required: false
        default: "false"
        type: string

jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run STR8BARZ
        run: |
          set -e
          ARGS="--prompt '${{ inputs.prompt }}' --tags '${{ inputs.tags }}' --duration ${{ inputs.duration }} --format ${{ inputs.format }} --print-json"
          if [ -n "${{ inputs.seed }}" ]; then
            ARGS="$ARGS --seed ${{ inputs.seed }}"
          fi
          if [ "${{ inputs.fail_on_warn }}" = "true" ]; then
            ARGS="$ARGS --fail-on-warn"
          fi
          python str8barz_runner.py $ARGS

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: str8barz-artifacts
          path: |
            runs/**
            artifacts/**
