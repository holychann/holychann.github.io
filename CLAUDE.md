# holychann.github.io

소프트웨어(인프라, 시스템 아키텍처, 안정적인 시스템 구축·운영) 기술 블로그.
Jekyll + Chirpy 테마(gem 방식, `jekyll-theme-chirpy` 7.x), GitHub Pages에 GitHub Actions로 배포한다.
사이트: https://holychann.github.io

## 작업 원칙

- 사용자에게는 한국어로 답한다.
- 글 본문은 사용자가 직접 쓴다. Claude는 설정, 구조, front matter, 문법 변환, 검증, 배포만 맡는다.
  본문 문장·내용은 고치지 않는다. 틀려 보이는 내용은 고치지 말고 알려준다.
- 커밋 메시지는 conventional commit 형식 (`feat:`, `fix:`, `chore:`, `docs:` …).
- GitHub Pages 설정 화면의 Jekyll "Configure" 워크플로는 추가하지 않는다 (`.github/workflows/pages-deploy.yml`과 충돌).

## 로컬 환경

- Ruby는 Homebrew `ruby@3.4`를 쓴다. 기본 `ruby`(4.x)는 Chirpy와 호환되지 않는다.
  모든 Ruby 명령 앞에 `export PATH=/opt/homebrew/opt/ruby@3.4/bin:$PATH`
- Claude의 Bash 셸은 로케일이 US-ASCII라 htmlproofer가 한글 HTML에서 `Encoding::InvalidByteSequenceError`를 낸다.
  `LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8`를 같이 설정한다.
- `gh` CLI는 없다. Actions 결과는 공개 API로 확인한다:
  `https://api.github.com/repos/holychann/holychann.github.io/actions/runs?head_sha=<sha>`

```bash
# 로컬 서버 (http://127.0.0.1:4000). 글은 자동 반영, _config.yml 변경 시에는 재시작 필요
bundle exec jekyll s

# CI와 같은 검증 (빌드 결과는 저장소 밖 scratchpad에)
JEKYLL_ENV=production bundle exec jekyll b -d <scratch>/build
bundle exec htmlproofer <scratch>/build --disable-external \
  --ignore-urls "/^http:\/\/127.0.0.1/,/^http:\/\/0.0.0.0/,/^http:\/\/localhost/"
```

배포는 `main` push 때만 돈다 (Build and Deploy 워크플로). 다른 브랜치 push로는 사이트가 바뀌지 않는다.

## 글 발행 절차 ("발행해줘")

초안은 Obsidian 볼트에서 쓰고, 발행본만 이 저장소로 복사한다.

- 볼트: `/Users/holychan/Library/Mobile Documents/iCloud~md~obsidian/Documents`
  - 블로그 초안: `01_벨로그/` (파일명 = 한글 제목)
  - 첨부 이미지: `99_Attachments/`
  - 템플릿: `98_Templates/03_Blog.md` (코어 Templates 플러그인, `{{title}}` `{{date}}` 문법)
- 볼트는 저장소 밖이라 세션에 폴더 접근 권한을 먼저 요청해야 한다.
- Obsidian 원본 노트는 수정하지 않는다.

순서:

1. 노트를 읽고 front matter 점검 (title, categories, tags, description). 비었거나 이상하면 사용자에게 알린다.
2. 영문 소문자·하이픈 슬러그를 제안하고 **사용자 확인을 한 번 받는다** (URL `/posts/<슬러그>/`로 굳어짐).
3. `_posts/YYYY-MM-DD-<슬러그>.md`로 복사한다.
   - `date`는 발행 시각으로 바꾼다. `+0900` 필수. 현재보다 미래면 목록에 안 나온다.
   - 템플릿의 `#` 가이드 주석 블록은 지운다.
4. 본문에 쓰인 이미지를 `99_Attachments/`에서 `assets/img/posts/<슬러그>/`로 복사한다.
   파일명은 공백 없는 영문으로 바꾼다 (예: `Pasted image 2026….png` → `01-arch.png`).
5. Obsidian 전용 문법을 Chirpy 문법으로 바꾼다.

   | Obsidian | Chirpy |
   | --- | --- |
   | `![[img.png]]` | `![설명](/assets/img/posts/<슬러그>/img.png)` |
   | `![[img.png\|300]]` | `![설명](…){: width="300" }` |
   | `[[노트]]` | 발행된 글이면 `[텍스트](/posts/<슬러그>/)`, 아니면 링크 없이 텍스트 |
   | `> [!tip] 내용` | `> 내용` + 다음 줄 `{: .prompt-tip }` (tip/info/warning/danger) |
   | `==강조==` | `**강조**` |
   | `%%주석%%` | 삭제 |
   | 본문 `# 제목` (h1) | `##` 이하로 (제목은 front matter `title`) |

6. 위 CI와 같은 검증(프로덕션 빌드 + htmlproofer)을 통과시킨다.
7. 글과 이미지를 커밋하고 `main`에 push한다. 예: `docs: add post on redis pipelining`
8. Actions 성공과 라이브 페이지(`https://holychann.github.io/posts/<슬러그>/`) 응답을 확인하고 링크를 전달한다.

## Chirpy 주의사항

- `description`이 비면 본문 앞부분이 meta description/목록 요약으로 쓰인다.
  본문에 HTML 주석(`<!-- -->`)을 두면 그 내용이 요약에 새어 나오므로 본문에 주석을 남기지 않는다.
- 카테고리는 최대 2단계 `[상위, 하위]`, 태그는 소문자. 철자·대소문자가 다르면 별개로 묶인다.
- 존재하지 않는 로컬 이미지 경로를 쓰면 htmlproofer가 실패해 배포가 막힌다.
- 아바타(`assets/img/avatar.jpg`)는 112×112로 늘려 그리므로(`object-fit` 없음) 반드시 정사각형이어야 한다.
- 이 파일(CLAUDE.md)은 `_config.yml`의 `exclude`에 있어 사이트에 올라가지 않는다.

## 남은 작업

- giscus 댓글 설정 (`_config.yml`의 `comments`)
- Google Search Console 등록 (`webmaster_verifications.google`)
- velog 글 이전: velog 이미지 URL은 내려받아 `assets/img/posts/<슬러그>/`로 옮긴다
