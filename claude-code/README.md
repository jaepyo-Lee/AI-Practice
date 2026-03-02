# Claude Code 완전 가이드

> Anthropic의 공식 CLI 에이전트 — 터미널에서 Claude를 직접 사용하는 방법

## 목차

1. [개요](#개요)
2. [설치 및 초기 설정](#설치-및-초기-설정)
3. [핵심 기능](#핵심-기능)
4. [슬래시 명령어 (Slash Commands)](#슬래시-명령어)
5. [CLAUDE.md 설정](#claudemd-설정)
6. [스킬 시스템 (Skills)](#스킬-시스템)
7. [MCP (Model Context Protocol)](#mcp-model-context-protocol)
8. [훅 시스템 (Hooks)](#훅-시스템)
9. [에이전트 & 서브에이전트](#에이전트--서브에이전트)
10. [헤드리스 & 배치 모드](#헤드리스--배치-모드)
11. [고급 멀티에이전트 패턴](#고급-멀티에이전트-패턴)
12. [커스텀 스킬 만들기 (레거시)](#커스텀-스킬-만들기-레거시)
13. [IDE 통합](#ide-통합)
14. [권한 및 보안 모드](#권한-및-보안-모드)
15. [실제 기업 활용 사례](#실제-기업-활용-사례)
16. [고급 패턴 및 Best Practices](#고급-패턴-및-best-practices)
17. [관련 문서](#관련-문서)

---

## 개요

Claude Code는 Anthropic이 만든 CLI 기반 AI 코딩 에이전트다. 터미널에서 자연어로 명령하면 코드 읽기, 수정, 테스트 실행, Git 조작까지 자율적으로 수행한다.

### 핵심 특징

| 특징 | 설명 |
|------|------|
| **Agentic 실행** | 단순 코드 생성이 아닌 파일 읽기→분석→수정→테스트 전체 루프 자율 처리 |
| **컨텍스트 유지** | 대화 전반에 걸쳐 코드베이스 맥락 유지 |
| **도구 호출** | Bash, 파일 읽기/쓰기, 웹 검색, MCP 서버 등 다양한 도구 활용 |
| **권한 제어** | 실행 전 사용자 승인, 읽기 전용 모드 등 세밀한 제어 |
| **확장성** | 커스텀 스킬, MCP 서버, 훅으로 워크플로우 완전 커스터마이징 |

---

## 설치 및 초기 설정

### 설치

```bash
npm install -g @anthropic-ai/claude-code
```

### 기본 사용법

```bash
# 대화형 모드 시작
claude

# 단일 명령 실행 (non-interactive)
claude -p "이 코드의 버그를 찾아줘"

# 특정 디렉토리에서 시작
claude --cwd /path/to/project

# 모델 지정
claude --model claude-opus-4-6

# 컨텍스트 크기 설정
claude --max-tokens 8192
```

### 설정 파일 위치

```
~/.claude/                    # 전역 설정
  settings.json               # 전역 설정 파일
  commands/                   # 전역 커스텀 스킬
  keybindings.json            # 키보드 단축키

.claude/                      # 프로젝트별 설정 (git 루트)
  settings.json               # 프로젝트 설정
  commands/                   # 프로젝트 커스텀 스킬

CLAUDE.md                     # 프로젝트 AI 지시사항 (자동 로드)
```

### settings.json 주요 옵션

```json
{
  "model": "claude-sonnet-4-6",
  "permissions": {
    "allow": ["Bash(git:*)", "Read", "Write"],
    "deny": ["Bash(rm -rf:*)"]
  },
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...]
  },
  "mcpServers": {
    "filesystem": { "command": "...", "args": [...] }
  }
}
```

---

## 핵심 기능

### 1. 파일 작업

Claude Code는 프로젝트 전체를 이해하고 파일을 조작한다.

```bash
# 파일 구조 이해 후 작업
claude "src/ 폴더의 전체 구조를 분석하고 의존성 다이어그램을 만들어줘"

# 여러 파일 동시 수정
claude "user 관련 컴포넌트들을 모두 찾아서 타입스크립트로 마이그레이션해줘"

# 대규모 리팩토링
claude "클래스 기반 컴포넌트를 함수형으로 전환해줘"
```

### 2. Git 통합

```bash
# 변경사항 리뷰 후 커밋
claude "변경된 파일을 검토하고 적절한 커밋 메시지로 커밋해줘"

# PR 설명 자동 생성
claude "현재 브랜치와 main의 차이를 분석해서 PR 설명을 작성해줘"

# 코드 리뷰
claude "최근 3개 커밋의 변경사항을 리뷰해줘"
```

### 3. 테스트 실행 및 디버깅

```bash
# 테스트 실패 자동 수정
claude "테스트를 실행하고 실패한 테스트를 분석해서 수정해줘"

# 에러 디버깅
claude "이 스택 트레이스를 분석하고 근본 원인을 찾아 수정해줘"
```

### 4. 코드 검색 및 분석

```bash
# 코드베이스 전체 검색
claude "인증 로직이 어디에 구현되어 있는지 찾아서 설명해줘"

# 의존성 분석
claude "이 함수를 호출하는 모든 곳을 찾아줘"

# 보안 감사
claude "SQL injection 취약점이 있는 부분을 모두 찾아줘"
```

### 5. 웹 검색 연동

```bash
# 최신 정보 검색 후 적용
claude "Next.js 15의 새로운 기능들을 찾아보고 우리 프로젝트에 적용 가능한 것들을 알려줘"

# 문서 참조
claude "shadcn/ui Button 컴포넌트 최신 API를 확인하고 우리 코드를 업데이트해줘"
```

---

## 슬래시 명령어

`/` 로 시작하는 내장 명령어들.

### 내장 슬래시 명령어

| 명령어 | 설명 |
|--------|------|
| `/help` | 도움말 표시 |
| `/clear` | 대화 컨텍스트 초기화 |
| `/compact` | 대화를 요약해서 컨텍스트 절약 |
| `/cost` | 현재 세션 토큰 비용 표시 |
| `/doctor` | Claude Code 환경 진단 |
| `/init` | CLAUDE.md 초기화 |
| `/login` | Anthropic 계정 로그인 |
| `/logout` | 로그아웃 |
| `/model` | 사용 모델 변경 |
| `/permissions` | 현재 권한 설정 확인 |
| `/fast` | Fast 모드 토글 (빠른 응답) |
| `/review` | 현재 변경사항 코드 리뷰 |
| `/rename` | 현재 세션에 이름 지정 |
| `/status` | 세션 상태 표시 |
| `/vim` | Vim 편집 모드 토글 |
| `/memory` | 메모리 파일 열기 |

### 커스텀 스킬 호출

```bash
/commit        # 커밋 스킬 (사용자 정의)
/review-pr     # PR 리뷰 스킬
/deploy        # 배포 스킬
/ralph-loop    # 자율 반복 루프 (Ralph 패턴)
```

> v2.1.3부터 `.claude/commands/`와 `.claude/skills/` 모두 슬래시 명령어로 동일하게 동작한다. 기존 commands/ 파일은 그대로 유지되며, 신규 작업에는 skills/ 시스템 권장.

---

## CLAUDE.md 설정

프로젝트 루트의 `CLAUDE.md`는 Claude Code가 항상 먼저 읽는 지시사항 파일이다.

### 기본 구조

```markdown
# 프로젝트 지시사항

## 기술 스택
- Next.js 15, TypeScript, Tailwind CSS
- 패키지 매니저: bun (npm/yarn 절대 사용 금지)
- 테스트: Vitest

## 코드 스타일
- 함수형 컴포넌트만 사용
- 타입은 interface 대신 type 사용
- 파일명: kebab-case

## 금지사항
- console.log 직접 커밋 금지 (logger 사용)
- any 타입 사용 금지
- 테스트 없는 기능 추가 금지

## 자주 사용하는 명령어
- 개발: bun dev
- 테스트: bun test
- 빌드: bun build
```

### 실제 기업 CLAUDE.md 패턴

```markdown
# Company AI Standards

## Security Requirements
- API 키를 코드에 하드코딩 금지
- 모든 외부 입력 검증 필수
- SQL 쿼리는 반드시 파라미터화

## Architecture Decisions
- 상태관리: Zustand (Redux 사용 금지)
- API 호출: React Query + Axios
- 스타일: CSS Modules (Styled Components 금지)

## Review Checklist Before Committing
- [ ] 타입 에러 없음
- [ ] 테스트 커버리지 80% 이상
- [ ] 접근성 (a11y) 고려

## Team Context
- 백엔드 API: /docs/api-spec.md 참조
- 디자인 시스템: Figma 링크 (사내 내부망)
```

### 계층적 CLAUDE.md

```
루트/CLAUDE.md          # 전체 프로젝트 지시사항
루트/frontend/CLAUDE.md  # 프론트엔드 특화 지시사항
루트/backend/CLAUDE.md   # 백엔드 특화 지시사항
```

하위 폴더에서 작업 시 해당 경로의 모든 CLAUDE.md가 계층적으로 로드된다.

---

## 스킬 시스템

v2.1.3부터 도입된 새로운 확장 표준. 기존 `commands/`와 통합되어 더 강력한 기능을 제공한다.

### 디렉토리 구조

```
~/.claude/skills/my-skill/     # 전역 스킬
  SKILL.md                     # 필수: 스킬 정의 파일
  context.md                   # 선택: 추가 컨텍스트
  examples/                    # 선택: 예시 파일

.claude/skills/my-skill/       # 프로젝트 스킬
  SKILL.md
```

### SKILL.md 구조 (YAML 프론트매터 + 마크다운)

```markdown
---
name: code-reviewer
description: 코드 리뷰를 수행하고 품질 피드백 제공
version: 1.0.0
author: team
user-invocable: true          # false면 슬래시 메뉴에서 숨김
context: fork                 # fork = 독립 서브에이전트로 실행
allowed-tools:                # 허용할 도구 제한
  - Read
  - Grep
  - Glob
---

# Code Reviewer Skill

주어진 코드를 다음 기준으로 리뷰해줘:

1. 코드 품질 및 가독성
2. 잠재적 버그
3. 성능 이슈
4. 보안 취약점

각 항목을 라인 번호와 함께 구체적으로 설명해줘.
```

### 프론트매터 주요 옵션

| 옵션 | 설명 |
|------|------|
| `user-invocable: false` | 슬래시 메뉴에서 숨김 (자동 호출 전용) |
| `context: fork` | 독립 서브에이전트로 실행 (메인 대화 격리) |
| `allowed-tools: [Read, Grep]` | 이 스킬 실행 시 허용 도구 제한 |
| `model: claude-opus-4-6` | 이 스킬에 특정 모델 사용 |

### Extended Thinking 활성화

스킬 내용에 `ultrathink` 키워드 포함 시 자동으로 Extended Thinking 활성화.

```markdown
---
name: architecture-planner
description: 복잡한 시스템 아키텍처 설계
context: fork
---

ultrathink를 사용해서 다음 시스템의 아키텍처를 깊이 분석하고 설계해줘:

$ARGUMENTS
```

### 플러그인 (Plugin)

스킬 + 서브에이전트 + 명령어를 하나의 패키지로 묶은 배포 단위.

```
my-plugin/
  SKILL.md          # 메인 스킬
  agents/           # 전용 서브에이전트 정의
  commands/         # 플러그인 슬래시 명령어
  README.md
```

설치:
```bash
# npm 패키지로 배포된 플러그인
npm install -g @company/claude-plugin-devtools

# 로컬 설치
ln -s /path/to/my-plugin ~/.claude/plugins/my-plugin
```

---

## MCP (Model Context Protocol)

MCP는 Claude가 외부 도구, 데이터 소스와 표준화된 방식으로 통신하는 프로토콜이다.

### MCP 아키텍처

```
Claude Code (Host)
    ↕ MCP Protocol (JSON-RPC over stdio/HTTP)
MCP Server
    ↕
외부 시스템 (DB, API, 파일시스템, etc.)
```

### MCP 서버 설정 (settings.json)

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_..." }
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": { "SLACK_BOT_TOKEN": "xoxb-..." }
    },
    "custom-api": {
      "command": "node",
      "args": ["./mcp-server/index.js"],
      "env": { "API_BASE_URL": "https://internal-api.company.com" }
    }
  }
}
```

### 주요 MCP 서버 목록

| 서버 | 기능 | 활용 예 |
|------|------|---------|
| `server-filesystem` | 파일 시스템 접근 | 프로젝트 외부 파일 읽기/쓰기 |
| `server-postgres` | PostgreSQL 직접 쿼리 | DB 스키마 분석, 마이그레이션 |
| `server-github` | GitHub API | PR 생성, 이슈 관리 |
| `server-slack` | Slack 메시지 | 배포 알림, 코드 리뷰 알림 |
| `server-google-maps` | 지도 API | 위치 기반 앱 개발 |
| `server-brave-search` | 웹 검색 | 최신 문서 검색 |
| `server-puppeteer` | 브라우저 자동화 | E2E 테스트, 스크래핑 |
| `server-memory` | 영구 메모리 | 세션 간 컨텍스트 유지 |

### 커스텀 MCP 서버 만들기

```typescript
// mcp-server/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { ListToolsRequestSchema, CallToolRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "company-internal-api", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// 도구 목록 등록
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: "get_user_data",
    description: "내부 사용자 DB에서 유저 정보 조회",
    inputSchema: {
      type: "object",
      properties: {
        userId: { type: "string", description: "사용자 ID" }
      },
      required: ["userId"]
    }
  }]
}));

// 도구 실행 핸들러
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "get_user_data") {
    const { userId } = request.params.arguments as { userId: string };
    const userData = await fetchFromInternalDB(userId);
    return { content: [{ type: "text", text: JSON.stringify(userData) }] };
  }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

---

## 훅 시스템

훅은 Claude Code의 도구 실행 전후에 쉘 명령을 자동 실행하는 시스템이다.

### 훅 이벤트 종류

| 이벤트 | 실행 시점 |
|--------|----------|
| `PreToolUse` | 도구 실행 전 |
| `PostToolUse` | 도구 실행 후 |
| `Notification` | Claude가 알림 전송 시 |
| `Stop` | 응답 생성 완료 후 |
| `SubagentStop` | 서브에이전트 완료 후 |

### 훅 설정 예시

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [{
          "type": "command",
          "command": "echo '[LOG] Bash 실행: ' && cat"
        }]
      },
      {
        "matcher": "Write",
        "hooks": [{
          "type": "command",
          "command": "node scripts/validate-file.js"
        }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [{
          "type": "command",
          "command": "npx eslint --fix ${file_path} 2>/dev/null || true"
        }]
      }
    ],
    "Stop": [
      {
        "hooks": [{
          "type": "command",
          "command": "osascript -e 'display notification \"Claude 작업 완료\" with title \"Claude Code\"'"
        }]
      }
    ]
  }
}
```

### 실용적인 훅 활용 패턴

```json
// 파일 저장 시 자동 포맷팅
{
  "PostToolUse": [{
    "matcher": "Write|Edit",
    "hooks": [{
      "type": "command",
      "command": "prettier --write \"${file_path}\" 2>/dev/null || true"
    }]
  }]
}

// Bash 실행 전 위험 명령어 차단
{
  "PreToolUse": [{
    "matcher": "Bash",
    "hooks": [{
      "type": "command",
      "command": "echo $CLAUDE_TOOL_INPUT | grep -qE '(rm -rf|DROP TABLE|force push)' && echo 'BLOCKED' && exit 2 || exit 0"
    }]
  }]
}

// 작업 완료 시 Slack 알림
{
  "Stop": [{
    "hooks": [{
      "type": "command",
      "command": "curl -X POST $SLACK_WEBHOOK -d '{\"text\": \"Claude Code 작업이 완료되었습니다\"}'"
    }]
  }]
}
```

---

## 에이전트 & 서브에이전트

Claude Code는 복잡한 작업을 여러 에이전트로 분산 처리할 수 있다.

### 서브에이전트 개념

```
Main Agent (Claude Code)
├── Explore Agent    # 코드베이스 탐색 전문
├── Plan Agent       # 구현 계획 수립 전문
├── General Agent    # 범용 작업 처리
└── Custom Agent     # 사용자 정의 에이전트
```

### 에이전트 SDK 사용 (Python)

```python
import anthropic
from anthropic.types.beta.messages import BetaContentBlockParam

client = anthropic.Anthropic()

# 도구 정의
tools = [
    {
        "name": "read_file",
        "description": "파일 내용 읽기",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"}
            },
            "required": ["path"]
        }
    },
    {
        "name": "write_file",
        "description": "파일에 내용 쓰기",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"},
                "content": {"type": "string"}
            },
            "required": ["path", "content"]
        }
    }
]

def run_agent(task: str):
    messages = [{"role": "user", "content": task}]

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )

        if response.stop_reason == "end_turn":
            return response.content[-1].text

        if response.stop_reason == "tool_use":
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })

            messages.append({"role": "assistant", "content": response.content})
            messages.append({"role": "user", "content": tool_results})
