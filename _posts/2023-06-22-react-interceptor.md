---
title: "Axios 인터셉터 내부에서 hooks 사용"

categories:
- React

tags:
- React
- Axios
---

# Axios 중복 코드 없애기

```javascript
axios.post(`http://localhost:8080/likes/api/${postId}`, null, {
			headers: {
				"Authorization": "Bearer " + localStorage.getItem("access_token"),
			}
		}).then(res => {
			setIsLike(res.data.isLike);
			setLikes(res.data.likeCount);
			console.log('add')
			console.log(likes)
		})
```

이런식으로 모든 api를 호출하기는 너무 번거로움
그래서 axios.create라는 걸 활용함
<br>
<br>

```javascript
const instance = axios.create({
  baseURL: process.env.REACT_APP_url,
  timeout: 10000,
  headers: cookie.load("accessToken") ? {"Authorization": `Bearer ${cookie.load("accessToken")}`} : undefined
});
```

이런식으로 내가 직접 axios 인스턴스를 만든 후

**instance.interceptors.response.use**   
**instance.interceptors.request.use**   
를 활용해서 request 전에 먼저 처리할 기능, response로 응답 받고 처리 전에
수행할 코드를 작성 한 후 Promise를 반환하고 이를 나중에 가져다 쓰면 된다.
그리고 먼저 처리할 기능을 설정해두고 이 instance를 가져다가 쓰면 쉽게 개발이 가능하다.

근데 문제가 이 함수들 내부에서 예외 발생시 처리를 하는 것 까진 문제가 없었는데   
navigate같은 hook을 사용할 수가 없었다
hook은 컴포넌트 내에서만 사용 가능하다고 한다.

[Axios 내부에서 hook 사용법](https://dev.to/arianhamdi/react-hooks-in-axios-interceptors-3e1h)

구글을 많이 뒤져봤지만 history를 쓴다던가 리액트에 대해서 잘 몰라서
위 링크에서 나온 방법을 활용했다.

```javascript
// CustomAxios.js
import axios from "axios";
import {useNavigate} from "react-router-dom";
import cookie from "react-cookies";
import {useEffect, useState} from "react";


const instance = axios.create({
	baseURL: process.env.REACT_APP_url,
	timeout: 10000,
	headers: cookie.load("accessToken") ? {"Authorization": `Bearer ${cookie.load("accessToken")}`} : undefined
});

const AxiosInterceptor = ({children}) => {
	const [isSet, setIsSet] = useState(false)

	const navigate = useNavigate()

	useEffect(() => {
		setIsSet(true)
		const resInterceptor = (response) => {
			return response;
		};

		const errInterceptor = (error) => {
			if (error.response.status === 403 || error.response.status === 404) {
				alert("로그인이 필요합니다.")
				navigate('/login')
			}

			return Promise.reject();
		};

		const interceptor = instance.interceptors.response.use(
			resInterceptor,
			errInterceptor
		);

		return () => instance.interceptors.response.eject(interceptor);
	}, []);

	return isSet && children;
}


export default instance;
export {AxiosInterceptor}
```

이렇게 컴포넌트로 만든 후 이 컴포넌트를 

```javascript
<AuthProvider>
      <BrowserRouter>
        <AxiosInterceptor> // 여기
          <Header />
          <Routes>
            <Route path="/" element={<Home />}></Route>
            <Route path="/home" element={<Home />}></Route>
            <Route path="/boards" element={<Posts />}></Route>
            <Route path="/login" element={<Login />}></Route>
            <Route path="/signup" element={<Signup />}></Route>
            <Route path="/write" element={<Write />}></Route>
            <Route path="/post/:postId" element={<Post />}></Route>
            <Route path="/:postId/edit" element={<Edit />}></Route>
            <Route path="/auth/google" element={<GoogleAuth />}></Route>
            <Route path="/auth/kakao" element={<KakaoAuth />}></Route>
            <Route path="/search" element={<Search />}></Route>
            <Route path="/test" element={<Test />}></Route>
            <Route path="*" element={<NotFound />}></Route>
          </Routes>
        </AxiosInterceptor> // 여기
      </BrowserRouter>
</AuthProvider>
```
이렇게 만든 컴포넌트로 라우트들을 감싼 후 사용하면 
interceptor 내부에서 바로 hook을 사용해서
예외처리를 손쉽게 할 수가 있다.