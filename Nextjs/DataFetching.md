### 새로고침해도 데이터 유지되게 하는 방법

```
export async function getServerSideProps({ query: { id } }) {
  return {
    props: {
      id,
    },
  };
}
```

SSR 쿼리값을 전달해주면 새로고침해도 값이 사라지지 않는다.

참고자료
https://nextjs.org/docs/basic-features/data-fetching/get-server-side-props