```

### 병렬 에이전트 패턴

```python
import asyncio

async def parallel_agents(tasks: list[str]):
    """여러 작업을 병렬로 처리"""
    results = await asyncio.gather(*[
        run_agent_async(task) for task in tasks
    ])
    return results

# 활용 예
tasks = [
    "frontend/ 폴더의 보안 취약점 분석",
    "backend/ 폴더의 성능 병목 찾기",
    "database/ 스키마의 최적화 포인트 제안"
]
results = asyncio.run(parallel_agents(tasks))
```

### 서브에이전트 최신 기능 (2025~)

**Worktree 격리 실행:**
```python
# isolation: 'worktree'로 임시 git worktree에서 안전하게 실행
{
  "type": "agent",
  "isolation": "worktree",    # 독립된 브랜치에서 작업
  "background": True,          # 백그라운드 실행
  "task": "대규모 리팩토링 작업"
}
```

**Stop/SubagentStop 훅의 새 필드:**
```json
{
  "hooks": {
    "SubagentStop": [{
      "hooks": [{
        "type": "command",
        "command": "node scripts/collect-results.js"
      }]
    }]
  }
}
```
```javascript
// SubagentStop 훅에서 사용 가능한 새 필드
const input = JSON.parse(process.env.CLAUDE_STOP_INPUT);
const agentId = input.agent_id;                        // 에이전트 식별자
const transcriptPath = input.agent_transcript_path;    // 전체 대화 로그
const lastMessage = input.last_assistant_message;      // 마지막 응답 텍스트
```

**동적 모델 선택:**
```bash
# 메인 에이전트가 작업 복잡도에 따라 서브에이전트 모델 자동 선택
claude "복잡한 설계 작업은 Opus로, 단순 작업은 Haiku로 서브에이전트를 실행해줘"
```

---

## 헤드리스 & 배치 모드

대화형 UI 없이 CLI 또는 스크립트에서 Claude Code를 자동화하는 방법.

### 기본 헤드리스 실행

```bash
# -p 플래그: 단일 프롬프트 실행 후 종료
claude -p "이 코드의 버그를 찾아줘"

