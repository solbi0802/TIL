### Postgresql Policy 설정
- Supabase에서는 RLS(Row Level Security) 설정 작업을 하다가 알게 되었다.
- 사용자별 데이터 조회, 수정, 삭제 등의 권한을 부여하기 위해 DB에서 정책을 설정할 수 있다.

  SQL문 
  ```
   CREATE POLICY 이름 ON 테이블이름
    [ AS { PERMISSIVE | RESTRICTIVE } ]
    [ FOR { ALL | SELECT | INSERT | UPDATE | DELETE } ]
    [ TO { 롤이름 | PUBLIC | CURRENT_USER | SESSION_USER } [, ...] ]
    [ USING ( 사용식 ) ]
    [ WITH CHECK ( 검사식 ) ]

   ```
  USING 사용식 <br/>
  SELECT일 경우 true일 때만 데이터가 보이고, false, null이면 보이지 않음. <br/>
  UPDATE, DELECT이면 true면 데이터 수정이 가능하고 false면 불가능하다. <br/>
  오류로 처리하지 않고 그냥 무시함. 

  WITH CHECK 검사식 <br/>
  INSERT, UPDATE 쿼리에 대해서 이 조건식을 계산해 true면 작업을 진행하고, false나 null이면 오류로 처리함.

- [Postgresql 공식 문서](https://postgresql.kr/docs/12/sql-createpolicy.html)
