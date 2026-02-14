# AI Job Source Agent 🤖

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

> Intelligent pipeline that extracts company career pages and job posting URLs from LinkedIn job listings using a cost-optimized 3-tier discovery strategy.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Usage](#usage)
- [Cost Breakdown](#cost-breakdown)
- [Project Structure](#project-structure)
- [Documentation](#documentation)
- [Development](#development)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## 🎯 Overview

The AI Job Source Agent automates the tedious process of finding company career pages and open positions from LinkedIn job postings. Instead of manually clicking through dozens of company websites, this agent does it for you in minutes.

**Input:** LinkedIn job search URL  
**Output:** JSON file with company names, career page URLs, and job posting URLs  
**Budget:** Under $50/month  
**Processing:** 20-50 companies per run in ~10 minutes

### How It Works

```
LinkedIn Jobs → Company Websites → Career Pages → Job Postings → JSON Output
```

The agent uses a smart 3-tier strategy to find career pages:
1. **Tier 1 (FREE):** Try common paths like `/careers`, `/jobs` → 80% success rate
2. **Tier 2 (FREE):** Scrape homepage for career links → 15% success rate  
3. **Tier 3 (PAID):** Use Claude AI as fallback → 5% success rate

This keeps costs low while maintaining high accuracy.

---

## ✨ Features

### Core Capabilities
- 🔍 **LinkedIn Integration** - Fetch job listings via third-party APIs (Apify/RapidAPI)
- 🎯 **3-Tier Career Page Discovery** - Cost-optimized heuristics → scraping → AI fallback
- 🤖 **AI-Powered Fallback** - Claude API for difficult cases (only when needed)
- 🌐 **JavaScript Rendering** - Playwright for dynamic career pages
- 📊 **Detailed Statistics** - Success rates, API usage, processing times
- 💾 **JSON Output** - Structured, timestamped results

### Smart Features
- ✅ **URL Validation** - Ensures all URLs are valid and accessible
- 🔄 **Graceful Error Handling** - Continues on failures, logs everything
- 💰 **Cost Tracking** - Monitors API usage to stay within budget
- 📈 **Real-time Progress** - See what the agent is doing as it runs
- ⚡ **Optimized Performance** - Processes 50 companies in ~10 minutes

### Budget-Conscious Design
- 🎯 **Free-first approach** - Uses free methods for 95% of cases
- 💵 **Cost monitoring** - Tracks spending to stay under $50/month
- 🚫 **Hard limits** - Prevents runaway API costs (max 50 Claude calls/month)
- 📉 **Efficient batching** - Minimizes API calls through smart caching

---

## 🏗️ Architecture

### High-Level Design

```
┌─────────────────────────────────────────────────────┐
│              JobSourcePipeline (Orchestrator)        │
└─────────────────────────────────────────────────────┘
            ↓               ↓               ↓
    ┌───────────┐   ┌──────────────┐   ┌─────────────┐
    │ LinkedIn  │→→→│Career Finder │→→→│  Position   │
    │  Fetcher  │   │(Tier 1,2,3)  │   │  Extractor  │
    └───────────┘   └──────────────┘   └─────────────┘
         ↓                  ↓                   ↓
    ┌────────────────────────────────────────────────┐
    │          JSON Output + Statistics              │
    └────────────────────────────────────────────────┘
```

### Components

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **LinkedInFetcher** | Apify/RapidAPI | Fetch job listings and company data |
| **CareerPageFinder** | BeautifulSoup4 | Discover career pages (Tier 1 & 2) |
| **ClaudeFallback** | Claude API | AI-powered fallback (Tier 3) |
| **PositionExtractor** | Playwright | Extract job URLs from career pages |
| **OutputManager** | JSON | Format and save results |

See [Design Documentation](#documentation) for detailed UML diagrams.

---

## 📦 Installation

### Prerequisites

- Python 3.10 or higher
- pip package manager
- API keys (see [Configuration](#configuration))

### Step 1: Clone Repository

```bash
git clone https://github.com/yourusername/ai-job-source-agent.git
cd ai-job-source-agent
```

### Step 2: Install Dependencies

```bash
pip install -r requirements.txt
```

**requirements.txt:**
```txt
apify-client>=1.7.0
anthropic>=0.25.0
beautifulsoup4>=4.12.0
playwright>=1.40.0
requests>=2.31.0
python-dotenv>=1.0.0
```

### Step 3: Install Playwright Browser

```bash
playwright install chromium
```

### Step 4: Set Up Environment Variables

Create a `.env` file in the project root:

```bash
# API Keys
APIFY_TOKEN=your_apify_token_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Optional: Customize paths and limits
OUTPUT_DIR=./output
MAX_CLAUDE_CALLS_PER_MONTH=50
REQUEST_TIMEOUT=5
BROWSER_TIMEOUT=15000
```

### Getting API Keys

**Apify (LinkedIn Data):**
1. Sign up at [apify.com](https://apify.com)
2. Navigate to Settings → Integrations → API tokens
3. Create new token and copy

**Anthropic (Claude AI):**
1. Sign up at [console.anthropic.com](https://console.anthropic.com)
2. Navigate to API Keys
3. Create new key and copy

---

## 🚀 Quick Start

### Basic Usage

```bash
python pipeline.py --linkedin-url "https://linkedin.com/jobs/search?keywords=software+engineer" --max 50
```

### Example with Location

```bash
python pipeline.py \
  --linkedin-url "https://linkedin.com/jobs/search?keywords=python+developer&location=San+Francisco" \
  --max 30
```

### Output

```
📊 Fetching LinkedIn data...
[1/30] Processing: Stripe
  ✓ Found via direct path: https://stripe.com/jobs
  ✓ Success!
[2/30] Processing: OpenAI
  ✓ Found via direct path: https://openai.com/careers
  ✓ Success!
[3/30] Processing: Anthropic
  ⚡ Using Claude fallback...
  ✓ Success!
...
💾 Saved 28 results to job_sources_2025-02-13.json

📈 Stats:
  Total processed: 30
  Heuristic success: 27 (90.0%)
  Claude fallback used: 3
  Failed: 2
```

### Result Format

**job_sources_2025-02-13.json:**
```json
{
  "results": [
    {
      "company_name": "Stripe",
      "career_page_url": "https://stripe.com/jobs",
      "open_position_url": "https://stripe.com/jobs/listing/senior-engineer/12345",
      "timestamp": "2025-02-13T10:30:00"
    },
    {
      "company_name": "OpenAI",
      "career_page_url": "https://openai.com/careers",
      "open_position_url": "https://openai.com/careers/software-engineer",
      "timestamp": "2025-02-13T10:32:15"
    }
  ],
  "stats": {
    "total_processed": 30,
    "successful": 28,
    "failed": 2,
    "tier1_success": 24,
    "tier2_success": 3,
    "tier3_success": 1,
    "claude_api_calls": 3,
    "linkedin_api_calls": 1,
    "total_processing_time": 287.5
  }
}
```

---

## ⚙️ Configuration

### config.py

Customize the agent's behavior by editing `config.py`:

```python
import os
from dotenv import load_env

load_env()

# API Configuration
APIFY_TOKEN = os.getenv('APIFY_TOKEN')
ANTHROPIC_API_KEY = os.getenv('ANTHROPIC_API_KEY')

# Career Page Patterns (Tier 1)
CAREER_PATHS = [
    '/careers', '/jobs', '/careers/', '/jobs/',
    '/about/careers', '/company/careers',
    '/work-with-us', '/join-us', '/opportunities',
    '/careers-at', '/employment', '/career',
    '/en/careers', '/us/careers'
]

# Career Keywords (Tier 2)
CAREER_KEYWORDS = [
    'careers', 'jobs', 'opportunities', 'work with us',
    'join our team', 'we\'re hiring', 'open positions',
    'join us', 'employment'
]

# Limits & Timeouts
MAX_CLAUDE_CALLS_PER_MONTH = 50
REQUEST_TIMEOUT = 5  # seconds
BROWSER_TIMEOUT = 15000  # milliseconds
MAX_RETRIES = 3

# Output
OUTPUT_DIR = './output'
LOG_DIR = './logs'
```

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `APIFY_TOKEN` | ✅ | - | Apify API token for LinkedIn data |
| `ANTHROPIC_API_KEY` | ✅ | - | Anthropic API key for Claude |
| `OUTPUT_DIR` | ❌ | `./output` | Directory for JSON results |
| `MAX_CLAUDE_CALLS_PER_MONTH` | ❌ | `50` | Monthly Claude API limit |
| `REQUEST_TIMEOUT` | ❌ | `5` | HTTP request timeout (seconds) |
| `BROWSER_TIMEOUT` | ❌ | `15000` | Browser timeout (milliseconds) |

---

## 💻 Usage

### Command Line Interface

```bash
python pipeline.py [OPTIONS]
```

**Options:**

| Flag | Required | Description | Example |
|------|----------|-------------|---------|
| `--linkedin-url` | ✅ | LinkedIn job search URL | `"https://linkedin.com/jobs/search?keywords=engineer"` |
| `--max` | ✅ | Max companies to process | `50` |
| `--output` | ❌ | Custom output filename | `my_jobs.json` |
| `--verbose` | ❌ | Enable verbose logging | - |
| `--dry-run` | ❌ | Test without API calls | - |

### Python API

```python
from pipeline import JobSourcePipeline

# Initialize pipeline
pipeline = JobSourcePipeline()

# Run with custom settings
results = pipeline.run(
    linkedin_url="https://linkedin.com/jobs/search?keywords=python",
    max_companies=25
)

# Access results
for result in results:
    print(f"{result.company_name}: {result.career_page_url}")

# Get statistics
stats = pipeline.statistics
print(f"Success rate: {stats.calculate_success_rate():.1f}%")
print(f"Claude calls used: {stats.claude_api_calls}/50")
```

### Advanced Usage

**Custom Career Paths:**
```python
from career_finder import CareerPageFinder

finder = CareerPageFinder()
finder.common_paths.append('/join-our-team')
finder.common_paths.append('/hiring')

career_url = finder.find_career_page("https://example.com")
```

**Disable Claude Fallback (Free-only):**
```python
pipeline = JobSourcePipeline()
pipeline.claude_fallback = None  # Disable Tier 3
results = pipeline.run(linkedin_url, max_companies=50)
```

---

## 💰 Cost Breakdown

### Monthly Budget: $50

| Service | Cost | Usage | Notes |
|---------|------|-------|-------|
| **LinkedIn API** (Apify) | $20-25 | ~1000 job listings | Pay per use |
| **Claude API** | $15-20 | Max 50 calls | ~$0.30-0.40/call |
| **Infrastructure** | $0 | - | Local execution, free tools |
| **Buffer** | $5-10 | - | Safety margin |

### Cost Optimization Tips

1. **Maximize Tier 1 & 2 Success** - Add common career paths to `CAREER_PATHS`
2. **Batch Processing** - Run weekly instead of daily to reduce API calls
3. **Cache Results** - Store results and skip re-checking same companies
4. **Monitor Usage** - Check `stats.claude_api_calls` to stay within limits

### Expected Processing Capacity

With $50/month budget:
- **Companies processed:** 200-300/month
- **LinkedIn API calls:** ~10-15/month (batched)
- **Claude API calls:** Max 50/month (5% fallback rate)
- **Success rate:** 85-90%

---

## 📁 Project Structure

```
ai-job-source-agent/
├── README.md                           # This file
├── requirements.txt                     # Python dependencies
├── .env.example                        # Environment variables template
├── config.py                           # Configuration settings
│
├── docs/                               # Documentation
│   ├── SRS_AI_Job_Source_Agent.md     # Software Requirements Spec
│   ├── DESIGN_Step1_ClassIdentification.md
│   ├── DESIGN_Step2_UML_Diagrams.md   # UML diagrams (PlantUML)
│   └── DESIGN_Step2_UML_Mermaid.md    # UML diagrams (Mermaid)
│
├── src/                                # Source code
│   ├── linkedin_fetcher.py            # LinkedIn API integration
│   ├── career_finder.py               # Career page discovery (Tier 1-2)
│   ├── claude_fallback.py             # Claude AI fallback (Tier 3)
│   ├── position_extractor.py          # Job position extraction
│   ├── output_manager.py              # JSON output management
│   ├── pipeline.py                    # Main orchestrator
│   │
│   ├── models/                        # Data models
│   │   ├── company_data.py
│   │   ├── job_source_result.py
│   │   └── execution_statistics.py
│   │
│   └── utils/                         # Utilities
│       ├── configuration.py
│       ├── logger.py
│       └── url_validator.py
│
├── tests/                              # Unit tests
│   ├── test_linkedin_fetcher.py
│   ├── test_career_finder.py
│   ├── test_claude_fallback.py
│   ├── test_position_extractor.py
│   └── test_pipeline.py
│
├── output/                             # Generated JSON files
│   └── job_sources_YYYY-MM-DD.json
│
└── logs/                               # Application logs
    └── pipeline_YYYY-MM-DD.log
```

---

## 📚 Documentation

### Available Documents

| Document | Description |
|----------|-------------|
| [SRS](docs/SRS_AI_Job_Source_Agent.md) | Software Requirements Specification |
| [Class Identification](docs/DESIGN_Step1_ClassIdentification.md) | Design phase - identified classes |
| [UML Diagrams (PlantUML)](docs/DESIGN_Step2_UML_Diagrams.md) | Complete UML diagrams |
| [UML Diagrams (Mermaid)](docs/DESIGN_Step2_UML_Mermaid.md) | GitHub-renderable diagrams |

### Key Design Decisions

**Why 3-Tier Strategy?**
- Most career pages use standard patterns (80%)
- Homepage scraping catches non-standard cases (15%)
- Claude AI for truly difficult cases (5%)
- Keeps costs low while maintaining high success rate

**Why Playwright vs BeautifulSoup?**
- Many career pages are JavaScript-heavy
- Playwright renders JS, BeautifulSoup doesn't
- Small performance trade-off for reliability

**Why Apify vs Direct LinkedIn Scraping?**
- Direct scraping violates LinkedIn ToS
- Apify provides legal, maintained API
- Pay-per-use pricing fits budget

---

## 🛠️ Development

### Running Tests

```bash
# Install dev dependencies
pip install pytest pytest-cov pytest-mock

# Run all tests
pytest

# Run with coverage
pytest --cov=src tests/

# Run specific test file
pytest tests/test_career_finder.py
```

### Code Style

This project uses:
- **Black** for code formatting
- **Pylint** for linting
- **mypy** for type checking

```bash
# Format code
black src/

# Lint code
pylint src/

# Type check
mypy src/
```

### Adding New Career Paths

1. Edit `config.py` and add to `CAREER_PATHS`:
```python
CAREER_PATHS = [
    '/careers',
    '/jobs',
    '/your-new-path',  # Add here
]
```

2. Test on a few companies
3. Monitor Tier 1 success rate increase

### Extending the Pipeline

**Add a new data source:**
```python
# 1. Create new fetcher class
class IndeedFetcher:
    def fetch_job_listings(self, url):
        # Implementation
        pass

# 2. Add to pipeline
pipeline = JobSourcePipeline()
pipeline.indeed_fetcher = IndeedFetcher()
```

**Add custom output formats:**
```python
# In output_manager.py
def save_to_csv(self, filename):
    import csv
    with open(filename, 'w') as f:
        writer = csv.DictWriter(f, fieldnames=['company_name', 'career_page_url'])
        writer.writeheader()
        for result in self.results:
            writer.writerow(result.to_dict())
```

---

## 🐛 Troubleshooting

### Common Issues

**1. "APIFY_TOKEN not found"**
```bash
# Solution: Create .env file
cp .env.example .env
# Then edit .env and add your API keys
```

**2. "Playwright browser not installed"**
```bash
# Solution: Install chromium
playwright install chromium
```

**3. "Rate limit exceeded"**
```bash
# Solution: Wait or reduce batch size
python pipeline.py --max 10  # Process fewer companies
```

**4. "No career pages found"**
```bash
# Solution: Enable verbose logging to debug
python pipeline.py --verbose --linkedin-url "..." --max 5
# Check logs/pipeline_YYYY-MM-DD.log
```

**5. "Claude API limit reached"**
```
# Solution: Wait until next month or disable Tier 3
# Edit pipeline.py:
pipeline.claude_fallback = None
```

### Debug Mode

```python
# Enable debug logging
from logger import Logger
logger = Logger(log_level='DEBUG')

# Enable dry-run (no API calls)
python pipeline.py --dry-run --linkedin-url "..." --max 5
```

### Getting Help

1. Check [existing issues](https://github.com/yourusername/ai-job-source-agent/issues)
2. Review documentation in `docs/`
3. Enable verbose logging: `--verbose`
4. Create new issue with:
   - Error message
   - Steps to reproduce
   - Relevant log excerpt

---

## 🤝 Contributing

Contributions are welcome! Please follow these guidelines:

### How to Contribute

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/amazing-feature
   ```
3. **Make your changes**
   - Write tests for new features
   - Update documentation
   - Follow code style guidelines
4. **Commit your changes**
   ```bash
   git commit -m "Add amazing feature"
   ```
5. **Push to your fork**
   ```bash
   git push origin feature/amazing-feature
   ```
6. **Open a Pull Request**

### Development Setup

```bash
# Clone your fork
git clone https://github.com/yourusername/ai-job-source-agent.git

# Install dev dependencies
pip install -r requirements-dev.txt

# Run tests before committing
pytest
```

### Coding Standards

- Follow PEP 8 style guide
- Write docstrings for all functions/classes
- Add type hints where possible
- Keep functions under 50 lines
- Test coverage > 80%

### Areas for Contribution

- 🌐 Additional career page patterns
- 🔌 New data source integrations (Indeed, Glassdoor)
- 📊 Enhanced statistics and reporting
- 🎨 Web UI/dashboard
- 🐳 Docker containerization
- 📧 Email notifications
- 💾 Database persistence

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

```
MIT License

Copyright (c) 2025 [Your Name]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🙏 Acknowledgments

- **Anthropic** - Claude API for intelligent fallback
- **Apify** - Reliable LinkedIn data access
- **Playwright** - Modern browser automation
- **BeautifulSoup** - HTML parsing made simple

---

## 📞 Contact

**Project Maintainer:** [Your Name]  
**Email:** your.email@example.com  
**GitHub:** [@yourusername](https://github.com/yourusername)

---

## 🗺️ Roadmap

### Version 1.0 (Current)
- ✅ LinkedIn integration
- ✅ 3-tier career page discovery
- ✅ Claude AI fallback
- ✅ JSON output

### Version 1.1 (Planned)
- 🔲 CSV/Excel output formats
- 🔲 Email notifications
- 🔲 Docker support
- 🔲 Enhanced error recovery

### Version 2.0 (Future)
- 🔲 Web UI/dashboard
- 🔲 Indeed integration
- 🔲 Database persistence
- 🔲 Scheduled automated runs
- 🔲 Multi-language support

---

## ⭐ Star History

If you find this project useful, please consider giving it a star! ⭐

---

**Built with ❤️ and Claude Code**