# JSON 출력 형식
claude -p "분석 결과를 JSON으로 출력해줘" --output-format json

# 특정 파일을 컨텍스트로 제공
cat error.log | claude -p "이 에러 로그를 분석해줘"

# 권한 자동 승인 (CI 환경)
claude --dangerously-skip-permissions -p "테스트 실행 후 실패 원인 수정해줘"
```

### 멀티턴 세션 (배치 작업)

```bash
# 세션 이름 지정
claude -p "프로젝트 분석 시작해줘" --session-id my-analysis-session

# 이전 세션 이어서 진행
claude -p "방금 분석 결과를 바탕으로 리팩토링 계획 세워줘" \
  --resume my-analysis-session
```

### 병렬 배치 처리 (Bash 스크립트)

```bash
#!/bin/bash
# 여러 파일을 병렬로 분석

analyze_file() {
  local file=$1
  claude -p "이 파일의 코드 품질을 평가해줘: $(cat $file)" \
    --output-format json > "results/${file##*/}.json"
}

export -f analyze_file

# GNU parallel로 병렬 실행
find src -name "*.ts" | parallel -j 4 analyze_file {}
```

### Git Worktree + 병렬 에이전트

```bash
# 각 에이전트가 독립된 워크트리에서 작업
git worktree add .worktrees/agent-1 -b feature/agent-1
git worktree add .worktrees/agent-2 -b feature/agent-2
git worktree add .worktrees/agent-3 -b feature/agent-3

