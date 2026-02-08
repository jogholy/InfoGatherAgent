InfoGather Agent

An intelligent information aggregation and delivery agent that automatically fetches, filters, ranks, and delivers high-quality news tailored to your interests—directly to your WeChat.

🌟 Overview

In daily life, we often want to stay updated on topics we care about—but manually checking multiple sources is time-consuming, and the quality of content from public accounts (e.g., WeChat Official Accounts) varies widely.  

InfoGather Agent solves this by:
- Automatically crawling relevant information based on your subscriptions  
- Evaluating source credibility and content quality  
- Curating a personalized daily digest  
- Delivering it via WeChat, with full control over your preferences through simple chat commands  

You no longer need to hunt for updates—your agent brings the best information to you.

✨ Features

1. WeChat-Based Interaction
- Receive daily digests directly in your WeChat chat
- Update your subscription topics, delivery frequency, and content preferences via natural-language messages (e.g., “Add AI news”, “Send only on weekdays”, “Reduce tech articles”)

2. Smart Information Gathering
- Daily automated scraping from trusted news sites, blogs, forums, and official sources
- Supports custom topic keywords and categories (e.g., “climate policy”, “LLM research”, “local food festivals”)

3. Credibility & Quality Scoring
- Cross-references information across multiple sources to verify facts
- Assigns a trust score based on source reputation, consistency, and timeliness
- Ranks articles using a composite quality metric (relevance + clarity + depth + originality)

4. Personalized Curation & Delivery
- Selects top-N articles per topic based on your preferences and historical feedback
- Auto-generates clean, readable summaries with consistent formatting
- Includes relevant images or charts when available
- Respects your preferred delivery schedule (e.g., every morning at 8 AM, or only on weekends)

5. Default Settings with Easy Customization
- Comes with sensible defaults:
  - Topics: Tech, World News, Science
  - Frequency: Once daily at 7:30 AM
  - Top 5 articles per topic
- All settings can be adjusted anytime via WeChat

🛠️ How It Works

1. Subscribe: Start a chat with the agent on WeChat and tell it what you’d like to follow.
2. Crawl: Every night, the agent fetches new content matching your topics.
3. Filter & Rank: Applies credibility checks and quality scoring; discards low-value or unverified info.
4. Curate: Compiles a visually appealing digest with headlines, summaries, sources, and media.
5. Deliver: Sends the digest at your chosen time via WeChat message.
6. Adapt: Learns from your feedback (e.g., “skip this topic”, “show more like this”) to improve future deliveries.

🔒 Privacy & Ethics

- Your subscription data and interaction history are stored securely and never shared
- Only publicly available content is scraped—no paywalled or private data
- Sources are always credited in the delivered digest

🚀 Getting Started

Note: This agent requires integration with the WeChat Official Account platform or a personal WeChat bot framework (e.g., using itchat or WeChat Work APIs). Deployment instructions assume technical familiarity.

1. Clone this repository  
      git clone https://github.com/yourname/infogather-agent.git
   cd infogather-agent
   

2. Install dependencies  
      pip install -r requirements.txt
   

3. Configure:
   - WeChat API credentials (wechat_config.yaml)
   - Default topics & schedule (config/default_prefs.json)
   - Trusted source list (data/trusted_sources.txt)

4. Run the scheduler  
      python main.py --mode=daily
   

5. Add the agent’s WeChat account and send help to see available commands!

💬 Example WeChat Commands
Command   Action
subscribe AI, renewable energy   Add new topics

unsubscribe politics   Remove a topic

frequency weekly   Change delivery to once per week

top 3   Show only top 3 articles per topic

pause until Monday   Temporarily stop delivery

feedback too much tech   Reduce tech content in future

📈 Roadmap

- [ ] Support voice-note preferences
- [ ] Multi-user personalization
- [ ] Integration with RSS/email as alternative channels
- [ ] User rating system to further refine ranking

🤝 Contributing

Contributions welcome! Please open an issue or PR for:
- New source parsers
- Better NLP-based quality scoring
- UI improvements for the WeChat message template

📜 License

MIT © 2026 InfoGather Team
Stay informed. Stay effortlessly.
