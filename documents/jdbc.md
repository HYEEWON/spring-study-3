# JDBC (Java DataBase Connectivity)

* 자바에서 DB 연동을 위해 사용하는 라이브러리 
* JDK는 DBMS에 종속되지 않는 JDBC API를 제공
* JDBC 드라이버: 각 DBMS 회사에서 제공하는 라이브러리 압축파일

<br>

## JDBC 드라이버 로드
```java
// DBC 드라이버 로드
// MySQL의 드라이버 클래스를 JVM Method Area에 로딩
Class.forName("com.mysql.Jdbc.Driver"); 
```

### DB 연결
```java
String jdbc_url = "jdbc:mysql://localhost:3306/datebase?serverTimezone=UTC";
Connection con = DriverManager.getConnection(URL, "user", "password");

```

<br>

## JdbcTemplate 클래스
* 템플릿 메소드 패턴이 적용된 클래스
* JDBC의 반복적인 코드를 제거하기 위해 사용하는 클래스

### 데이터소스 설정
* DB Connection을 얻기 위한 방법

* properties 파일과 xml 파일 이용
  * properties를 이용하지 않고 xml에 직접 입력해도 됨

```
// datasource.properties
db.driverClass=com.mysql.cj.jdbc.Driver
db.url=jdbc:mysql://localhost:3306/DATABASE_NAME?useSSL=false&serverTimezone=Asia/Seoul
db.username=name
db.password=pw
```

```xml
<!-- properties 파일과 연결 -->
<context:property-placeholder location="classpath:config/datasource.properties"/>

<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource" 
    destroy-method="close">
    <property name="driverClass" value="${db.driverClass}" />
    <property name="url" value="${db.url}"/>
    <property name="username" value="${db.username}" />
    <property name="password" value="${db.password}" />
</bean>
```

### JdbcTemplate 클래스 등록
* JdbcTemplate 클래스를 등록 <bean>으로 등록하고 의존성 주입으로 처리

```xml
<!-- JdbcTemplate 등록 -->
	<bean class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"/>
	</bean>
```

### DAO 작성
* update(): DB 갱신 (INSERT, DELETE, UPDATE)
* queryForObject(): SELECT 결과 하나의 튜플이 나올 경우 사용
* query(): SELECT 결과 많은 튜플이 반환될 경우 사용

```java
@Repository
public class DaoClass {

	@Autowired
	private JdbcTemplate jdbcTemplate;
	
	// BOARD 테이블 관련 SQL 명령어들
	private final String ITEM_INSERT = "insert into item(seq, name) values((select nvl(max(seq), 0)+1 from item),?)";
	private final String ITEM_GET = "select * from item where seq=?";
	private final String ITEM_LIST = "select * from item order by seq desc";
	
	// 아이템 등록
    // INSERT, DELETE, UPDATE 모두 동일한 방법으로 진행
	public void insertItem(BoardVO vo) {
		jdbcTemplate.update(ITEM_INSERT, "name");
	}
	
	// 아이템 상세 조회
	public BoardVO getItem(BoardVO vo) {
		Object[] params = {vo.getSeq()};
		return jdbcTemplate.queryForObject(ITEM_GET, params, new BoardRowMapper());
	}
	
	// 아이템 목록 검색
	public List<BoardVO> getItemList(BoardVO vo) {
		return jdbcTemplate.query(ITEM_LIST, new BoardRowMapper());
	}
}
```