# 병렬 실행
claude -p "frontend 리팩토링" --cwd .worktrees/agent-1 &
claude -p "backend API 개선" --cwd .worktrees/agent-2 &
claude -p "테스트 커버리지 확대" --cwd .worktrees/agent-3 &
wait
```

### CI/CD 파이프라인 통합

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review
on: [pull_request]
jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm install -g @anthropic-ai/claude-code
      - run: |
          claude --dangerously-skip-permissions \
            --output-format json \
            -p "이 PR의 보안 취약점을 검사하고 JSON 리포트를 출력해줘" \
            > security-report.json
      - name: Upload Report
        uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: security-report.json
```

---

## 고급 멀티에이전트 패턴

### 1. Ralph Wiggum Loop (자율 반복 루프)

**개념:** Stop 훅을 이용해 Claude가 종료를 시도할 때 다시 시작 프롬프트를 주입. 완료 조건을 만족할 때까지 자율 반복.

```
[시작 프롬프트]
     ↓
[Claude 실행]
     ↓
[완료 조건 확인]
  성공 → 종료
  실패 → Stop 훅이 프롬프트 재주입 → [Claude 실행]
```

**Stop 훅 설정:**

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "node scripts/ralph-check.js"
      }]
    }]
  }
}
```

```javascript
// scripts/ralph-check.js
const transcript = JSON.parse(process.env.CLAUDE_STOP_INPUT);
const lastMessage = transcript.last_assistant_message;

