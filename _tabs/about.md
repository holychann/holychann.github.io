---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
description: Spring으로 백엔드를 만들고 직접 배포해 운영하는 성찬입니다. 운영하며 원인을 찾은 기록과 기술을 고른 이유를 정리합니다.
---

{% assign mail = site.social.email | split: '@' %}
<div class="about">
  <section class="about-hero">
    <p class="about-hello">안녕하세요, 성찬입니다.</p>
    <p class="about-lead">Spring으로 백엔드를 만들고 직접 배포해 운영합니다.</p>
    <p class="about-intro">운영하면서 예상과 다르게 동작하는 일을 여러 번 겪었습니다. 이 블로그에는 그때 원인을 어떻게 찾았는지, 무엇을 왜 골랐는지를 기록합니다. 문제를 이해하려고 공부한 개념도 함께 정리합니다.</p>
    <p class="about-links">
      <a class="about-link" href="https://github.com/{{ site.github.username }}" target="_blank" rel="noopener"><i class="fab fa-github" aria-hidden="true"></i>GitHub</a>
      <a class="about-link" href="javascript:void(location.href = 'mailto:' + ['{{ mail[0] }}','{{ mail[1] }}'].join('@'))"><i class="fas fa-envelope" aria-hidden="true"></i>메일</a>
      <a class="about-link" href="{{ '/feed.xml' | relative_url }}"><i class="fas fa-rss" aria-hidden="true"></i>RSS</a>
    </p>
  </section>

  <h2>원인을 찾은 기록</h2>

  <article class="about-case">
    <h3 class="about-case-title"><a href="{{ '/posts/dynamodb-write-throttling/' | relative_url }}">DynamoDB에서 발생한 저장 지연의 원인을 찾아서</a></h3>
    <p class="about-case-note">약 300행을 저장하는 데 19초 가까이 걸렸고, 온디맨드로 바꾼 뒤 1.8초로 줄었습니다.</p>
    <dl class="about-trace">
      <dt>처음 생각</dt>
      <dd><del>배치 요청을 순차로 보내는 방식</del></dd>
      <dt>실제 원인</dt>
      <dd><strong>1 WCU로 잡혀 있던 쓰기 용량</strong></dd>
    </dl>
  </article>

  <article class="about-case">
    <h3 class="about-case-title"><a href="{{ '/posts/redis-pipelining-slower/' | relative_url }}">Redis 에서 파이프라이닝을 적용했는데 왜 느려질까?</a></h3>
    <p class="about-case-note">캐시 네 개를 지우는 왕복을 줄이려고 파이프라이닝을 넣었는데 처리 시간이 약 4배 늘었습니다.</p>
    <dl class="about-trace">
      <dt>처음 생각</dt>
      <dd><del>왕복이 한 번으로 줄면 빨라진다</del></dd>
      <dt>실제 원인</dt>
      <dd><strong>풀링되지 않은 전용 연결</strong></dd>
    </dl>
  </article>

  <h2>고른 이유를 남긴 기록</h2>

  <article class="about-case">
    <h3 class="about-case-title"><a href="{{ '/posts/dynamodb-iam-role/' | relative_url }}">DynamoDB 접근 키를 없애기 위해 Secrets Manager 대신 IAM 역할을 선택한 이유</a></h3>
    <p class="about-case-note"><code>.env</code>에 두던 AWS 액세스 키를 서버에서 없애야 했습니다.</p>
    <dl class="about-trace">
      <dt>후보</dt>
      <dd class="about-options"><del>Secrets Manager</del> <del>Lambda</del> <strong>IAM 역할</strong></dd>
      <dt>고른 이유</dt>
      <dd>Lambda는 호출용 키가 서버에 새로 남고, Secrets Manager는 관리할 곳만 늘었습니다. 역할은 서버에 키를 두지 않습니다.</dd>
    </dl>
  </article>

  <article class="about-case">
    <h3 class="about-case-title"><a href="{{ '/posts/kafka-vs-rabbitmq/' | relative_url }}">Kafka 와 RabbitMQ 비교와 선택의 이유</a></h3>
    <p class="about-case-note">직렬에 가깝게 돌던 Spring Scheduler 폴링에서 처리 로직을 메시지 브로커로 떼어 내야 했습니다.</p>
    <dl class="about-trace">
      <dt>후보</dt>
      <dd class="about-options"><del>Kafka</del> <strong>RabbitMQ</strong></dd>
      <dt>고른 이유</dt>
      <dd>이벤트 200개 내외를 병렬로 처리하기에 충분했고 연동이 간단했습니다. Kafka로 바꿀 수 있게 설계해 두었습니다.</dd>
    </dl>
  </article>

  <p class="about-more">다른 글은 <a class="about-link" href="{{ '/categories/' | relative_url }}">카테고리</a>와 <a class="about-link" href="{{ '/archives/' | relative_url }}">아카이브</a>에서 볼 수 있습니다.</p>

  <h2>다뤄 본 기술</h2>

  <dl class="about-stack">
    <dt>백엔드</dt>
    <dd>Java, Spring Boot, Spring Security, JPA</dd>
    <dt>데이터</dt>
    <dd>Redis, DynamoDB</dd>
    <dt>메시징</dt>
    <dd>RabbitMQ</dd>
    <dt>배포·인프라</dt>
    <dd>GitHub Actions, Docker, Fly.io, AWS EC2, IAM, CloudWatch</dd>
    <dt>모니터링</dt>
    <dd>Prometheus, Grafana</dd>
  </dl>

  <p class="about-foot">틀린 내용이나 궁금한 점은 각 글 아래 댓글로 남겨 주세요. 댓글은 GitHub 계정으로 달 수 있습니다.</p>
</div>
