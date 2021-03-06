# Part3 정리

## 스프링 MVC 프로젝트의 기본 구성

앞으로 Controller, Service, Repository 등 패키지를 나누어 코딩을 할 것인대 왜 나누는 지에 대해 이해가 부족한 것 같아서 구글링을 통해 조사해보았다.

### 계층형 아키텍처

웹 어플리케이션은 성격이 다른 것들을 다른 아키텍처 레벨에서 처리해줘 유지보수를 편리하게 해준다.

만약, 분리하지 않고 JSP처럼 HTML, JDBC 코드등이 함께 존재한다면, 유지보수는 거의 불가능해진다.

이를 계층형 아키텍처라고 부르고, Java 즉 Spring 프레임워크에서는 주로 3-Tier 어플리케이션 아키텍처를 사용한다.

### Web Project의 3-Tier(티어) 기본 구성

브라우저에서 전송한 데이터를 스프링 MVC는 아래와 같은 3-tier 방식을 거쳐 처리하게 된다.

![WEB Project 3-Tier](./picture/WEB_3_Tier.JPG)

1. 화면 계층(Presentation Tier)

    화면에 보여주는 기술을 사용하는 영역으로 주로 Servlet/JSP 나 스프링MVC가 담당하는 영역

    클라이언트의 종류와 상관없이 HTTP 프로토콜을 사용한다.

2. 서비스,비지니스(Business Tier)

    순수한 비즈니스 로직을 담고 있는 영역

    고객이 원하는 요구 사항을 반영하는 계층으로 고객의 요구 사항과 정확히 일치해야한다.

    주로 'xxxService'와 같은 이름으로 구성하고, 메서드의 이름 역시 고객들이 사용하는 용어를 그대로 사용하는 것이 좋다.

    이상적인 서비스 코드는 DB와 연결되는 데이터 엑세스 계층이 바뀌거나 클라이언트와 연결되는 프레젠테이션 계층이 모두 바뀌어도 그대로 유지 될 수 있어야 한다.

3. 영속계층, 데이터 엑세스 계층(Persistence Tier)

    데이터를 어떤 방식으로 보관하고, 사용하는가에 대한 설계가 들어가는 계층

    일반적인 경우에는 데이터베이스를 많이 이용하지만, 경우에 따라선 네트워크 호출이나 원격 호출 등의 기술이 접목될 수 있다.

    Spring에서는 아래 그림처럼 같은 데이터 엑세스 계층이지만 역할에 맞게 수직적으로 나눌 수 있다.

    ![데이터 엑세스 계층 수직 계층 구조](./picture/Persistence_Tier.JPG)

---

## Part 3 각 영역의 Naming 명명 규칙

- xxxController   
    : 스프링 MVC에서 동작하는 Controller 클래스를 설계할 때 사용
- xxxService, Servicelmpl  
    : 비지니스 영역의 인터페이스는 Service, 인터페이스를 구현한 클래스는 Servicelmpl 사용
- xxxDao, xxxRepository  
    : 데이터 엑세스 영역을 DAO(Data-Access-Object)나 Repository라는 이름으로 명명, 이 책에서는 별도의 DAO를 구성하는 대신 MyBatis의 Mapper 인터페이스 활용
- VO,DTO  
    : VO와 DTO는 일반적으로 유사한 의미로 데이터를 담고 있는 객체를 의미한다는 공통점이 있다. 다만, VO는 주로 Read Only의 목적이 강하고, 데이터 자체도 Immutable(불변)하게 설계하는 것이 정석이다. DTO는 주로 데이터 수집의 용도가 더 강하다.

    ex) 웹 화면에서 로그인하는 정보를 DTO로 처리

---

## 영속 계층의 작업 순서

영속 계층의 작업은 항상 다음과 같은 순서로 진행한다.

0. JDBC 연결 테스트(SQL Developer의 연결로 처리 완료)
1. 테이블의 칼럼 구조를 반영하는 VO(Value Object) 클래스의 생성
2. Mybatis의 Mapper 인터페이스의 작성/XML 처리
3. 작성한 Mapper 인터페이스의 테스트

### 1. VO 클래스의 생성

VO 클래스 생성은 테이블 설계를 기준으로 작성하면 된다.  
현재 tbl_board의 구성은 아래와 같다.

![tbl_board 구성](./picture/tbl_board.JPG)

이 구성표를 가지고 BoardVO 클래스를 만들어 준다.

