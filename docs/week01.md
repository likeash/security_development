# 1주차

# 1. 토큰 실습 정리

## Q1. 내 토큰으로 `https://api.github.com/user` 를 호출했을 때 **무엇이 보였는지**

```xml
['login', 'id', 'node_id', 'avatar_url', 'gravatar_id', 'url', 'html_url',
 'followers_url', 'following_url', 'gists_url', 'starred_url',
 'subscriptions_url', 'organizations_url', 'repos_url', 'events_url',
 'received_events_url', 'type', 'user_view_type', 'site_admin', 'name',
 'company', 'blog', 'location', 'email', 'hireable', 'bio',
 'twitter_username', 'notification_email', 'public_repos', 'public_gists',
 'followers', 'following', 'created_at', 'updated_at']
```

와 같은 값이 출력되었으며, 닉네임, id, github url, 이름, 회사, 이메일 등 유저에 대한 정보를 얻을 수 있었다. 

## Q2. 토큰을 revoke한 후에 같은 호출이 어떻게 달라졌는지

```bash
PS C:\WINDOWS\System32> curl.exe -s -H "Authorization: Bearer $TOKEN" https://api.github.com/user
{
  "message": "Bad credentials",
  "documentation_url": "https://docs.github.com/rest",
  "status": "401"
}
```

Q1과 같은 명령어로 호출했을 때, bad credentials 오류가 뜨며 이전처럼 사용자 정보가 출력되지 않음을 확인할 수 있다.

## Q3. Fine-grained token으로 새로 발급한 뒤, 보이는 범위가 어떻게 줄었는지

fine-grained token을 발급한 후 호출했을 때 보이는 범위는 다음과 같다.

```bash
'login', 'id', 'node_id', 'avatar_url', 'gravatar_id', 'url', 'html_url',
 'followers_url', 'following_url', 'gists_url', 'starred_url',
 'subscriptions_url', 'organizations_url', 'repos_url', 'events_url',
 'received_events_url', 'type', 'user_view_type', 'site_admin', 'name',
 'company', 'blog', 'location', 'email', 'hireable', 'bio',
 'twitter_username', 'notification_email', 'public_repos', 'public_gists',
 'followers', 'following', 'created_at', 'updated_at'
```

살펴보면 Q1때 사용했던 classic token과 호출 후 보이는 범위에 대해서는 차이가 없는 것을 알 수 있다.

## Q4. 한 줄 소감 (권한을 좁게 주는 게 왜 중요한지)

토큰을 얻는다는 것은 곧 특정 레포에 접근할 수 있는 권한을 얻는다는 것을 의미한다. 권한을 좁게 주는 것이 중요한 이유는 토큰이 유출되더라도 제한된 범위에서만 권한을 사용할 수 있기 때문에 피해의 범위를 줄일 수 있어서이다.

# 2. 키가 새는 실수 5가지 정리

## 1. 코드에 하드 코딩

레포지토리 안에 있는 파일 안에 키를 주석 등의 형태로 달아놓는 경우이다. 

레포가 public으로 공개되어 있는 경우 모든 사람이 키에 접근할 수 있기 때문에 위험하다. 

따라서, 키를 환경 변수로 설정하여 사용하거나 public으로 올라가지 않는 파일에 저장하여 사용해야 한다.

## 2. .env만 만들고 .gitignore 안 함

.env와 같은 파일에 올렸지만 .gitignore에 .env 파일을 넣지 않아 public하게 올라간 경우이다.

이 경우도 코드에 하드 코딩하는 것과 같이 모든 사람이 키에 접근할 수 있기 때문에 위험하다.

따라서, .gitignore 안에 키 관련 파일을 집어넣음으로써 키 값이 public하게 공개되지 않도록 설정해야 한다.

## 3. .gitignore를 나중에 추가

이미 git에 올린 키 관련 파일을 확인해 .gitignore에 사후에 추가한 경우이다.

이 경우, .gitignore에 추가한 이후 커밋되는 범위에서는 .gitignore에 의해 키 유출이 일어나지 않지만 .gitignore을 추가하기 전 커밋 버전에 키 관련 파일이 여전히 남아있기 때문에 위험하다.

따라서, .gitignore에 추가하는 것만으로는 부족하고 이전 커밋 버전을 삭제한 후 유출된 키 값을 재발급받아 사용해야 한다.

## 4. 변형 파일

.gitignore에 .env과 같은 파일은 적어놨지만 .env.local, .env.product과 같은 변형 파일은 .gitignore에 추가하지 않은 경우이다.

이 경우, .env와 같은 파일은 필터링되겠지만 .env.*에 해당하는 중요 파일은 그대로 public으로 올라가기 때문에 위험하다.

따라서, .gitignore에는 변형 파일까지 고려해 값을 집어넣어야 한다.

## 5. PR, 댓글에 붙여넣기

key 관련 내용을 pr 또는 댓글에 붙여넣는 경우이다.

pr과 댓글을 public으로 볼 수 있을 뿐만 아니라 수정하더라도 편집 기록에 남기 때문에 위험하다.

따라서, pr이나 댓글에 key를 붙여넣는 짓은 하질 말아야 한다.

# +

## step 1

왜 `.env.*` 까지 넣었는가? 

- `.env` 까지만 넣어서는 `.env.local` 과 같은 변형 파일이 올라가는 것을 막을 수 없음. 따라서, `.env.*` 를 추가해 변형 파일까지 올라가는 것을 막을 수 있도록 설정함

## step2

실제로 막히는 지 검증

```bash
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   .env
        modified:   .env.local

no changes added to commit (use "git add" and/or "git commit -a")
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   .env
        modified:   .env.local

no changes added to commit (use "git add" and/or "git commit -a")
rm '.env'
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    .env

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   .env.local
```

처음 실행했을 때는 위와 같은 결과가 나왔다. 이는 전에 적용했던 커밋에 .env, .env.local 파일이 올라가있었기 때문이다. 이를 `git rm --cached .env, git rm --cached .env.local`을 이용해 추적에서 지우고 커밋 후 다시 결과를 받아오면 아래와 같은 결과가 나온다.

```bash
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
fatal: pathspec '.env' did not match any files
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```