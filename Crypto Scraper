import requests
from bs4 import BeautifulSoup
import datetime
import os
import pytz

# ---------------------------------------------------------
# 설정 (Configuration)
# ---------------------------------------------------------
# 검색할 키워드 리스트 (대소문자 무관하게 처리됨)
KEYWORDS = ["Bitcoin", "Ethereum", "Solana", "BTC", "ETH", "SOL"]

# 스크랩 대상 URL (예시: CoinDesk RSS Feed 또는 주요 뉴스 섹션)
# RSS를 사용하는 것이 HTML 구조 변경에 덜 민감하여 유지보수에 유리합니다.
TARGET_URL = "https://www.coindesk.com/arc/outboundfeeds/rss/"

# 옵시디언 저장을 위한 파일 경로 (GitHub 리포지토리 기준 상대 경로)
SAVE_DIR = "ObsidianVault/News/Crypto"

# ---------------------------------------------------------
# 스크랩 로직
# ---------------------------------------------------------
def fetch_crypto_news():
    """
    뉴스 피드를 가져와서 키워드와 관련된 기사만 필터링합니다.
    """
    try:
        response = requests.get(TARGET_URL, timeout=10)
        response.raise_for_status()
    except requests.exceptions.RequestException as e:
        print(f"Error fetching news: {e}")
        return []

    # XML 파싱 (RSS Feed 기준)
    soup = BeautifulSoup(response.content, features="xml")
    items = soup.find_all("item")
    
    filtered_articles = []

    for item in items:
        title = item.title.text.strip() if item.title else "No Title"
        link = item.link.text.strip() if item.link else "#"
        description = item.description.text.strip() if item.description else ""
        pub_date = item.pubDate.text.strip() if item.pubDate else ""

        # 키워드 필터링 (제목이나 설명에 키워드가 포함된 경우)
        text_to_check = (title + " " + description).lower()
        matched_keywords = [k for k in KEYWORDS if k.lower() in text_to_check]

        if matched_keywords:
            filtered_articles.append({
                "title": title,
                "link": link,
                "desc": description[:200] + "..." if len(description) > 200 else description,
                "date": pub_date,
                "keywords": matched_keywords
            })

    return filtered_articles

# ---------------------------------------------------------
# Markdown 생성 로직 (Obsidian 최적화)
# ---------------------------------------------------------
def generate_markdown(articles):
    """
    수집된 기사를 Obsidian 스타일의 Markdown으로 변환합니다.
    """
    # 한국 시간(KST) 기준 현재 시간 설정
    kst = pytz.timezone('Asia/Seoul')
    now = datetime.datetime.now(kst)
    
    date_str = now.strftime("%Y-%m-%d")
    time_str = now.strftime("%H:%M")
    
    # 오전/오후 구분
    period = "Morning" if now.hour < 12 else "Evening"
    filename = f"{date_str}-{period}-Brief.md"
    
    # Obsidian Frontmatter (YAML)
    md_content = f"""---
title: "{date_str} {period} Crypto Briefing"
date: {date_str}
time: {time_str}
tags:
  - news
  - crypto
  - automation
---

# 📅 {date_str} {period} 가상화폐 주요 뉴스

> **자동화 봇 메시지**: 현재 시각 {time_str} 기준, **{', '.join(KEYWORDS)}** 관련 주요 기사를 요약했습니다.

---

"""

    if not articles:
        md_content += "\n### 🚫 관련된 새로운 기사가 없습니다.\n"
    else:
        for idx, article in enumerate(articles, 1):
            # Obsidian Callout 기능을 활용한 스타일링
            md_content += f"## {idx}. {article['title']}\n\n"
            md_content += f"- **관련 키워드**: #{' #'.join(article['keywords'])}\n"
            md_content += f"- **발행일**: {article['date']}\n"
            md_content += f"- **요약**: {article['desc']}\n\n"
            md_content += f"> [🔗 원문 기사 바로가기]({article['link']})\n\n"
            md_content += "---\n"

    return filename, md_content

# ---------------------------------------------------------
# 실행 및 파일 저장
# ---------------------------------------------------------
def main():
    print("🔍 뉴스 스크랩 시작...")
    articles = fetch_crypto_news()
    print(f"✅ {len(articles)}개의 관련 기사 발견.")

    filename, content = generate_markdown(articles)

    # 디렉토리가 없으면 생성
    if not os.path.exists(SAVE_DIR):
        os.makedirs(SAVE_DIR)
        print(f"📂 디렉토리 생성: {SAVE_DIR}")

    file_path = os.path.join(SAVE_DIR, filename)

    with open(file_path, "w", encoding="utf-8") as f:
        f.write(content)
    
    print(f"💾 파일 저장 완료: {file_path}")

if __name__ == "__main__":
    main()