// 완료 조건 체크
const isComplete = checkCompletionCriteria();

if (!isComplete && currentIteration < MAX_ITERATIONS) {
  // exit code 2 = Claude를 다시 시작
  process.stdout.write(RETRY_PROMPT);
  process.exit(2);
} else {
  process.exit(0);
}
```

**Ralph 스킬 (`ralph-loop.md`):**

```markdown
---
name: ralph-loop
description: 완료 조건을 만족할 때까지 자율 반복 실행
user-invocable: true
---

다음 작업을 완료 조건을 만족할 때까지 반복 수행해줘:

작업: $ARGUMENTS

완료 조건:
1. 모든 테스트가 통과할 것
2. TypeScript 타입 에러가 없을 것
3. ESLint 경고가 없을 것

각 반복마다:
- git commit으로 진행상황 저장
- 다음 단계 계획 수립
- 실패한 부분 분석 및 수정

최대 20회 반복 허용.
```

**활용 예:**

```bash
/ralph-loop "Jest 테스트를 Vitest로 전체 마이그레이션"
/ralph-loop "모든 any 타입을 명시적 타입으로 교체"
/ralph-loop "API 엔드포인트에 대한 테스트 커버리지 80% 달성"
```

---

### 2. Agent Teams (역할 분담 멀티에이전트)

**개념:** 각각 전문 역할을 가진 에이전트들이 병렬 또는 순차로 협력.

```
오케스트레이터 (Main Claude)
├── Coder Agent      # 구현 전담
├── Reviewer Agent   # 코드 리뷰 전담
└── Tester Agent     # 테스트/QA 전담
```

**CLAUDE.md에 팀 정의:**

```markdown
## Agent Team 정의

