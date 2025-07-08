# Promptfoo <!-- omit in toc -->

- [Overview](#overview)
- [Setup](#setup)
  - [Docker](#docker)
- [Configure](#configure)
  - [Telemetry](#telemetry)
  - [Tracing - OpenTelemetry](#tracing---opentelemetry)
- [Evaluation](#evaluation)
- [Red Teaming](#red-teaming)

## Overview

[promptfoo](https://www.promptfoo.dev/) is an open-source CLI and library for evaluating and
red-teaming LLM apps.

[Privacy policy](https://www.promptfoo.dev/privacy/)

## Setup

### Docker

```yml
services:
  promptfoo:
    image: ghcr.io/promptfoo/promptfoo:latest
    ports:
      - "3100:3000"
    environment:
      # User docker container to resolve Ollama, must be on same network.
      - 'OLLAMA_BASE_URL=http://ollama:11434'
      - 'PROMPTFOO_DISABLE_TELEMETRY=1'
    volumes:
      - promptfoo_data:/home/promptfoo/.promptfoo
volumes:
   promptfoo_data:
```

To map to a local folder:

```yml
    volumes:
      - ./data/promptfoo:/home/promptfoo/.promptfoo
```

Ensure correct folder permissions:

```shell
mkdir -p ./data/promptfoo
sudo chmod 777 ./data/promptfoo
```

## Configure

### Telemetry

To disable telemetry, set the following environment variable: `PROMPTFOO_DISABLE_TELEMETRY=1`.

Promptfoo hosts free unaligned inference endpoints for harmful test case generation when running
`promptfoo redteam generate`. Disable remote generation: `PROMPTFOO_DISABLE_REDTEAM_REMOTE_GENERATION=1`

### Tracing - OpenTelemetry

@See <https://www.promptfoo.dev/docs/tracing/>

```yml
tracing:
  enabled: true # Required to send OTLP telemetry
  otlp:
    http:
      enabled: true # Required to start the built-in OTLP receiver
```

## Evaluation

```shell
npx promptfoo@latest eval
```

To view reports:

```shell
npx promptfoo@latest view
```

```yml
# yaml-language-server: $schema=https://promptfoo.dev/config-schema.json
# Learn more about building a configuration: https://promptfoo.dev/docs/configuration/guide
description: "eval-demo"
prompts:
  - "Write a tweet about {{topic}}"
  - "Write a concise, funny tweet about {{topic}}"
providers:
  - "ollama:llama3.2:1b"
  - "ollama:llama3.2:latest"
tests:
  - vars:
      topic: bananas
  - vars:
      topic: avocado toast
    assert:
      # For more information on assertions, see https://promptfoo.dev/docs/configuration/expected-outputs
      # Make sure output contains the word "avocado"
      - type: icontains
        value: avocado
      # Prefer shorter outputs
      - type: javascript
        value: 1 / (output.length + 1)
  - vars:
      topic: new york city
    assert:
      # For more information on model-graded evals, see https://promptfoo.dev/docs/configuration/expected-outputs/model-graded
      - type: llm-rubric
        value: ensure that the output is funny
        provider: ollama:llama3.2:latest
```

## Red Teaming

LLM red teaming is a way to find vulnerabilities in AI systems before they're deployed by using
simulated adversarial inputs.

<!-- textlint-disable -->
- Automatically scans 50+ vulnerability types:
  - Security & data privacy: jailbreaks, injections, RAG poisoning, etc.
  - Compliance & ethics: harmful & biased content, content filter validation, compliance, etc.
  - Custom policies: enforce organizational guidelines.
- Generates dynamic attack probes tailored to your application using specialized uncensored models.
- Implements state-of-the-art adversarial ML research from Microsoft, Meta, and others.
- Integrates with CI/CD.
- Tests via HTTP API, browser, or direct model access.
<!-- textlint enable -->

To install:

```shell
npx promptfoo@latest redteam setup
```

To run:

```shell
npx promptfoo@latest redteam run
```

To view reports:

```shell
npx promptfoo@latest redteam report
```
