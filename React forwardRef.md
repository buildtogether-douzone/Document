## React forwardRef

### forwardRef

+ 사용자 컴포넌트에 다음과 같이 ref를 전송 하면 에러가 발생한다.

```react
<Custom ref={submitRef} >
```

react 공식 문서를 찾아 보면 이러한 이유는 함수 컴포넌트는 ref가 존재 하지 않기 때문이다.

![tempsnip](https://user-images.githubusercontent.com/67888402/135396909-4e15f81b-fe1d-4cba-aee6-ff68d7e71f34.png)

따라서 ref 를 넘겨 주려면 다음과 같이 **forwardRef** 를 사용해야 한다.

```react
const Custom = React.forwardRef(({ props }, ref) => {
    return(
        .
        .
        .
        .
        .
        
    );
});
export default Custom;
```



만약 현재 페이지에서 redux를 사용 하는 도중이면 다음과 같이 export 하는 부분에 설정을 해 줘야한다.

```react
const Custom = React.forwardRef(({ props }, ref) => {
    return(
        .
        .
        .
        .
        .
        
    );
});

const mapStateToProps = (state)=>{
  return{
    ...
  }
}

const mapDispatchToProps = {
    ...
}

export default connect(mapStateToProps, mapDispatchToProps, null, {forwardRef: true}) (Custom);
```