### Coder Agent
역할: 기능 구현
- 테스트 없이 코드만 작성
- 빠른 초안 우선
- Reviewer의 피드백 반영

### Reviewer Agent
역할: 코드 품질 검증
- 보안, 성능, 가독성 검토
- 구체적인 라인 번호와 함께 피드백
- 승인/거부 판정

### Tester Agent
역할: 테스트 및 QA
- 엣지 케이스 발견
- 통합 테스트 작성
- 성능 벤치마크
```

**Python으로 Agent Teams 구현:**

```python
import asyncio
import anthropic

client = anthropic.Anthropic()

async def coder_agent(task: str) -> str:
    """구현 전담 에이전트"""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=4096,
        system="당신은 코드 구현 전문가입니다. 빠르게 동작하는 코드를 작성하세요.",
        messages=[{"role": "user", "content": task}]
    )
    return response.content[0].text

async def reviewer_agent(code: str) -> str:
    """코드 리뷰 전담 에이전트"""
    response = client.messages.create(
        model="claude-opus-4-6",   # 더 강력한 모델로 리뷰
        max_tokens=2048,
        system="당신은 시니어 코드 리뷰어입니다. 버그, 보안, 성능 문제를 찾으세요.",
        messages=[{"role": "user", "content": f"다음 코드를 리뷰해줘:\n\n{code}"}]
    )
    return response.content[0].text

async def tester_agent(code: str) -> str:
    """테스트 전담 에이전트"""
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=3000,
        system="당신은 QA 엔지니어입니다. 엣지 케이스를 찾고 테스트를 작성하세요.",
        messages=[{"role": "user", "content": f"이 코드의 테스트를 작성해줘:\n\n{code}"}]
    )
    return response.content[0].text

async def agent_team_pipeline(task: str):
    """팀 파이프라인 실행"""
    # 1단계: 구현
    print("🔨 Coder Agent 실행 중...")
    code = await coder_agent(task)

    # 2단계: 리뷰 + 테스트 병렬 실행
    print("🔍 Reviewer + Tester 병렬 실행 중...")
    review, tests = await asyncio.gather(
        reviewer_agent(code),
        tester_agent(code)
    )

    # 3단계: 리뷰 반영하여 최종 코드 수정
    final_code = await coder_agent(
        f"원본 코드:\n{code}\n\n리뷰 피드백:\n{review}\n\n피드백을 반영해서 수정해줘"
    )

    return {"code": final_code, "review": review, "tests": tests}

# 실행
result = asyncio.run(agent_team_pipeline("사용자 인증 미들웨어 구현"))
```

---

### 3. Adversarial Review (적대적 리뷰)

**개념:** 두 AI가 서로 상대방의 분석을 반박하며 더 정확한 결론을 도출.

```
Agent A: "이 코드에 문제가 있음"
     ↓ 피드백 전달
Agent B: "A의 분석에 동의/반박 + 추가 발견사항"
     ↓ 피드백 전달
Agent A: "B의 반박에 대한 재반박"
     ↓ [N라운드 반복]
오케스트레이터: 최종 합의된 결론 도출
```

```python
async def adversarial_review(code: str, rounds: int = 3):
    """두 에이전트가 서로 비판하며 코드 품질 향상"""

    # 첫 번째 에이전트: 문제점 발견
    agent_a_findings = await review_agent(
        f"이 코드의 모든 문제점을 찾아줘:\n{code}"
    )

    debate_history = [("A", agent_a_findings)]

    for round_num in range(rounds):
        # B가 A를 반박/보완
        agent_b_response = await review_agent(
            f"코드:\n{code}\n\n에이전트 A의 분석:\n{agent_a_findings}\n\n"
            f"A의 분석에서 틀린 부분을 지적하고 A가 놓친 문제를 추가로 찾아줘"
        )
        debate_history.append(("B", agent_b_response))

        # A가 B를 반박/보완
        agent_a_response = await review_agent(
            f"코드:\n{code}\n\n에이전트 B의 반박:\n{agent_b_response}\n\n"
            f"B의 반박 중 틀린 부분을 지적하고 최종 결론을 도출해줘"
        )
        debate_history.append(("A", agent_a_response))
        agent_a_findings = agent_a_response

    # 최종 합의 도출
    consensus = await synthesize_agent(
        f"다음 토론을 바탕으로 최종 합의된 코드 리뷰를 작성해줘:\n"
        + "\n".join([f"Agent {agent}: {text}" for agent, text in debate_history])
    )

    return consensus
```

---

### 4. Hybrid: Agent Teams + Ralph Loop

**개념:** Agent Teams(창의적 판단)와 Ralph Loop(기계적 반복)를 결합한 최강 패턴.

```
[오케스트레이터]
      ↓ 작업 배분
[Agent Teams] ← 설계/아키텍처 결정 (창의적 판단 필요)
      ↓ 구현 초안
