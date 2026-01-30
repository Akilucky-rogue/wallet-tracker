# Wallet Tracker - Architecture Diagram & System Design

## Overview

This document describes the system architecture of the Wallet Tracker application, including component relationships, data flow, and technology stack.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     USER DEVICES                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Web Browser │  │ Mobile App   │  │ Desktop App  │          │
│  │  (Chrome,    │  │ (iOS,        │  │ (Electron)   │          │
│  │   Firefox)   │  │  Android)    │  │              │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
└─────────┼──────────────────┼──────────────────┼──────────────────┘
          │                  │                  