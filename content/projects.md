---
title: "Projects & Labs"
subtitle: "Hands-on builds, broken down step by step."
description: "Security labs and technical projects by Austin Chen — ransomware internals, Linux backdoors, and Python automation."
layout: "projects"

# Each entry renders as a card. `post` links to the full write-up on this site;
# `repo` and `link` are optional and only render when set.
projects:
  - title: "Self-Hosted Recursive DNS — Pi-hole + Unbound"
    year: "2026"
    summary: >-
      Replaced my home network's dependence on third-party DNS providers with a
      Pi-hole + Unbound stack that blocks ad and tracker domains and performs full
      recursive resolution against root, TLD, and authoritative servers directly.
    tech: ["Pi-hole", "Unbound", "DNS", "DNSSEC", "Linux", "Privacy"]
    post: "/blog/pi-hole-unbound-home-dns-server/"
    repo: ""

  - title: "Python Email Automation"
    year: "2024"
    summary: >-
      A Python tool that reads a contact list and sends personalised outreach email at
      scale, using an LLM to generate the body copy for each recipient.
    tech: ["Python", "SMTP", "AI/LLM", "Automation"]
    post: "/blog/python-email-automation-project/"
    repo: ""

  - title: "Linux Backdoor — With a SOC Perspective"
    year: "2023"
    summary: >-
      Built a persistent reverse-shell backdoor on a Linux host using named pipes and
      netcat, then flipped sides and analysed exactly what traces a SOC analyst would
      find in the logs.
    tech: ["Linux", "Netcat", "Bash", "Blue Team", "Detection"]
    post: "/blog/linux-backdoor-with-a-soc-perspective/"
    repo: ""

  - title: "Ransomware From Scratch (Educational)"
    year: "2023"
    summary: >-
      A working ransomware proof-of-concept written in Python with Fernet symmetric
      encryption, built and detonated entirely inside a disposable VM to understand how
      the real thing operates.
    tech: ["Python", "Fernet", "Cryptography", "Malware Analysis"]
    post: "/blog/creating-ransomware-with-python-ethically/"
    repo: ""
---

Everything below was built in an isolated lab environment for learning purposes. The
offensive projects exist so I can understand what defenders are looking at — each one
ends with the detection side of the story.