```
org.zerock.domain package/BoardVO

@Data
public class BoardVO {
	private Long bno;
	private String title;
	private String content;
	private String writer;
	private Date regdate;
	private Date updateDate;
}
```

### 2. Mybatis의 Mapper 인터페이스와 Mapper XML

PART1에서 처럼 MyBatis는 SQL을 처리하는데 어노테이션이나 XML을 이용할 수 있다. 주로 간단한 SQL이라면 어노테이션을, 복잡하고 상황에 따라 다른 SQL문이 처리되는 경우라면 XML을 사용한다.

Mybatis를 이용하려면 PART1에서 처럼 Mapper 인터페이스를 읽을 수 있도록 스캔할 'base-package'를 'root-context.xml'에 등록해주어야 한다.

```
root-context.xml

<mybatis-spring:scan base-package="org.zerock.mapper"/>
```

**Mapper 인터페이스 방법**

Mapper 인터페이스를 작성할 때에는 select와 insert 작업을 우선해서 작성한다. org.zerock.mapper 패키지를 작성하고 BoardMapper 인터페이스를 작성한다.

```
org.zerock.mapper/BoardMapper
어노테이션 방법

public interface BoardMapper {
	@Select("select * from tbl_board where bno > 0")
	public List<BoardVO> getList();
}
```

Mapper 인터페이스에선 앞서 작성된 VO 클래스를 이용해 필요한 SQL을 어노테이션의 속성 값으로 처리할 수 있다. **이 방법으로 SQL을 작성할 때는 ';'이 없도록 작성해야 한다.**

**Mapper XML 방법**

'src/main/resources'에 'org/zerock/mapper'단계의 폴더를 생성하고 'BoardMapper.xml' 파일을 작성한다.(폴더를 한 번에 생서하지 말고 하나씩 생성해야 한다.) 파일의 폴더 구조나 이름은 상관없지만 주로 패키지와 클래스 이름과 동일하게 해주어 혼란스러운 상황을 피한다.

```
src/main/resources/org/zerock/mapper/BoardMapper.xml

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 
	mapper 연결
	namespace 값 : Mapper 인터페이스와 동일해야 한다.
 -->
<mapper namespace="org.zerock.mapper.BoardMapper">

<!-- 
	select SQL 연결
	id 값 : 메서드의 이름과 동일
	resultType 값 : select 쿼리 결과를 담을 클래스의 객체
	
	CDATA : XML에서 부등호를 사용하기 위해 사용
 -->
<select id="getList" resultType="org.zerock.domain.BoardVO">
<![CDATA[
select * from tbl_board where bno > 0
]]>
</select>
</mapper>
```

XML로 SQL문을 처리하는 경우는 Mapper 인터페이스에 SQL문은 제거해줘야 한다.

```
org.zerock.mapper/BoardMapper

public interface BoardMapper {
	// XML방법을 사용하므로 어노테이션 제거
    // @Select("select * from tbl_board where bno > 0")
	public List<BoardVO> getList();
}
```

### 3. Mapper 인터페이스 테스트

위에서 작성한 Mapper 인터페이스를 테스트 하기위해 'src/test/java'에 'org.zerock.mapper' 패키지를 작성하고 'BoardMapperTests' 클래스를 작성한다.

```
src/test/java/org.zerock.mapper/BoardMapperTests

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("file:src/main/webapp/WEB-INF/spring/root-context.xml")
@Log4j
public class BoardMapperTests {
	@Setter(onMethod_ = @Autowired)
	private BoardMapper mapper;
	
	@Test
	public void testGetList() {
		mapper.getList().forEach(board -> log.info(board));
	}
}
```

---

## 영속 영역의 CRUD 구현

영속 영역은 테이블과 VO(DTO) 등 약간의 준비만으로도 비즈니스 로직과 무관하게 CRUD 작업을 자성할 수 있다.

MyBatis는 내부적으로 JDBC의 PreparedStatement를 활용하고 필요한 파라미터를 처리하는 '?'를 '#{속성}'을 이용해서 처리한다.

### **create(insert) 처리**

tbl_board 테이블은 PK 칼럼으로 bno를 이용하고 시퀀스를 이용해서 자동으로 데이터가 추가될 때 번호가 만들어지는 방식을 사용한다.

이처럼 자동으로 PK값이 정해지는 경우에는 다음과 같은 2가지 방식으로 처리할 수 있다.

- insert문만 처리되고 생성된 PK값을 알 필요가 없는 경우
- insert문이 실행되고 생성된 PK값을 알아야 하는 경우

