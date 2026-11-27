# project-md-wiki

문서 목록과 각 문서를 읽을 시점은 [docs/README.md](docs/README.md)에, 모든 작업이 따르는 절차는 [AGENTS.md](AGENTS.md)에 있습니다.

## 문서 체이닝

커밋 `58f4118` 기준으로 문서 사이의 연결을 점검한 결과입니다. 줄 번호는 이 커밋의 파일 기준이라서 문서가 바뀌면 달라질 수 있습니다.

체이닝은 2단 허브 구조입니다. 링크는 `AGENTS.md`와 인덱스 `docs/README.md` 두 곳에만 있습니다. 개별 문서 18개는 서로 링크하지 않는데, `writing-docs.md:22`가 "다른 문서는 인덱스와 AGENTS.md에서만 가리킨다"고 정해 두었기 때문입니다. 링크는 모두 실제 파일로 이어집니다. 다만 링크 없이 용어로만 이어진 곳 몇 군데가 약한 고리입니다.

### 1. 링크가 걸린 곳

```text
AGENTS.md                              모든 작업의 진입점
|-- docs/README.md                     :8 시작 단계, :12 작업 중 다른 행에 해당하면 그 문서도 읽기
|   |-- standards/writing-docs.md      :3 소개문 (문서를 어느 폴더에 둘지)
|   `-- "When to read" 표 18행         :9-26, 한 행에 문서 하나
|-- docs/standards/writing-docs.md     :22 마무리: 문서를 바꾸면 이 기준으로 검토 에이전트를 돌림
`-- docs/standards/glossary-sample.md  :23 마무리: 이 파일을 바꾼 PR은 머지하지 않음
```

- **들어오는 링크 수:** writing-docs.md는 세 곳(docs/README.md:3·18, AGENTS.md:22)에서, glossary-sample.md는 두 곳(docs/README.md:15, AGENTS.md:23)에서 링크됩니다. 나머지 16개는 인덱스에서만 링크됩니다.
- **개별 문서가 이름으로 가리키는 곳:** 허용된 두 곳뿐입니다. `AGENTS.md`는 writing-docs.md:3·22·24와 quality-gates.md:65에서, 인덱스는 writing-docs.md:19·21·30·32에서 가리킵니다.

### 2. 체이닝을 정한 규칙

| 위치 | 내용 |
| --- | --- |
| writing-docs.md:22 | 다른 문서는 인덱스와 AGENTS.md에서만 가리킵니다. 서술로 가리키는 것도 금지입니다. |
| writing-docs.md:21 | 한 종류의 작업에 필요한 규칙은 그 행의 문서 하나에 둡니다. |
| writing-docs.md:23 | 두 프레임워크가 함께 따르는 규칙은 `standards/`에 둡니다. 양쪽 폴더에 같은 주제 문서가 있으면 양쪽에 복사하고 함께 고칩니다. |
| writing-docs.md:24 | 모든 작업이 따르는 절차는 AGENTS.md에만 둡니다. |
| writing-docs.md:30·32 | 인덱스는 하나만 둡니다. 문서 배치를 바꾸면 같은 변경에서 인덱스도 고칩니다. |
| AGENTS.md:12 | 작업 도중 다른 행에 해당하게 되면 그 문서를 읽고 계속합니다. |
| AGENTS.md:21 | 규칙을 바꾸면 그 규칙이 들어 있는 모든 문서를 같은 변경에서 고칩니다. |

### 3. 인덱스 행이 겹쳐서 걸리는 체인

여러 행의 조건이 겹치게 쓰여 있어서, 링크가 없어도 한 작업이 여러 문서로 이어집니다.

| 작업 | 걸리는 문서 (docs/README.md 줄 번호) |
| --- | --- |
| NestJS 앱 추가 | nx-monorepo(10), nestjs/logging(21), nestjs/environment-variables(22) |
| 앱이 시작할 때 필요한 환경 변수 추가 | nx-monorepo(10), deploy(12), nestjs(22) 또는 nextjs(25) environment-variables |
| 다른 앱을 띄우는 테스트 | nx-monorepo(10, 행에 직접 적혀 있음), nestjs/testing(20) |
| PR로 끝나는 작업 | linear(14), github(11) |
| 문서 변경 | writing-docs(18)와 AGENTS.md:22의 검토 2개. PR이면 github.md:16(바뀐 부분마다 리뷰 코멘트)도 걸림 |
| Omni 용어 사용 | glossary-sample(15). writing-docs.md:45, nextjs/architecture.md:23·41은 "Omni term"이라는 말로만 이 행에 연결됨 |

그래서 두 문서에 걸친 다음 내용도 링크 없이 함께 읽힙니다.

