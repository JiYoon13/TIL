# QueryDSL - transform 



### 구현 단계

* 게시글 프로젝트 진행 중 한 게시글 당 최대 5개의 이미지 저장 가능한 기능이 필요 
* 게시글 단건 조회, 게시글 목록 조회 시 한 게시글이 가지고 있는 이미지 리스트 조회 필요 
* BOARD : BOARD_IMAGE = 1 : N 

```java
@Entity
@Table(name = "BOARD")
@Getter
@Setter
public class Board{
    @OneToMany(mappedBy = "board", cascade = CascadeType.REMOVE, orphanRemoval = true)
    private List<BoardImage> image = new ArrayList<>();

    @Column(name = "TITLE")
    private String title;

    @Column(name = "CONTENT")
    private String content;
}
```

```java
@Entity
@Table(name = "BOARD_IMAGE")
@Getter
@Setter
public class BoardImage {
    @Id
    @Column(name = "ID", unique = true)
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;

    @Column(name = "URL")
    private String url;

    @Column(name = "FILE_NAME")
    private String fileName;

    @ManyToOne
    @JoinColumn(name = "BOARD_ID")
    private Board board;

    public void setBoard(Board board) {
        if (this.board != null) {
            this.board.getImage().remove(this);
        }
        this.board = board;
        board.getImage().add(this);
    }
}
```



#### 게시글 단건 조회 

* parameter : boardId 
  * boardId로 게시글 조회
* boardId로 boardImage 조회해 subquery 작성 

```java
@Override
    public BoardWithBest findBoard(Long boardId) {
        List<String> images = from(boardImage)
                .select(boardImage.url)
                .where(boardImage.board.id.eq(boardId))
                .fetch();

        return from(board)
                .select(Projections.constructor(BoardWithBest.class,
                        board.id,
                        ConstantImpl.create(images),
                        board.title,
                        board.content,
                        JPAExpressions.select(comment.count())
                                .from(comment)
                                .where(comment.boardId.eq(board.id)),
                        board.createTs,
                        board.modifyTs))
                .where(board.id.eq(boardId))
                .fetchFirst();
    }
```



#### 게시글 목록 조회

1. 조인 없이 board 조회 후 결과로 나온 boardId들로 BoardImage 조회해서 서비스단 로직에서 aggregation
2. board와 boardImage 조인
3. querydsl의 transform() 사용해 query에서 aggregation 



* 1번, 2번) 위에서는 간단히 board와 boardImage만 조인하면 된다고 작성했지만 연관된 관계가 많아 실제 구현하기에는 너무 복잡한 이슈가 있었음
  * ex) boardType, comment, boardBookmark, boardRanking, boardLike 등의 연관관계에 boardImage까지 추가되는 상황 
* 3번) 쿼리는 조금 복잡해지지만 한번에 가져올 수 있다는 장점





#### transform() 

* 결과 집합 aggregation 

* groupBy() : 메모리에서 쿼리 결과에 대한 집합 연산을 수행하는 집합 함수를 제공 
* ref) http://querydsl.com/static/querydsl/3.7.2/reference/ko-KR/html/ch03s02.html

```java
import static com.querydsl.core.group.GroupBy.groupBy;

// postId에 해당하는 comment 리스트 조회
Map<Integer, List<Comment>> results = query.from(post, comment)
	.where(comment.post.id.eq(post.id))
	.transform(groupBy(post.id)).as(list(comment)));
```





#### BOARD : BOARD_IMAGE = 1 : N

```java
@Override
public Page<Board> getBoards(Pageable pageable) {
	// board와 boardImage left join
    var query = from(board)
        .select(board)
        .leftJoin(boardImage)
        .on(board.id.eq(boardImage.board.id))
		// 필요한 조건 추가 
        // .where();

    // boardId로 groupBy 
    List<Board> boardList = query
        .transform(groupBy(board.id)
                   .list(Projections.constructor(Board.class,
                                                 board.id,
                                                 list(boardImage.url),
                                                 board.title,
                                                 board.content,
                                                 JPAExpressions.select(comment.count())
                                                 	.from(comment)
                                                 	.where(comment.boardId.eq(board.id)),
                                                 board.createTs,
                                                 board.modifyTs
                                                )));
}
```



#### Paging 추가 

* 기존에는 org.springframework.data.domain.Pageable  + PageableExecutionUtils.getPage() 사용 

```java
@Override
    public Page<Board> getBoards(Pageable pageable) {
        var query =  from(board)
                .select(Projections.constructor(Board.class,
                        board.id,
                        board.image,
                        board.title,
                        board.content,
                        board.views,
                        board.likes,
                        JPAExpressions.select(comment.count())
                                .from(comment)
                                .where(comment.boardId.eq(board.id)),
                        board.createTs,
                        board.modifyTs))
                .where();

        List<Board> boardList = Objects.requireNonNull(getQuerydsl())
                .applyPagination(pageable, query)
                .fetch();

        var countQuery = from(board)
                .select(board.count())
                .where();

        return PageableExecutionUtils.getPage(boardList, pageable, countQuery::fetchOne);
    }
```



* transform()에서는 countQuery 작성이 어려움 
* query에 offset, limit을 추가 해 페이징 처리 

```java
.offset(pageable.getOffset())
.limit(pageable.getPageSize())
```



* 정렬까지 구현하기 위해 OrderSpecifier를 작성

```java
private List<OrderSpecifier> getOrderSpecifier(Sort sort) {
    List<OrderSpecifier> orders = new ArrayList<>();

    sort.stream().forEach(order -> {
        Order direction = order.isAscending() ? Order.ASC : Order.DESC;
        String property = order.getProperty();
        PathBuilder orderByExpression = new PathBuilder(Board.class, "board");
        orders.add(new OrderSpecifier(direction, orderByExpression.get(property)));
    });

    return orders;
}

...
    
var query =  from(board)
    .orderBy(getOrderSpecifier(pageable.getSort())
            .toArray(OrderSpecifier[]::new))
    .offset(pageable.getOffset())
    .limit(pageable.getPageSize())
	...

    
long total = query.fetchCount();
return new PageImpl<>(boardList, pageable, total);
```



* 위 방법으로 BOARD : BOARD_IMAGE = 1 : N 연관관계에서 한번에 이미지까지 조회 후 페이징(정렬 + 페이징)까지 처리 