이와 같은 경우를 처리하기 위해 BoardMaper 인터페이스에  
- insert(생성된 PK값을 알 필요가 없는 경우)
- insertSelectKey(생성된 PK값을 알아야 하는 경우)  

메서드를 추가적으로 선언해준다.

```
org.zerock.mapper/BoardMapper

public interface BoardMapper {	
    // 추가되는 내용
    public void insert(BoardVO board);
	public void insertSelectKey(BoardVO board);

    // 어노테이션을 이용하면 아래와 같다.
    
    // @Insert("insert into tbl_board (bno,title,content,writer) values (seq_board.nextval, #{title}, #{content}, #{writer})")
	public void insert(BoardVO board);

    // bno값을 알야해서 select가 먼저 실행되어야 하는데 처리 방법을 모르겠음 
	public void insertSelectKey(BoardVO board);
}
```

BoardMapper.xml에는 다음과 같은 내용이 추가되어야 한다.

```
src/main/resources/org/zerock/mapper/BoardMapper.xml

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.zerock.mapper.BoardMapper">

<insert id="insert">
insert into tbl_board (bno,title,content,writer) values (seq_board.nextval, #{title}, #{content}, #{writer})
</insert>

<insert id="insertSelectKey">
    <!-- bno 값을 구하는 코드 -->
	<selectKey keyProperty="bno" order="BEFORE" resultType="long">
	select seq_board.nextval from dual
	</selectKey>

insert into tbl_board (bno,title,content,writer) values (#{bno}, #{title}, #{content}, #{writer}) 
</insert>

</mapper>
```

insert()는 단순히 다음 시퀀스 값을 구해서 insert할 때 사용하므로 1번의 SQL 처리만으로 작업이 완료된다.

insertSelectKey()는 @SelectKey라는 MyBatis의 어노테이션을 이용해 PK값을 미리(before) SQL을 통해서 처리해 두고 특정한 이름으로 결과를 보관하는 방식을 이용해 이 처리된 결과로 insert문을 실행하는 방식을 이용한다.

아래는 위의 Mapper를 Test할 수 있는 Test 클래스 추가 코드이다.

```
src/test/java/org.zerock.mapper/BoardMapperTests

// insert문 Test
// bno가 null로 입력
@Test
public void testInsert() {
	BoardVO board = new BoardVO();
	board.setTitle("새로 작성하는 글");
	board.setContent("새로 작성하는 내용");
	board.setWriter("newbie");

	mapper.insert(board);
	log.info(board);
}

// insertSelectKey문 Test
@Test
public void testInsertSelectKey() {
	BoardVO board = new BoardVO();
	board.setTitle("새로 작성하는 글 select key");
	board.setContent("새로 작성하는 내용 select key");
	board.setWriter("newbie");

	mapper.insertSelectKey(board);
	log.info(board);
}
```

### **Read(Select) 처리**

Read 처리는 데이터를 조회하는 작업(Select)으로 PK를 이용해서 처리하므로 BoardMapper의 파라미터 역시 BoardVO 클래스의 bno 타입 정보를 이용해서 처리한다.

따라서, 아래와 같이 BoardMapper 인터페이스의 메서드가 bno를 파라미터로 가진다.

```
org.zerock.mapper/BoardMapper 인터페이스 일부

public interface BoardMapper {
    public BoardVO read(Long bno);
}
```

MyBatis는 Mapper 인터페이스의 리턴 타입에 맞게 select의 결과를 처리하기 때문에 아래의 select문의 결과는 BoardVO에 각 속성에 맞는 값으로 처리된다.

```
src/main/resources/org/zerock/mapper/BoardMapper.xml
<select id="read" resultType="org.zerock.domain.BoardVO">
    select * from tbl_board where bno = #{bno}
</select>
```

위 Read 인터페이스를 테스트 할 코드는 아래와 같다

```
src/test/java/org.zerock.mapper/BoardMapperTests
//Read 처리 Test
@Test
public void testRead() {
    // read함수로 bno값이 5인 board를 찾아서 출력
    BoardVO board = mapper.read(5L);
    log.info(board);
}
```

### **Delete 처리**

Delete 처리 역시 특정한 데이터를 삭제하는 것으로 PK를 이용해서 처리한다. 등록, 삭제, 수정과 같은 DML 작업은 '몇 건의 데이터가 삭제(혹은 수정) 되었는지를' 반환할 수 있다.

read와 같이 bno를 파라미터로 가진다.

```
org.zerock.mapper/BoardMapper 인터페이스 일부

public interface BoardMapper {
	public int delete(long bno);
}
```

