---
title: "Claude Code CLI로 멀티 에이전트 봇 만들기"
date: 2026-06-09
# published: false
---

이번 포스트에서는 멀티 에이전트 패턴을 익히기 위해 GeekNews 새 글을 Discord에 자동 포스팅하는 봇을 만들어봤습니다.
<br><br>

## Overview

[GeekNews](https://news.hada.io) RSS를 주기적으로 확인해서 새 글이 올라오면 Discord 채널에 포스팅하는 봇입니다.<br>
사실 기능 자체는 Python 스크립트 수십 줄로 충분합니다. 멀티 에이전트를 써야 하는 복잡도는 아니지만, 패턴을 한 번 직접 만들어보려는 것을 목적으로 일부러 적용했습니다.

### Really Simple Syndication

RSS는 웹사이트의 새 글을 표준화된 형식으로 제공하는 포맷입니다. 사이트가 피드 URL을 제공하면 클라이언트가 이를 읽어 새 글 목록을 가져올 수 있습니다.<br>
피드는 XML 형식으로 작성되는데, XML은 데이터를 태그로 감싸서 구조화하는 형식으로 HTML과 비슷하게 생겼지만 태그 이름을 목적에 맞게 자유롭게 정의할 수 있습니다.

```xml
<item>
  <title>GeekNews 글 제목</title>
  <link>https://news.hada.io/topic?id=123</link>
  <pubDate>Mon, 09 Jun 2026 10:00:00 +0000</pubDate>
</item>
```

GeekNews도 위처럼 `https://news.hada.io/rss/news` 피드를 제공합니다. 이걸 파싱하는 방식으로 최신 글 목록을 가져올 수 있었습니다.

## Claude Code CLI 설정

`.claude/agents/` 기반의 서브 에이전트 패턴은 Anthropic API를 직접 호출하는 방식으로는 구현할 수 없고, Claude Code가 제공하는 기능입니다.<br>
GitHub Actions 같은 자동화 환경에서 실행하려면 워크플로우에 CLI 설치 단계가 필요합니다.

- Anthropic API:
  - Claude에게 텍스트를 보내고 응답을 받는 방식입니다
  - 코드에서 HTTP 요청으로 직접 호출합니다
- Claude Code:
  - Anthropic이 만든 공식 CLI 도구입니다
  - 대화 외에도 파일 읽기/쓰기, Bash 실행 같은 도구가 내장되어 있어 로컬 환경에 직접 접근할 수 있고 `.claude/agents/`나 `CLAUDE.md` 같은 파일 기반 구조를 지원합니다

```yaml
- name: Install Claude Code
  run: npm install -g @anthropic-ai/claude-code

- name: Run bot
  run: |
    claude -p "GeekNews 새 글 확인해서 포스팅해" \
      --allowedTools "Read,Write,Bash,WebFetch" \
      --permission-mode bypassPermissions
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

- `-p`: 비대화형 실행 모드이며 프롬프트를 인자로 넘겨 한 번 실행하고 종료합니다
- `--allowedTools`: 여기서는 파일 읽기/쓰기(`Read`, `Write`), Bash 실행(`Bash`), 외부 URL 요청(`WebFetch`)만 허용했습니다
- `--permission-mode bypassPermissions`:
  - 도구 실행 시 프롬프트 승인을 건너뜁니다
  - CI처럼 사람이 개입하지 않는 환경에서 필요합니다

## 서브 에이전트 정의

`.claude/agents/` 폴더에 마크다운 파일을 넣으면 서브 에이전트로 등록됩니다.

```markdown
---
name: news-fetcher
description: GeekNews에서 최신 글 목록을 가져올 때 사용. 뉴스 수집 요청 시 호출.
tools: WebFetch
---

https://news.hada.io/rss/news 를 가져와서 파싱한 뒤,
아래 형식의 JSON 배열로 반환해.

[{ "id": "...", "title": "...", "link": "..." }]
```

각 필드의 역할은 다음과 같습니다.

- `name`:
  - 에이전트 식별자입니다
  - 오케스트레이터가 에이전트를 선택할 때는 `description`을 기준으로 판단합니다
- `description`:
  - 오케스트레이터가 에이전트를 선택하는 기준입니다
  - "어떤 상황에서 이 에이전트를 써야 하는가"를 명시하면 안정적으로 동작합니다
- `tools`:
  - 에이전트가 사용할 수 있는 도구를 제한합니다
  - news-fetcher는 WebFetch만, discord-poster는 Bash만 허용하는 식입니다

이 프로젝트에서 정의한 서브 에이전트는 세 개입니다.

- `news-fetcher`: GeekNews RSS 피드를 파싱해서 최신 글 목록을 JSON으로 반환합니다
- `discord-poster`: `curl`로 Discord 웹훅에 POST 요청을 보내 메시지를 포스팅합니다
- `code-reviewer`:
  - 가독성, 보안, 성능 관점에서 코드를 리뷰합니다
  - `Read`, `Glob`, `Grep`만 허용해서 코드를 직접 수정하지 못하도록 제한했습니다
  - 그런데 이 프로젝트에서는 실제 코드를 작성하지 않기 때문에 사용할 일이 없었습니다

## `CLAUDE.md` 작성

`CLAUDE.md`는 프로젝트 루트에 두는 파일로, Claude Code가 세션을 시작할 때마다 자동으로 로드합니다. 프로젝트 배경, 실행 규칙, 주의사항 등을 여기에 써두면 매번 다시 설명하지 않아도 됩니다.<br>
CI는 매 실행마다 새 세션으로 시작하기 때문에 Claude는 이전 실행의 내용을 알지 못합니다. `CLAUDE.md`에 실행 순서를 명시해 어떤 환경에서 실행되든 동일한 규칙을 따르도록 했습니다.

```markdown
## 오케스트레이터 실행 규칙

1. `seen_ids.txt`를 읽어 이미 포스팅한 ID 목록을 확인한다
2. news-fetcher를 호출해 최신 글 목록을 가져온다
3. `seen_ids.txt`에 없는 신규 글만 필터링한다
4. 신규 글이 없으면 종료한다
5. discord-poster를 호출해 신규 글을 포스팅한다
6. 포스팅한 ID를 `seen_ids.txt`에 추가한다
```

## `seen_ids.txt` 커밋

`seen_ids.txt` 파일에 이미 포스팅한 글의 ID를 한 줄씩 기록하도록 했습니다. 실행할 때마다 이 파일을 읽어 신규 글인지 판단하고, 포스팅 후에는 ID를 추가합니다. 중복 포스팅을 막는 장치입니다.<br>
문제는 GitHub Actions가 매 실행마다 리포를 새로 체크아웃한다는 것입니다. 실행 중에 `seen_ids.txt`를 업데이트해도 기록하지 않으면 다음 실행 때 원래대로 돌아오기 때문에, 워크플로우 마지막 단계에 커밋 스텝을 추가해서 해결했습니다.

```yaml
- name: Save seen IDs
  run: |
    git config user.name "github-actions[bot]"
    git config user.email "github-actions[bot]@users.noreply.github.com"
    git add seen_ids.txt
    git diff --staged --quiet || git commit -m "chore: update seen_ids"
    git push
```

- `git config`: Actions 환경에는 git 사용자 정보가 없어서 커밋 전에 설정해야 합니다
- `git diff --staged --quiet || git commit`:
  - 스테이징된 변경사항이 없으면 커밋을 건너뜁니다
  - 없는 경우에도 커밋하면 빈 커밋이 계속 쌓이기 때문입니다

## Refactoring

멀티 에이전트 패턴을 익히는 것이 목적이었지만, 실제로 테스트해보니 이 프로젝트에 LLM이 필요가 없다는 것을 확인했습니다.<br>
RSS 파싱 -> 신규 글 필터링 -> Discord 포스팅의 각 단계가 완전히 결정론적입니다. 입력이 같으면 출력도 항상 같아야 하는 작업이라 LLM의 판단이 필요하지 않았고, 오히려 Claude를 거치면 API 비용이 발생하고 실행 시간도 길어졌습니다.<br>

그래서 `main.py` 하나로 리팩터링했습니다!

```python
HTTP_TOO_MANY_REQUESTS = 429

def fetch_feed():
    response = requests.get(FEED_URL, timeout=5)
    response.raise_for_status()
    return ET.fromstring(response.content)

def post_to_discord(title, link):
    webhook_url = os.environ["DISCORD_WEBHOOK_URL"]
    while True:
        response = requests.post(webhook_url, json={"content": f"[{title}]({link})"}, timeout=5)
        if response.status_code == HTTP_TOO_MANY_REQUESTS:
            try:
                retry_after = float(response.json().get("retry_after", 1))
            except Exception:
                retry_after = 1
            time.sleep(max(retry_after, 0))
            continue
        response.raise_for_status()
        break
```

- RSS 피드 요청과 XML 파싱은 표준 라이브러리(`xml.etree.ElementTree`)와 `requests`로 충분했습니다
- Claude Code CLI 설치 단계와 `ANTHROPIC_API_KEY` 시크릿이 불필요해졌고, 의존성도 `requests` 하나로 줄었습니다
- GitHub Actions 스케줄도 1시간에서 5분으로 줄여 새 글이 올라오는 즉시 포스팅되도록 했습니다

![Discord 포스팅 결과](/assets/img/discord-bot-result.png)

멀티 에이전트는 단계마다 인간의 판단이 필요하거나, 맥락에 따라 도구를 동적으로 선택해야 할 때 진가를 발휘합니다.
이 봇처럼 파이프라인이 고정적이고 각 단계의 로직이 명확하다면, 단순한 스크립트가 더 효율적임을 배울 수 있었습니다.
<br><br>

_출처:<br>
[1] Harvard Berkman Klein Center (["RSS 2.0 Specification"](https://cyber.harvard.edu/rss/rss.html))<br>_
