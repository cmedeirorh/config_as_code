# Ansible Automation Platform - Configuration as Code

This repository contains Ansible Configuration as Code (CaC) for managing Red Hat Ansible Automation Platform (AAP) resources, including Controller, Event-Driven Ansible (EDA), and Gateway settings.

## ⚠️ Security Notice

This repository uses Jinja2 template variables for all sensitive values. **Never commit actual credentials or secrets to this repository.**

All sensitive values must be provided separately via:
- An Ansible Vault encrypted variables file
- A `vars.yml` file (excluded from git via `.gitignore`)
- An external secrets manager

## Repository Structure