- `checkEnv`: nestjs/environment-variables.md:29, nx-monorepo.md:8·17
- `envSchema`: nestjs/logging.md:14, nestjs/environment-variables.md:18
- 다른 앱의 build 결과를 쓰는 테스트: nestjs/testing.md:21, nx-monorepo.md:48

### 4. 함께 고쳐야 하는 복사본

| nestjs | nextjs | 상태 |
| --- | --- | --- |
| environment-variables.md:10-11 (비밀값) | environment-variables.md:5-6 | 같음 |
| testing.md:5 (`@/` import) | testing.md:5 | 같음 |
| testing.md:39-40 (테스트 데이터) | testing.md:11-12 | 같음 |
| testing.md:41 (fixture 위치) | testing.md:13 | 각 프레임워크의 테스트 위치에 맞게 바꿈 |
| testing.md:65 (결과가 바뀌는 테스트) | testing.md:17 | caller가 user로 바뀐 것만 다름 |
| testing.md:42 (기대값은 손으로 쓰기) | 없음 | 확인 필요 |
| testing.md:64 (처음 통과한 테스트는 일부러 깨 보기) | 없음 | 확인 필요 |

`standards/` 안에도 같은 사실이 두 곳에 있습니다. github.md:13과 linear.md:41 모두 "`Fixes`로 연결된 Issue는 PR이 머지되면 Done이 된다"고 적고 있습니다. 프레임워크 사이의 복사가 아니어서 writing-docs.md:23의 대상은 아니지만, AGENTS.md:21에 따라 함께 고쳐야 합니다.

### 5. 점검 결과

**정상**

- 명시적 링크 19개가 모두 실제 파일로 이어집니다.
- 개별 문서 18개와 인덱스 행 18개가 하나씩 짝을 이룹니다. 인덱스에 빠진 문서나 중복된 행은 없습니다.
- 개별 문서가 다른 개별 문서를 파일 이름으로 가리키는 곳은 없습니다.
- 최근 이름을 바꾼 세 파일(glossary-sample, deployed-databases-sample, nx-monorepo)에 대해 옛 이름을 쓰는 곳이 없습니다. AGENTS.md와 docs/README.md에도 새 이름이 반영되어 있습니다.

**약한 고리**

1. **github.md:26:** "unattended"와 "end-of-work report"를 정의 없이 씁니다. 이 두 말은 linear.md:28·39에만 정의되어 있습니다. GitHub 설정만 바꾸는 작업은 PR로 끝나지 않아서 linear 행(docs/README.md:14)에 해당하지 않을 수 있습니다. 그러면 github.md만 읽은 사람은 이 말의 뜻을 알 수 없습니다. 가장 작은 수정은 github.md:26을 이 용어 없이 풀어 쓰는 것입니다.
2. **github.md의 PR 템플릿:** 범용 문구라서 문서를 서술로 가리킵니다.
   - :54의 "glossary terms"는 glossary 문서를 특정해서 가리키므로 writing-docs.md:22에 어긋납니다. 점검한 범위에서는 이런 곳이 여기 하나입니다. 다른 문서처럼 "Omni terms"로 바꾸면 인덱스 15행으로 연결됩니다.
   - :125의 "the repository's docs mark as sensitive"는 가리킬 대상이 없습니다. 민감 영역을 표시한 문서가 없고, "sensitive"라는 말은 github.md에만 나옵니다.
3. **복사 비대칭:** 4번 표의 nestjs/testing.md:42와 :64는 프레임워크와 상관없는 규칙으로 보이는데 nextjs/testing.md에는 없습니다. NestJS에만 적용하려고 뺀 것인지 복사를 빠뜨린 것인지는 문서만으로 판단할 수 없습니다.
4. **이 저장소에는 `.github/`가 없습니다.** 그래서 AGENTS.md:18(`ci.yml`), quality-gates.md:65(`docs.spec.ts`), nx-monorepo.md:7·17, deployed-databases-sample.md:32가 가리키는 코드가 여기 없습니다. 특히 quality-gates.md:65에 나오는 문서 검사 `docs.spec.ts`가 없어서, 이 저장소에서는 위 체이닝 규칙을 어겨도 실패하는 검사가 없습니다.

nextjs/design-system.md:5의 `DESIGN.md`는 설치된 패키지 안의 문서를 일부러 가리키는 외부 참조라서 문제가 아닙니다.

### 점검 방법

20개 파일을 모두 읽고, 다음 네 가지를 스크립트로 확인했습니다.

- 링크가 실제 파일로 이어지는지
- 인덱스가 모든 문서를 한 번씩 가리키는지
- 문서끼리 파일 이름으로 가리키는 곳이 있는지
- 문서 사이에 유사도 0.9 이상인 문장이 있는지
