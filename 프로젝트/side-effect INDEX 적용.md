# **_좋아요 기능 INDEX 적용_**

해당 내용은 [사이드 이펙트](https://github.com/Side-Effect-Team/side-effect-backend) 프로젝트와 관련된 내용입니다.

---

## **_적용 배경_**

프로젝트의 모집 게시글에 **_"좋아요"_** 하는 기능이 존재하며, 좋아요 데이터를 저장하는 테이블 또한 존재합니다.

엔티티는 아래와 같이 정의되어 있는 상태입니다.

```java
@Getter
@Entity
@Table(name = "RECRUIT_LIKES")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class RecruitLike {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "recruit_like_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "recruit_board_id")
    private RecruitBoard recruitBoard;
}
```

좋아요는 토글 방식으로 동일한 API를 호출하며 처음에 호출한 경우에는 좋아요 데이터를 생성하고, 다시 동일하게 호출하면 좋아요 데이터를 삭제합니다.

자세한 로직은 아래와 같습니다.

```java
@Transactional
public RecruitLikeResponse toggleLike(User user, Long boardId) {
    Optional<RecruitLike> recruitLike = recruitLikeRepository.findByUserIdAndRecruitBoardId(user.getId(), boardId);

    if (recruitLike.isPresent()) {
        RecruitLike findRecruitLike = recruitLike.get();
        recruitLikeRepository.delete(findRecruitLike);
        return RecruitLikeResponse.of(findRecruitLike, LikeResult.CANCEL_LIKE);
    }

    return RecruitLikeResponse.of(likeBoard(user, boardId), LikeResult.LIKE);
}

private RecruitLike likeBoard(User user, Long boardId) {
    RecruitBoard findRecruitBoard = recruitBoardRepository.findById(boardId)
            .orElseThrow(() -> new EntityNotFoundException(ErrorCode.RECRUIT_BOARD_NOT_FOUND));

    return recruitLikeRepository.save(RecruitLike.createRecruitLike(user, findRecruitBoard));
}
```

여기서 맨 위의 findByUserIdAndRecruitBoardId 메서드는 좋아요를 누른 사용자의 id와 좋아요의 대상이 되는 모집 게시글의 Id로 좋아요 테이블에 쿼리를 날립니다.

해당 부분에서 팀원 분이 recruitBoardId와 userId 2개의 컬럼에 복합 인덱스(unique)를 거는 것이 좋을수도 있다고 의견을 주셨습니다.

그런데 좋아요의 경우 처음에 API를 호출할 때에는 save 즉, insert 쿼리가 발생하며, 다시 눌러서 취소하는 경우에는 delete 쿼리가 발생하게 됩니다.

인덱스는 "조회"에 대해서는 성능이 향상되지만, update, delete, insert와 같이 테이블의 데이터에 영향이 가는 쿼리를 사용하는 경우 이에 따라 인덱스도 변경이 되어야 하기 때문에 갱신이 자주 발생하는 테이블에 대해서는 오히려 인덱스가 성능을 약화시킨다는 내용을 봤었습니다.

하지만, 이론만 알고 실제로 제대로 된 인덱스를 적용해 본적이 없기 때문에 위 경우에 인덱스를 적용하냐, 안하느냐에 따라 성능차이가 어떤지 판단을 내릴 수가 없었습니다.

그래서 이번 기회에 실제로 인덱스에 따라 성능 차이가 어떤지 테스트를 해보고, 프로젝트에도 적용을 할지 안할지 판단을 내리기로 하였습니다.

</br>

---

## **_인덱스 적용 전_**

### **_테스트 환경_**

- 개인 로컬 환경
- mariaDB (DBever 툴 사용)
- postMan

</br>

### **_테스트 데이터 수_**

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/b48f8005-e6ab-48ed-9b76-c3df0bb86321" width = 70%>
</p>

배포된 서버의 DB에는 의미없는 데이터를 넣을 수 없어, 개인 로컬 환경에서 테스트를 합니다.  
인덱스를 적용할 테이블에 데이터가 너무 적으면, 제대로 된 테스트를 할 수 없을 것 같아 우선 위와 같이 150만건 정도의 데이터를 넣었습니다.

</br>

### **_테이블 인덱스 현황_**

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/741ba773-edb9-4824-9d7e-ba76d1517083" width = 70%>
</p>

위와 같이 현재 기본키에 대한 인덱스(클러스터 인덱스)만 있는 상태입니다.

</br>

### **_실행 계획_**

아래와 같은 쿼리를 날릴 경우 DB내에서의 어떤 실행 계획을 세우는지 확인해 보겠습니다.

```sql
select SQL_NO_CACHE *
from recruit_likes rl
where rl.recruit_board_id = 17 and rl.user_id = 22;
```

조건문에 recruit_board_id와 user_id를 사용하는 쿼리입니다. SQL_NO_CACHE는 DB 캐시 기능에 의해 쿼리를 다시 돌릴 시 캐시로 인한 속도 개선을 해줍니다.  
하지만 저는 인덱스만을 통해 성능의 개선이 되는지 확인하고 싶어, 캐시 기능을 막아놓았습니다.

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/a2d308a6-2d19-4fe8-9202-4301d7816c18" width = 70%>
</p>

type이 ALL일 경우에는 인덱스를 이용한 방식이 아닌 풀 스캔이 동작한다고 합니다.  
풀 스캔으로 동작하게 된다면 150만건의 모든 데이터를 스캔하여 조건으로 필터링을 해야됩니다.

</br>

### **_쿼리 성능_**

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/5d4f4d45-1a61-4648-bb10-7ee0851200bf" width = 60%>
</p>

383번의 Query Id가 위의 쿼리에 해당됩니다. 해당 Query Id를 상세 조회를 해보면 다음과 같습니다.

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/a7ffb2ec-576a-4d56-a168-1cd018a51d16" width = 20%>
</p>

핵심은 sending data부분의 소요 시간입니다. 인덱스를 적용하기 전 0.359042 정도의 시간이 걸리는 것을 확인할 수 있습니다.  
그럼 인덱스를 적용하기 전의 해당 쿼리를 사용하는 좋아요 API는 평균적으로 어느정도 시간이 소요가 되는지 확인을 해보겠습니다.  
postMan을 통해 API를 호출해 봤습니다.

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/92f70c71-2c31-48fc-9fbd-982f2469f93f" width = 60%>
</p>

좋아요를 누른 경우의 API는 317ms가 걸렸습니다.

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/57af5c54-fdc3-4c61-b86d-c063d2cfa78c" width = 60%>
</p>

좋아요를 취소한 경우의 API는 370ms가 걸렸습니다.  
혹시 몰라 여러번 API를 호출해 봤는데, 평균적으로 330ms ~ 340ms 정도의 시간이 소요가 되었습니다.  
현재는 150만건의 데이터로 테스트를 한 경우이지만 만약 데이터가 몇천만건 이상이 된다면 기본으로 초단위 시간이 소요가 될 것으로 예상됩니다.

</br>

---

## **_인덱스 적용 후_**

### **_엔티티에 인덱스 추가_**

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/08ea0136-1b91-47a6-9783-bf609e5f66fc" width = 60%>
</p>

위와 같이 엔티티에 인덱스를 추가하게 되면, 아래와 같은 로그를 통해 인덱스가 생성되는 것을 확인할 수 있습니다.

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/f766c2f0-8aa0-4cbf-8a2f-1a2247ffa31d" width = 60%>
</p>

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/fbef93cf-50ac-4b54-bd8f-d479272f055d" width = 60%>
</p>

테이블에도 인덱스가 추가된 것을 확인할 수 있습니다.

### **_실행 계획_**

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/9afb08c9-ab97-4b89-9286-21ee807697f5" width = 60%>
</p>

- type : const  
  const인 경우는 PK나 유니크 인덱스 검색을 이용해서 레코드에 접근할 때 사용된다고 하며, 가장 빠른 검색 방식이라고 합니다.

- possible_keys  
  레코드에 접근하기 위해 사용할 수 있는 키 혹은 인덱스 목록을 보여준다고 하며, 해당 값은 이전에 추가한 인덱스임을 알 수 있습니다.

- key, key_len  
  레코드에 접근하기 위해 어떤 index를 사용하는지, 인덱스 총 몇바이트를 참조하는지에 대한 정보입니다.

결론은 해당 쿼리를 호출하게 된다면, 추가한 인덱스가 정상적으로 사용이 될 것이라는 것을 확인할 수 있습니다.

</br>

### **_쿼리 성능_**

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/827ccca5-7241-4669-8d6f-a2f2e1fc9a50" width = 60%>
</p>

460, 462, 464에 해당하는 Query Id 부분이 인덱스가 적용된 쿼리를 호출한 것으로 3번을 호출함으로써, 평균적으로 소요시간을 알 수 있습니다.  
460의 경우 0.0003572, 462의 경우 0.0002328, 464의 경우 0.0003142 정도 소요가 됩니다. 3번의 평균 소요 시간은 0.0003014로, 인덱스 적용하기 전의 소요시간인 0.3594455 보다 약 1,192배 빠른 것을 확인할 수 있었습니다.  
물론, 겨우 몇번의 결과로들만 비교하였기 때문에 정확하지는 않지만 그래도 엄청난 속도차이가 있음을 알 수 있었습니다.

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/dbf0bb3f-f010-41fd-893c-c8b76162bf45" width = 25%>
</p>

위는 460번의 Query Id를 상세조회한 것으로, Sending data 부분의 소요시간이 0.000009로, 엄청나게 빠른 시간으로 처리가 됨을 알 수 있습니다.

아래는 인덱스를 적용한 후의 API 성능은 어느정도인지 postMan을 통해 호출을 해본 결과입니다.

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/7c05b8db-dd24-49b4-bbd6-ea57ab354818" width = 60%>
</p>

인덱스를 적용한 후의 좋아요를 누른 경우의 API는 28ms의 속도가 나왔습니다. 인덱스 적용하기 전의 경우 317ms가 나왔는데, 거의 11배 차이의 속도가 난다는 것을 확인할 수 있었습니다.

</br>

<p align = "center">
<img src="https://github.com/sksrpf1126/study/assets/62879192/1234e3c4-c26f-4994-b80f-b720158f6eef" width = 60%>
</p>

인덱스를 적용한 후의 좋아요를 취소하는 경우의 API는 25ms로, 적용하기 전의 370ms보다 15배가까이 속도차이가 난다는 것을 확인할 수 있었습니다.  
인덱스를 적용한 후 평균적으로 27ms ~ 28ms의 속도가 나타났으며, 인덱스를 적용하기 전 평균 속도인 330 ~ 340 보다 약 12배정도 빨라진 것을 확인할 수 있었습니다.

</br>

---

## **_결론_**

올바른 방법으로 테스트를 하는 방법을 몰라, 저가 할 수 있는 방향으로 테스트를 해서 오류가 있을 순 있겠지만 좋아요의 비즈니스 로직에서 insert나 delete의 갱신 질의에 의해 인덱스 변경이 일어나 성능 저하가 발생하는 부분보다 사용자가 해당 게시글에 좋아요를 했는지 안했는지에 대해 판별하는 조회 쿼리에 복합 인덱스가 적용되는 것이 성능이 좋다는 것을 알 수 있었습니다.

그런데, 인덱스에 대해 찾아보면서 데이터의 수가 적을 경우 인덱스를 태워서 데이터를 가져오는 경우보다 풀스캔으로 처음부터 데이터를 전부 가져오는 것이 성능에 있어 더 좋은 경우도 있다고 합니다.  
적은 데이터를 가져올 뿐인데도 인덱스에 대한 연산을 추가로 수행하기 때문이라고 합니다.

해당 프로젝트에 운영 서버에서 사용중인 DB에는 데이터가 매우 적게 들어있어, 인덱스를 사용하지 않는 편이 성능이 더 좋을 수 있을 것이라 생각합니다.  
하지만, 해당 프로젝트는 실제로 운영하는 것이 목표가 아닌 개발자로써의 역량을 늘리기 위한 것이 목표이기 때문에 인덱스를 적용한 경험을 프로젝트에 녹이기로 하였고, 로직에도 반영을 하기로 결론을 내렸습니다.