```
src/main/resources/org/zerock/mapper/BoardMapper.xml
<delete id="delete">
	delete from tbl_board where bno = #{bno}
</delete>
```

위 Delete 인터페이스를 테스트 할 코드는 아래와 같다

```
src/test/java/org.zerock.mapper/BoardMapperTests
// Delete 처리 Test
@Test
public void testDelete() {
	// delete함수로 bno값이 3인 board를 찾아서 삭제 후 삭제 count 출력
	log.info("DELETE COUNT: " + mapper.delete(3L));
}
```

### **Update 처리**

Update는 게시물등의 제목, 내용, 작성자를 수정하는 것으로 업데이트할 때에 최종 수정시간을 데이터베이스 내 현재 시간으로 수정해줘야 한다. Update 처리는 delete와 마찬가지로 '몇 개의 데이터가 수정되었는가'를 처리할 수 있게 int타입으로 메서드를 설계할 수 있다.

```
org.zerock.mapper/BoardMapper 인터페이스 일부

public interface BoardMapper {
	public int update(BoardVO board);
}
```

수행할 SQL문은 updateDate가 현재 시간으로 수정되고, regdate 칼럼은 최초 생성 시간이므로 건드리지 않는 것이 중요하다.

```
src/main/resources/org/zerock/mapper/BoardMapper.xml
<update id="update">
	update tbl_board
	set title = #{title},
	content = #{content},
	writer = #{writer},
	updateDate = sysdate
	where bno = #{bno}
</update>
```

위 Update 인터페이스를 테스트 할 코드는 아래와 같다.  
테스트 코드는 read()를 이용해서 가져온 BoardVO 객체의 일부를 수정하거나, 새로운 객체를 생성해서 처리할 수 있다. 예제는 객체를 생성해서 테스트를 진행한다.

```
src/test/java/org.zerock.mapper/BoardMapperTests
// Update 처리 Test
@Test
public void testUpdate() {
	BoardVO board = new BoardVO();
	// 실행전 수정할 번호가 존재하는 지 확인해줄 것
	board.setBno(5L);
	board.setTitle("수정된 제목");
	board.setContent("수정된 내용");
	board.setWriter("user00");
	
	int count = mapper.update(board);
	log.info("UPDATE COUNT: " + count);
}
```

---

## **비지니스 계층의 구현**

비즈니스 계층은 고객의 요구사항을 반영하는 계층으로 프레젠테이션과 영속 계층의 중간 다리 역할을 한다. 비즈니스 계층은 로직을 기준으로 해서 처리하게 된다.

ex) 상품을 구매한다 -> 영속계층 : '상품' '회원'으로 나누어 설계  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> 비즈니스 계층 : '구매 서비스'로 상품과 회원을 합쳐서 로직으로 설계


비지니스 계층 구현 시에 객체들을 주로 'Service(서비스)'용어로 표현한다.

### **BoardService 인터페이스와 BoardServiceImpl 구현**

예제에서 사용할 BoardService 인터페이스와 이를 구현한 BoardServiceImpl을 작성한다.

```
org.zerock.service/BoardService

public interface BoardService {
	
	public void register(BoardVO board);
	
	public BoardVO get(Long bno);
	
	public boolean modify(BoardVO board);
	
	public boolean remove(Long bno);
	
	public List<BoardVO> getList();
	
}
```

인터페이스의 메서드를 설계할 때 메서드 이름은 현실적인 로직의 이름을 붙이는 것이 좋다. 명백하게 반환해야 할 데이터가 있는 경우는 메서드의 리턴 타입을 지정할 수 있다.

```
org.zerock.service/BoardServiceoImpl

@Log4j
@Service
@AllArgsConstructor
public class BoardServiceImpl implements BoardService{

	private BoardMapper mapper;
	
	@Override
	public void register(BoardVO board) {
		
	}

    ...
}
```

BoardService를 구현한 BoardServiceImpl은 지금은 약간의 로그를 기록할 수 있는 정도의 코드를 작성한다.

mapper 객체를 이용해서 영속 계층 작업을 수행하는 것을 알 수 있다.

@Service 어노테이션으로 이 객체가 비즈니스 영역을 담당하는 객체임을 표시해준다.
이후에 root-context.xml에 표시된 Service가 Spring에서 인식할 수 있도록 base-package로 Service 패키지를 등록해준다. (Context항목이 없으면 Namespace에서 추가)

```
root-context.xml

<context:component-scan base-package="org.zerock.service"></context:component-scan>
```

### **비지니스 계층 테스트**

