# Nexus

A cross-club data sharing and analysis platform for AI safety organizations.

## Overview

Nexus enables university clubs to share their programmatic data with other clubs and run AI-powered analyses to answer questions like:
- What's the conversion rate from intro fellowships to specialized programs?
- Which factors correlate with fellowship completion?
- How do different program formats compare?

## Features

- **Data Aggregation**: Securely aggregate Airtable data from multiple clubs
- **AI Query Interface**: Ask natural language questions and get instant analysis
- **Unified Portal**: Browse shared curricula, resources, and best practices from other clubs
- **Curation System**: Community upvoting and trusted recommendations

## Quick Start

See [project-overview.md](project-overveiw.md) for the full strategy and technical architecture options.

## Architecture

The system supports multiple implementation approaches:
1. **Airtable Multi-Source Sync** (simplest MVP)
2. **Airtable API + Custom Script** (most flexible)
3. **ETL to PostgreSQL** (most powerful)
4. **n8n/FlowiseAI Agent** (best AI UX)

## Key Resources

- [Data Collection Framework](docs/) - Schema and guidelines for clubs
- [Technical Options Analysis](project-overveiw.md) - Detailed comparison of implementation paths

## Contributing

This project is open source and welcomes contributions. See our issues for ways to help.

## Sustainability

- Open source and community-driven
- Designed for low maintenance burden
- Delegated co-maintainer model

## License

MIT