[Ralph Loop]  ← 테스트 통과까지 자율 반복 (기계적 반복)
      ↓ 완료
[Reviewer]    ← 최종 품질 게이트
```

**Claude Code에서 오케스트레이터 프롬프트 예시:**

```
새로운 결제 모듈을 구현해줘.

단계:
1. [Plan Agent] 아키텍처 설계 및 파일 구조 확정
2. [Coder Agent] 기본 구현 (parallel worktree에서)
3. [Ralph Loop] 모든 테스트가 통과할 때까지 자동 반복 수정
4. [Reviewer Agent] 최종 보안/성능 리뷰

각 단계를 순서대로 실행하되, 3단계는 자율적으로 반복해줘.
```

---

## 커스텀 스킬 만들기 (레거시)

커스텀 스킬은 `~/.claude/commands/` 또는 `.claude/commands/`에 마크다운 파일로 저장된다.

> **참고:** v2.1.3 이후로는 SKILL.md 기반의 [스킬 시스템](#스킬-시스템) 사용을 권장한다. 기존 commands/ 파일도 계속 동작한다.

### 기본 구조

```markdown
<!-- ~/.claude/commands/my-skill.md -->
# My Skill

$ARGUMENTS 변수로 인자를 받을 수 있다.

## 지시사항
1. 이렇게 하고
2. 저렇게 해줘
```

### 호출 방법

```bash
/my-skill          # 인자 없이 호출
/my-skill arg1     # $ARGUMENTS = "arg1"
```

### 실용 스킬 예시

**커밋 스킬 (`commit.md`)**
```markdown
staged된 변경사항을 검토하고 Conventional Commits 형식으로 커밋해줘.

형식:
- feat: 새로운 기능
- fix: 버그 수정
- docs: 문서 변경
- refactor: 리팩토링
- test: 테스트 추가/수정

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

**PR 리뷰 스킬 (`review-pr.md`)**
```markdown
PR #$ARGUMENTS 를 리뷰해줘.

다음을 확인해줘:
1. 코드 품질과 가독성
2. 잠재적 버그
3. 보안 취약점
4. 성능 이슈
5. 테스트 커버리지
6. 문서화 여부

각 항목에 대해 구체적인 라인 번호와 함께 피드백을 제공해줘.
```

**배포 스킬 (`deploy.md`)**
```markdown
$ARGUMENTS 환경에 배포를 수행해줘 (staging/production).

배포 전 체크리스트:
1. 모든 테스트 통과 확인
2. 환경변수 설정 확인
3. 데이터베이스 마이그레이션 필요 여부 확인
4. 배포 후 헬스체크

staging이면 bun run deploy:staging
production이면 bun run deploy:prod (추가 확인 필요)
```

---

## IDE 통합

### VS Code

```bash
# VS Code Extension 설치
code --install-extension anthropic.claude-code
```

VS Code에서 Claude Code 기능:
- 인라인 코드 제안 (Ghost text)
- 선택한 코드 블록에 대한 즉시 질문
- 터미널 내 Claude Code 실행
- 진단(Diagnostics) 자동 전달

### JetBrains (IntelliJ, WebStorm, etc.)

JetBrains Marketplace에서 "Claude Code" 플러그인 설치

### Cursor / Windsurf

Claude Code CLI와 병렬 사용 가능 — CLI는 배치 작업, IDE 플러그인은 인라인 편집에 활용

---

## 권한 및 보안 모드

### 권한 레벨

```
Default Mode     → 모든 도구 실행 시 사용자 확인
Auto-approve     → 특정 도구 자동 승인
Read-only Mode   → 파일 수정 및 명령 실행 불가
```

### settings.json 권한 설정

```json
{
  "permissions": {
    "allow": [
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git status:*)",
      "Bash(bun test:*)",
      "Read",
      "Glob",
      "Grep"
    ],
    "deny": [
      "Bash(git push --force:*)",
      "Bash(rm -rf:*)",
      "Bash(DROP:*)"
    ]
  }
}
```

### 허용/거부 패턴

```
"Bash(git:*)"           → git으로 시작하는 모든 명령
"Bash(npm install:*)"   → npm install 명령
"Write(src/**)"         → src 하위 파일 쓰기만 허용
"Read"                  → 모든 읽기 허용
```

### CI/CD에서의 Non-interactive 모드

```bash
# 환경변수로 자동 승인 설정
CLAUDE_CODE_SKIP_PERMISSIONS=1 claude -p "테스트 실행 후 결과 리포트 생성"

# 안전한 CI 모드
claude --no-web-search --read-only -p "코드 리뷰해줘"
```

---

## 실제 기업 활용 사례

### Shopify — 개발자 생산성 향상