첫 테스트는 BoardService 객체가 제대로 주입이 가능한지 확인한다.

```
src/test/java/org/zerock/service/BoardServiceTests

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("file:src/main/webapp/WEB-INF/spring/root-context.xml")
@Log4j
public class BoardServiceTests {
	@Setter(onMethod_ = {@Autowired})
	private BoardService service;
	
    // service 의존성 주입 Test
	@Test
	public void testExist() {
		log.info(service);
		assertNotNull(service);
	}
}
```

### **등록 작업의 구현과 테스트**

등록 작업은 BoardServiceImpl에서 파라미터로 전달되는 BoardVO 타입의 객체를 BoardMapper를 통해서 처리한다.

```
org.zerock.service/BoardServiceoImpl

public class BoardServiceImpl implements BoardService{

    private BoardMapper mapper;

    @Override
    public void register(BoardVO board) {
        log.info("register....." + board);
        
        mapper.insertSelectKey(board);
    }
}
```

```
src/test/java/org/zerock/service/BoardServiceTests

// Register(영속 : insertkey) 처리 Test
@Test
public void testRegister() {
    BoardVO board = new BoardVO();
    board.setTitle("새로 작성하는 글");
    board.setContent("새로 작성하는 내용");
    board.setWriter("newbie");
    
    service.register(board);
    
    log.info("생성된 게시물의 번호: " + board.getBno());
}
```

### **목록(리스트) 작업의 구현과 테스트**

목록(리스트) 작업은 BoardServiceImpl에서 현재 테이블에 저장된 모든 데이터를 가져오므로 반환 값으로 `List<BoardVO>`을 사용하는 형태로 구현한다.

```
org.zerock.service/BoardServiceoImpl

public class BoardServiceImpl implements BoardService{

    private BoardMapper mapper;

    @Override
	public List<BoardVO> getList() {
		
		Log.info("getList......");
		
		return mapper.getList();
	}
}
```

getList의 Test코드는 결과값을 반복해서 찍어보는 형식으로 작성한다.

```
src/test/java/org/zerock/service/BoardServiceTests

// getList(영속 : 전체 select) 처리 Test
@Test
public void testGetList() {
    service.getList().forEach(board->log.info(board));
}
```

### **조회 작업의 구현과 테스트**

조회 작업은 BoardServiceImpl에서 특정 게시물의 번호(bno)가 파라미터이고, 그 bno 값에 해당하는 BoardVO의 인스터스가 리턴되는 메서드로 작성한다.

```
org.zerock.service/BoardServiceoImpl

public class BoardServiceImpl implements BoardService{

    private BoardMapper mapper;

    @Override
	public BoardVO get(Long bno) {
		
		Log.info("get....." + bno);
		
		return mapper.read(bno);
	}
}
```

get의 Test코드는 결과값을 출력하는 형식으로 작성한다.

```
src/test/java/org/zerock/service/BoardServiceTests

// get(영속 : Read) 처리 Test
@Test
public void testGet() {
    log.info(service.get(1L));
}
```

### **삭제/수정 작업의 구현과 테스트**

삭제/수정 작업은 메서드 리턴 타입을 void로 설계해도 되지만 엄격하게 처리하기 위해 Boolean 타입으로 처리한다.

```
org.zerock.service/BoardServiceoImpl

public class BoardServiceImpl implements BoardService{

    private BoardMapper mapper;

    // 수정
    @Override
	public boolean modify(BoardVO board) {
		
		Log.info("modify......" + board);
		
        // boolean 형으로 반화하기 위해 == 1을 해준다.
		return mapper.update(board) == 1;
	}

    //삭제
	@Override
	public boolean remove(Long bno) {
		
		Log.info("remove......" + bno);
		
        // boolean 형으로 반화하기 위해 == 1을 해준다.
		return mapper.delete(bno) == 1;
	}
}
```

Test코드는 결과값을 출력하는 형식으로 작성한다.

```
src/test/java/org/zerock/service/BoardServiceTests

// Delete(영속 : delete) 처리 Test
@Test
public void testUpdate() {
    // 수정할 게시물 가져오기
    BoardVO board = service.get(1L);
    
    // 존재한다면
    if(board == null) {
        return;
    }
    
    // 수정 후 출력
    board.setTitle("제목 수정합니다.");
    log.info("MODIFY RESULT: " + service.modify(board));
}

// Delete(영속 : delete) 처리 Test
@Test
public void testDelete() {
    log.info("REMOVE RESULT: " + service.remove(2L));
}
```