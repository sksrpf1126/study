# git 명령어 및 동작 참고 글 정리

★ 2개 -> 무조건 숙지하고 넘어가자.  
★ 1개 -> 나중에 필요한 경우에 스스로 찾아서 쓸 수 있을 정도로만 이해하고 넘어가자.  
★ 0개 -> 좀 더 알고 싶거나, 시간에 여유가 있을 때 보도록 하자.

---

### ***★★ git pull, git fetch, git merge***

참고 글  
https://velog.io/@msung99/push-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EA%B9%83%ED%94%8C%EB%A1%9C%EC%9A%B0-pull  

원격 레포지토리가 누군가에 의해 변경됐을 경우 즉, 본인이 github 페이지의 원격 레포지토리로 가서 직접 수정하거나 협업에 의해 다른 사람이 원격 레포지토리에 변경사항을 적용한 경우에는 내 로컬로 해당 변경사항을 가져와야 하며 이때 사용하는 명령어가 ***git pull*** 이다.   

git pull 명령어는 git fetch와 git merge를 동시에 하는 명령어이다. 

```
git pull의 내용을 이해하기 위해 알고 있어야 할 명령어
1. git fetch
2. git merge
```

---


### ***★★ git clone***

참고 글  
https://choiiis.github.io/git/how-to-clone-project/ -> git clone을 중점으로 설명  
https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-%EA%B9%83%ED%97%99-%EC%9B%90%EA%B2%A9-%EC%A0%80%EC%9E%A5%EC%86%8C-%EA%B4%80%EB%A6%AC-git-remote  -> git pull, git clone 외 여러 내용을 같이 설명(추천)

github에 올려둔 원격 레포지토리의 작업 내용들을 내 로컬로 가져오고 싶다면 쓰는 명령어  
***"처음 한번"***, 내 로컬에 해당 원격 레포지토리의 작업 내용이 없을 때 가져오는 것이다.  

git clone 명령어는 git remote add(원격 저장소와 연결하는 명령어) + git pull(작업 내용 가져오는 명령어)를 합쳐서 실행한다고 한다.

즉, 정리하면  

***git clone = git remote add + git pull***

```
git clone의 내용을 이해하기 위해 알고 있어야 할 명령어
1. git remote add
2. git pull
```

---

### ***★git add, commit, push 취소 명령어***

참고 글  
https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-git-add-commit-push-%EC%B7%A8%EC%86%8C%ED%95%98%EA%B8%B0-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC-git-reset-restore-clean#git_commit_%EC%B7%A8%EC%86%8C%ED%95%98%EA%B8%B0

간혹, git add 또는 git commit 또는 git push를 취소 즉, 롤백하고 싶은 경우가 있을 것이다.  
그런 상황에서 쓰이는 명령어들을 위 글에서 잘 정리해놓았다.  

---

### ***★git stash***

참고 글  
https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-%EC%BB%A4%EB%B0%8B%ED%95%98%EC%A7%80%EC%95%8A%EA%B3%A0-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EA%B0%84-%EC%9D%B4%EB%8F%99-git-stash

협업을 할 때에는 브랜치를 많이 만들며, 브랜치 간에 이동이 빈번하다. 그런데 이동하기 전에 내 현재 브랜치의 로컬환경에서 작업한 내용이 남아있다면 이동에 제한이 걸리며 덮어씌울것인지 아니면 해당 명령어인 stash를 사용해서 임시로 저장해 둘 것인지를 선택해야 한다.  

그런데 대부분 해당 명령어를 통해 임시 저장을 한다.  

---

### ***★★★★★ branch 관련***

참고 글  
https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-%EA%B9%83-Branch-%EC%A0%95%EB%A6%AC-branch-checkout-merge-rebase  
솔직히 위에 분의 글들이 너무 잘 정리가 되어 있어서, branch 내용도 정리가 되어있나 찾아봤는데 매우 잘 정리가 되어있다.  
위에서 정리한 명령어들만 자유자재로 쓸 줄 안다면, 협업할 때 문제가 없을 거라고 생각한다.  

만약 branch에 대한 명령어보다 개념이 잘 이해가 안된다면, 아래의 영상들을 참고하도록 하자.  

https://www.youtube.com/watch?v=PmWPdYkAMg4&t=1s //생활코딩 유튜브 - 코드가 아닌 branch라는 개념을 그림으로 쉽게 이해시켜줌 (먼저 보는 것을 추천)

https://www.youtube.com/watch?v=XFm2qNs30BE  //코딩애플 유튜브 - 실제 코드를 통해 동작과정 설명

https://www.youtube.com/watch?v=1I3hMwQU6GU //얄팍한 코딩사전 유튜브 - 브랜치 내용은 1:06:01 ~ 1:25:11 참고  

---

