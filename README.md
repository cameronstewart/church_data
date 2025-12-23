# Church Data Analytics Platform

A comprehensive platform for discovering, analyzing, and visualizing sermon content across churches. Track what's being preached, where, and extract biblical insights from sermon audio and transcripts.

## Vision

Create an intelligent directory of churches that can:
- 📡 **Aggregate** sermon content from RSS feeds and podcast platforms
- 🎙️ **Transcribe** audio sermons (future feature)
- 📖 **Extract** Bible verses, references, and theological themes
- 📊 **Visualize** "what's preached where" across churches and regions
- 🔍 **Analyze** sermon trends, topics, and biblical coverage

## Current Features

### RSS Feed Ingestion ✅
- Parse RSS and iTunes podcast feeds from churches
- Extract sermon metadata:
  - Title and author
  - Summary/description
  - Audio links
  - Publication dates (via GUID)
- Support for multiple RSS formats (standard, iTunes, Podbean)
- Data organization with pandas DataFrames

### Supported Churches (Examples)
Currently testing with Australian churches:
- Balgowlah Anglican Church
- In Church
- All Souls Church
- CCH (Christ Church Hunters Hill)
- Sydney Church of Christ
- Sydney Life Church

## Roadmap

### Phase 1: Data Collection (In Progress)
- [x] RSS feed parser
- [x] Basic metadata extraction
- [ ] Church directory/database
- [ ] Automated feed discovery
- [ ] Data persistence (SQLite/PostgreSQL)

### Phase 2: Audio Processing (Planned)
- [ ] Audio file download and storage
- [ ] Speech-to-text transcription (Whisper, AWS Transcribe, etc.)
- [ ] Transcript cleanup and formatting
- [ ] Speaker diarization

### Phase 3: Content Analysis (Planned)
- [ ] Bible verse detection and extraction
- [ ] Bible reference parsing (e.g., "John 3:16", "Romans 8")
- [ ] Theological theme classification
- [ ] Topic modeling (LDA, BERTopic)
- [ ] Named entity recognition (biblical figures, places)

### Phase 4: Visualization & Insights (Planned)
- [ ] Geographic mapping of churches and sermon content
- [ ] "What's preached where" dashboard
- [ ] Verse coverage heatmaps
- [ ] Theme trends over time
- [ ] Comparative analysis between churches/denominations
- [ ] Bible book/passage frequency analysis

## Getting Started

### Prerequisites
```bash
pip install feedparser pandas beautifulsoup4 pyPodcastParser requests
```

### Quick Start

```python
import feedparser
import pandas as pd

# Define RSS feeds
church_feeds = [
    'http://balgowlahanglican.com.au/feed/podcast/',
    'https://inchurch.com.au/podcast?format=rss'
]

# Parse feeds
feeds = [feedparser.parse(url) for url in church_feeds]

# Extract sermon data
sermons = []
for feed in feeds:
    for entry in feed.entries:
        sermons.append({
            'title': entry.title,
            'link': entry.link,
            'summary': entry.summary,
            'guid': entry.guid
        })

# Create DataFrame
df = pd.DataFrame(sermons)
print(df.head())
```

## Project Structure

```
church_data/
├── README.md                          # This file
├── Import_RSS_Church_Feeds.ipynb      # RSS feed experimentation
└── (future)
    ├── src/
    │   ├── ingestion/                 # Feed parsers and downloaders
    │   ├── transcription/             # Audio-to-text processing
    │   ├── analysis/                  # Bible reference extraction, NLP
    │   └── visualization/             # Charts, maps, dashboards
    ├── data/                          # Stored sermons and metadata
    ├── tests/                         # Unit tests
    └── requirements.txt               # Python dependencies
```

## Technical Architecture (Proposed)

### Data Pipeline
```
RSS Feeds → Parser → Database → Transcription → Analysis → Visualization
                                      ↓
                               Audio Storage
```

### Key Technologies (Suggested)
- **Data Collection**: `feedparser`, `requests`, `beautifulsoup4`
- **Storage**: SQLite (development), PostgreSQL (production)
- **Transcription**: OpenAI Whisper, AssemblyAI, AWS Transcribe
- **NLP**: spaCy, NLTK, transformers (BERT)
- **Bible Reference Parsing**: Regex patterns, bible-parser libraries
- **Visualization**: Plotly, Dash, Streamlit, Folium (maps)
- **Orchestration**: Apache Airflow, Prefect (for scheduled ingestion)

## Use Cases

1. **Researchers**: Study preaching trends across denominations
2. **Church Leaders**: Understand sermon coverage and thematic gaps
3. **Congregants**: Find sermons on specific Bible passages or topics
4. **Theologians**: Analyze doctrinal emphasis across churches
5. **Newcomers**: Discover churches by theological focus

## Data Privacy & Ethics

- Only process publicly available sermon content
- Respect robots.txt and feed usage policies
- Provide attribution to churches and speakers
- Store minimal personal data
- Allow churches to opt-out

## Contributing

Contributions welcome! Areas for help:
- Additional RSS feed formats and parsers
- Bible reference extraction algorithms
- Transcription pipeline development
- Visualization and dashboard creation
- Testing with international churches

## Future Enhancements

- **Multi-language support**: Transcription and analysis in multiple languages
- **Sermon recommendations**: "If you liked this, try..."
- **Cross-referencing**: Link related sermons across churches
- **API**: Programmatic access to sermon database
- **Mobile app**: Search and discover on-the-go
- **Denomination filtering**: Compare theological traditions
- **Historical archive**: Preserve sermon content over time

## License

To be determined (suggest MIT or Apache 2.0 for open-source collaboration)

## Contact

For questions, suggestions, or collaboration opportunities, please open an issue.

---

**Note**: This project is in early experimental stages. The notebook `Import_RSS_Church_Feeds.ipynb` demonstrates initial RSS parsing capabilities.
