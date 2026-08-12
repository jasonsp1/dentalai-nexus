# DentalAI Nexus — publication boundary

This repository is intended to be public-facing, but the underlying system remains private. Every addition should pass a human review before publication.

## Intended public content

- Project motivation and personal research context.
- High-level architecture and Mermaid diagrams.
- Capability-level descriptions of local inference, retrieval, speech, multimodal processing, orchestration, evaluation, and hardware diversity.
- Aggregate benchmark results whose inputs and outputs can be safely summarized.
- Public upstream references and open-source collaboration interests.
- Clearly labeled implementation status: planned, implemented, experimentally validated, or live-proven.

## Intentionally excluded

- Patient information, PHI, Dentrix data, clinical notes, transcripts, raw audio, identifiers, and encounter records.
- Credentials, API keys, tokens, passwords, `.env` contents, secret filenames, and authentication material.
- Private IP addresses, hostnames, VPN/Tailscale names or addresses, service ports, ingress rules, and network topology.
- Office security configuration, Windows security policy, certificates, firewall rules, and operational recovery procedures.
- Private source code from BossAI or AI-Lab unless explicitly approved for release.
- Private datasets, model weights, generated outputs, embeddings, indexes, and corpus contents.
- Copyrighted, gated, licensed, or proprietary third-party material that cannot be redistributed.
- Exact filesystem paths, mount identifiers, storage serials, database DSNs, model cache locations, and deployment commands.
- Claims derived only from stale plans, superseded architecture, or unverified experiments.

## Review checklist

Before each public update, verify:

- Is every factual claim supported by current private documentation or a reproducible public artifact?
- Does the wording distinguish current evidence from architectural direction?
- Does it reveal any identity, address, path, credential, topology, or office-security detail?
- Does it reproduce any source text, clinical content, private dataset row, prompt, completion, transcript, or image?
- Are third-party model names, benchmarks, and datasets described within their licenses and attribution requirements?
- Is the result clearly labeled as a benchmark, compatibility test, research observation, or clinical evaluation?
- Would a collaborator understand what is available without assuming that the private implementation is open source?

## Licensing posture

The documentation repository should not select a software license for private implementation code that is not present. A repository license can be added later after deciding how to license original documentation, diagrams, benchmark summaries, and any future released components. Upstream projects and model/dataset licenses remain separate and must be respected.