**사용 방식:**
- 수천 개 마이크로서비스의 코드 리뷰 자동화
- 레거시 Ruby 코드를 현대적 패턴으로 리팩토링
- 온보딩 시 새 개발자가 대규모 코드베이스 빠르게 이해

**결과:** 코드 리뷰 시간 40% 단축, 신규 개발자 온보딩 시간 60% 감소

**핵심 설정:**
```markdown
# Shopify CLAUDE.md (예시)
## Standards
- Ruby on Rails conventions 준수
- RuboCop 스타일 가이드 따르기
- 새 기능에 Sorbet 타입 명세 필수
```

### Stripe — 결제 API 문서화 자동화

**사용 방식:**
- API 변경사항 감지 후 문서 자동 업데이트
- SDK 코드에서 타입 정의 자동 추출
- 개발자 포털의 코드 예제 자동 생성 및 검증

**결과:** 문서 지연 시간 제거, API 정확도 99.9%

### Anthropic 내부 — Claude Code 자체 개발

**사용 방식 (dogfooding):**
- Claude Code가 Claude Code를 개발하는 데 참여
- 대규모 리팩토링 작업 자동화
- 테스트 커버리지 자동 확대

### 스타트업 패턴 — MVP 빠른 구축

```bash
# 단계별 MVP 구축
claude "Next.js 15로 SaaS 스타터를 만들어줘:
- Auth.js 인증
- Prisma + PostgreSQL
- Stripe 결제
- 기본 대시보드
CLAUDE.md를 먼저 만들고 시작해줘"
```

### 금융권 활용 — 컴플라이언스 자동화

**사용 방식:**
- SOX 컴플라이언스 체크 자동화
- GDPR 데이터 처리 코드 감사
- 보안 취약점 스캔 및 리포팅

```markdown
# 금융 CLAUDE.md 패턴
## Compliance Requirements
- PII 데이터는 반드시 암호화
- 모든 금융 계산은 BigDecimal 사용 (float 금지)
- 감사 로그 누락 금지
- OWASP Top 10 체크리스트 자동 검토
```

---

## 고급 패턴 및 Best Practices

### 1. 컨텍스트 윈도우 최적화

```bash
# 대화가 길어지면 /compact로 요약
/compact

# 새 작업은 /clear로 컨텍스트 초기화
/clear

# 큰 파일은 특정 범위만 읽도록 지시
claude "src/auth/index.ts의 100-200번 라인만 분석해줘"
```

### 2. 단계적 작업 분해 (Task Decomposition)

```bash
# 큰 작업을 단계로 나눠서 진행
claude "먼저 현재 인증 시스템을 분석해줘"
# → 분석 결과 확인
claude "분석 결과를 바탕으로 JWT 마이그레이션 계획을 세워줘"
# → 계획 검토
claude "계획대로 단계적으로 구현해줘"
```

### 3. 워크트리 (Worktree) 활용

```bash
# 독립된 환경에서 실험적 작업
claude --worktree feature/experimental

# 안전한 환경에서 대규모 리팩토링
claude "워크트리를 만들고 타입스크립트 마이그레이션을 안전하게 해줘"
```

### 4. 파이프라인 통합

```bash
# CI/CD 파이프라인에서 활용
# .github/workflows/ai-review.yml
name: AI Code Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install -g @anthropic-ai/claude-code
      - run: |
          claude -p "이 PR의 변경사항을 리뷰하고 문제점을 GitHub Actions 로그에 출력해줘" \
            --no-interactive \
            --model claude-haiku-4-5
```

### 5. 멀티 프로젝트 관리

```bash
# ~/.claude/settings.json에 프로젝트별 별칭
{
  "projects": {
    "frontend": "/Users/me/projects/web-app",
    "backend": "/Users/me/projects/api-server",
    "infra": "/Users/me/projects/terraform"
  }
}

# 프로젝트 전환
claude --cwd ~/projects/backend "API 엔드포인트 추가해줘"
```

### 6. Plan Mode 활용

```bash
# 실행 전 계획 검토
claude --plan "결제 시스템을 Stripe v3로 업그레이드해줘"
# → Claude가 계획만 제시하고 승인 대기
# → 승인 후 실행
```

### 7. 메모리 시스템 활용

```bash
# 세션 간 컨텍스트 유지
# ~/.claude/projects/{project-hash}/memory/MEMORY.md에 자동 저장

# 명시적 메모리 저장 지시
claude "우리 팀은 항상 bun을 사용한다는 것을 기억해줘"
```

---

## 관련 문서

- [AI 핵심 기법들](../techniques/README.md) — RAG, RLHF, ReAct 등
- [Claude Code 공식 문서](https://docs.anthropic.com/claude-code) *(외부 링크)*
- [MCP 공식 스펙](https://modelcontextprotocol.io) *(외부 링크)*

---

*마지막 업데이트: 2026-03-02 | 버전: Claude Code 2.1.x+*
