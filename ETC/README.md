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
----------------------------------------------------------------------------
### Supabase에서 RLS설정된 테이블 API 조회시 고려할 점
- 상황: 테이블 조회하는 API 연동 후 호출했을 때 오류는 안 나는데, 데이터가 있는데도 불구하고 빈 배열로 조회됨
- 원인: RLS때문에 로그인 유저만 조회 가능하도록 설정되서, API 호출시 로그인 정보를 전달해줘야하는데 유저 정보가 빠져있었음
- 해결방법: supabase 호출 시 유저 정보 (access token) 포함되게 해야하고, 서버 API 요청시  session 기준으로 처리

  - src/app/api/shelf/route.ts

  AS-IS
  ```
  export async function GET() {
  const { data, error } = await supabase.from("shelf").select("*");

  if (error) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
  return NextResponse.json(data);
  }
  ```

  TO-BE
  ```
  export async function GET() {
  const supabase = createRouteHandlerClient({ cookies });
  const {
    data: { session },
  } = await supabase.auth.getSession();

  if (!session) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const { data, error } = await supabase
    .from("shelf")
    .select("*")
    .eq("owner_id", session.user.id); // 현재 로그인한 유저 기준 필터

    ...
  ```
   - page에서 API 호출할 때 credentials 포함해서 보내기
     ```
     const res = await fetch("/api/shelf", {
          method: "GET",
          credentials: "include", // 세션 쿠키 포함해서 보냄
        });
     ```
  자세한 코드는 [PR내용](https://github.com/solbi0802/clip-n-keep/pull/46) 참조 


